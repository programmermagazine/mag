## Nand2Tetris 第三週 -- 記憶體

### a 部份

[Bit.hdl](nand2tetris/projects/03/a/Bit.hdl)

```
CHIP Bit {
    IN in, load;
    OUT out;

    PARTS:
		Mux(a=do, b=in, sel=load, out=mo);
		DFF(in=mo, out=do);
		And(a=do, b=do, out=out);
}
```


[Register.hdl](nand2tetris/projects/03/a/Register.hdl)

```
CHIP Register {
    IN in[16], load;
    OUT out[16];

    PARTS:
    Bit(in=in[15], load=load, out=out[15]);
    Bit(in=in[14], load=load, out=out[14]);
    Bit(in=in[13], load=load, out=out[13]);
    Bit(in=in[12], load=load, out=out[12]);
    Bit(in=in[11], load=load, out=out[11]);
    Bit(in=in[10], load=load, out=out[10]);
    Bit(in=in[9],  load=load, out=out[9]);
    Bit(in=in[8],  load=load, out=out[8]);
    Bit(in=in[7],  load=load, out=out[7]);
    Bit(in=in[6],  load=load, out=out[6]);
    Bit(in=in[5],  load=load, out=out[5]);
    Bit(in=in[4],  load=load, out=out[4]);
    Bit(in=in[3],  load=load, out=out[3]);
    Bit(in=in[2],  load=load, out=out[2]);
    Bit(in=in[1],  load=load, out=out[1]);
    Bit(in=in[0],  load=load, out=out[0]);
}
```

[PC.hdl](nand2tetris/projects/03/a/PC.hdl)

![圖、以下程式的電路結構](PcCircuit.jpg)

```
CHIP PC {
    IN in[16],load,inc,reset;
    OUT out[16];

    PARTS:
    Or(a=load, b=inc, out=loadInc);
    Or(a=loadInc, b=reset, out=loadIncReset);

    Register(in=if3, load=loadIncReset, out=o);
    Inc16(in=o, out=oInc);
    And16(a=o, b=o, out=out);
    
    Mux16(a=o,   b=oInc,  sel=inc,   out=if1);
    Mux16(a=if1, b=in,    sel=load,  out=if2);
    Mux16(a=if2, b=false,  sel=reset, out=if3);
}

```

[RAM8.hdl](nand2tetris/projects/03/a/RAM8.hdl)

```
CHIP RAM8 {
    IN in[16], load, address[3];
    OUT out[16];

    PARTS:
    DMux8Way(in=true, sel=address, a=E0, b=E1, c=E2, d=E3, e=E4, f=E5, g=E6, h=E7);
    
    And(a=load, b=E0, out=L0); Register(in=in, load=L0, out=r0);
    And(a=load, b=E1, out=L1); Register(in=in, load=L1, out=r1);
    And(a=load, b=E2, out=L2); Register(in=in, load=L2, out=r2);
    And(a=load, b=E3, out=L3); Register(in=in, load=L3, out=r3);
    And(a=load, b=E4, out=L4); Register(in=in, load=L4, out=r4);
    And(a=load, b=E5, out=L5); Register(in=in, load=L5, out=r5);
    And(a=load, b=E6, out=L6); Register(in=in, load=L6, out=r6);
    And(a=load, b=E7, out=L7); Register(in=in, load=L7, out=r7);
    
    Mux8Way16(a=r0, b=r1, c=r2, d=r3, e=r4, f=r5, g=r6, h=r7, sel=address, out=out);
}

```


[RAM64.hdl](nand2tetris/projects/03/a/RAM64.hdl)

```
CHIP RAM64 {
    IN in[16], load, address[6];
    OUT out[16];

    PARTS:
    DMux8Way(in=true, sel=address[3..5], a=E0, b=E1, c=E2, d=E3, e=E4, f=E5, g=E6, h=E7);
    
    And(a=load, b=E0, out=L0); RAM8(in=in,  load=L0, address=address[0..2], out=o0);
    And(a=load, b=E1, out=L1); RAM8(in=in,  load=L1, address=address[0..2], out=o1);
    And(a=load, b=E2, out=L2); RAM8(in=in,  load=L2, address=address[0..2], out=o2);
    And(a=load, b=E3, out=L3); RAM8(in=in,  load=L3, address=address[0..2], out=o3);
    And(a=load, b=E4, out=L4); RAM8(in=in,  load=L4, address=address[0..2], out=o4);
    And(a=load, b=E5, out=L5); RAM8(in=in,  load=L5, address=address[0..2], out=o5);
    And(a=load, b=E6, out=L6); RAM8(in=in,  load=L6, address=address[0..2], out=o6);
    And(a=load, b=E7, out=L7); RAM8(in=in,  load=L7, address=address[0..2], out=o7);

    Mux8Way16(a=o0, b=o1, c=o2, d=o3, e=o4, f=o5, g=o6, h=o7, sel=address[3..5], out=out);
}

```

[RAM512.hdl](nand2tetris/projects/03/b/RAM512.hdl)

```
CHIP RAM512 {
    IN in[16], load, address[9];
    OUT out[16];

    PARTS:
    DMux8Way(in=true, sel=address[6..8], a=E0, b=E1, c=E2, d=E3, e=E4, f=E5, g=E6, h=E7);
    
    And(a=load, b=E0, out=L0); RAM64(in=in,  load=L0, address=address[0..5], out=o0);
    And(a=load, b=E1, out=L1); RAM64(in=in,  load=L1, address=address[0..5], out=o1);
    And(a=load, b=E2, out=L2); RAM64(in=in,  load=L2, address=address[0..5], out=o2);
    And(a=load, b=E3, out=L3); RAM64(in=in,  load=L3, address=address[0..5], out=o3);
    And(a=load, b=E4, out=L4); RAM64(in=in,  load=L4, address=address[0..5], out=o4);
    And(a=load, b=E5, out=L5); RAM64(in=in,  load=L5, address=address[0..5], out=o5);
    And(a=load, b=E6, out=L6); RAM64(in=in,  load=L6, address=address[0..5], out=o6);
    And(a=load, b=E7, out=L7); RAM64(in=in,  load=L7, address=address[0..5], out=o7);

    Mux8Way16(a=o0, b=o1, c=o2, d=o3, e=o4, f=o5, g=o6, h=o7, sel=address[6..8], out=out);
}

```

[RAM4K.hdl](nand2tetris/projects/03/b/RAM4K.hdl)

```
CHIP RAM4K {
    IN in[16], load, address[12];
    OUT out[16];

    PARTS:
    DMux8Way(in=true, sel=address[9..11], a=E0, b=E1, c=E2, d=E3, e=E4, f=E5, g=E6, h=E7);
    
    And(a=load, b=E0, out=L0); RAM512(in=in,  load=L0, address=address[0..8], out=o0);
    And(a=load, b=E1, out=L1); RAM512(in=in,  load=L1, address=address[0..8], out=o1);
    And(a=load, b=E2, out=L2); RAM512(in=in,  load=L2, address=address[0..8], out=o2);
    And(a=load, b=E3, out=L3); RAM512(in=in,  load=L3, address=address[0..8], out=o3);
    And(a=load, b=E4, out=L4); RAM512(in=in,  load=L4, address=address[0..8], out=o4);
    And(a=load, b=E5, out=L5); RAM512(in=in,  load=L5, address=address[0..8], out=o5);
    And(a=load, b=E6, out=L6); RAM512(in=in,  load=L6, address=address[0..8], out=o6);
    And(a=load, b=E7, out=L7); RAM512(in=in,  load=L7, address=address[0..8], out=o7);

    Mux8Way16(a=o0, b=o1, c=o2, d=o3, e=o4, f=o5, g=o6, h=o7, sel=address[9..11], out=out);
}
```
