## Nand2Tetris -- HDL 模擬器源碼解析

HDL 模擬器的核心，代表 CHIP 的物件為 Gate，其源碼路徑如下：

/SimulatorPackageSource/Hack/Gates 

```
    /**
     * Re-computes the values of all output pins according to the gate's functionality.
     */
    protected abstract void reCompute();

...
    /**
     * Marks the gate as "dirty" - needs to be recomputed.
     */
    public void setDirty() {
        isDirty = true;

        // notify listeners
        if (dirtyGateListeners != null)
            for (int i = 0; i < dirtyGateListeners.size(); i++)
                ((DirtyGateListener)dirtyGateListeners.elementAt(i)).gotDirty();
    }

    /**
     * Returns the GateClass of this gate.
     */
    public GateClass getGateClass() {
        return gateClass;
    }

    /**
     * Returns the node according to the given node name (may be an input or an output).
     * If doesn't exist, returns null.
     */
    public Node getNode(String name) {
        Node result = null;

        byte type = gateClass.getPinType(name);
        int index = gateClass.getPinNumber(name);
        switch (type) {
            case GateClass.INPUT_PIN_TYPE:
                result = inputPins[index];
                break;
            case GateClass.OUTPUT_PIN_TYPE:
                result = outputPins[index];
                break;
        }

        return result;
    }

    /**
     * Returns the input pins.
     */
    public Node[] getInputNodes() {
        return inputPins;
    }

    /**
     * Returns the output pins.
     */
    public Node[] getOutputNodes() {
        return outputPins;
    }

    /**
     * Recomputes the gate's outputs if inputs changed since the last computation.
     */
    public void eval() {
        if (isDirty)
            doEval();
    }

    /**
     * Recomputes the gate's outputs.
     */
    private void doEval() {
        if (isDirty) {
            isDirty = false;

            // notify listeners
            if (dirtyGateListeners != null)
                for (int i = 0; i < dirtyGateListeners.size(); i++)
                    ((DirtyGateListener)dirtyGateListeners.elementAt(i)).gotClean();
        }

        reCompute();
    }

    /**
     * First computes the gate's output (from non-clocked information) and then updates
     * the internal state of the gate (which doesn't affect the outputs)
     */
    public void tick() {
        doEval();
        clockUp();
    }

    /**
     * First updates the gate's outputs according to the internal state of the gate, and
     * then computes the outputs from non-clocked information.
     */
    public void tock() {
        clockDown();
        doEval();
    }

}

```

/SimulatorPackageSource/Hack/CompositeGate

```

package Hack.Gates;

public class CompositeGate extends Gate {

    // the internal pins
    protected Node[] internalPins;

    // The contained parts (Gates), sorted in topological order.
    protected Gate[] parts;

    protected void clockUp() {
        if (gateClass.isClocked)
            for (int i = 0; i < parts.length; i++)
                parts[i].tick();
    }

    protected void clockDown() {
        if (gateClass.isClocked)
            for (int i = 0; i < parts.length; i++)
                parts[i].tock();
    }

    protected void reCompute() {
        for (int i = 0; i < parts.length; i++)
            parts[i].eval();
    }

    /**
     * Returns the node according to the given node name (may be input, output or internal).
     * If doesn't exist, returns null.
     */
    public Node getNode(String name) {
        Node result = super.getNode(name);

        if (result == null) {
            byte type = gateClass.getPinType(name);
            int index = gateClass.getPinNumber(name);
            if (type == CompositeGateClass.INTERNAL_PIN_TYPE)
                result = internalPins[index];
        }

        return result;
    }

    /**
     * Returns the internal pins.
     */
    public Node[] getInternalNodes() {
        return internalPins;
    }

    /**
     * Returns the parts (internal gates) of this gate, sorted in topological order.
     */
    public Gate[] getParts() {
        return parts;
    }

    /**
     * Initializes the gate
     */
    public void init(Node[] inputPins, Node[] outputPins, Node[] internalPins, Gate[] parts,
                     GateClass gateClass) {
        this.inputPins = inputPins;
        this.outputPins = outputPins;
        this.internalPins = internalPins;
        this.parts = parts;
        this.gateClass = gateClass;
        setDirty();
    }

}

```

BuiltInGate

```
package Hack.Gates;

/**
 * A BuiltIn Gate. The base class for all gates which are implemented in java.
 */
public abstract class BuiltInGate extends Gate {

    protected void clockUp() {}

    protected void clockDown() {}

    protected void reCompute() {}

    /**
     * Initializes the gate
     */
    public void init(Node[] inputPins, Node[] outputPins, GateClass gateClass) {
        this.inputPins = inputPins;
        this.outputPins = outputPins;
        this.gateClass = gateClass;
        setDirty();
    }
}

```

但是我找到了一個 ruby 的實作版，網址如下：

* <https://github.com/tuzz/hdl>
 * <https://github.com/tuzz/hdl/blob/master/lib/hdl/chip/schema_chip/evaluator.rb>


* 当之无愧的哈佛CS101
 * <http://book.douban.com/review/7115224/>