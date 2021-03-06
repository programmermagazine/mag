## Enigma 密碼機的運作原理

![圖、Enigma 密碼機](Four-rotor-enigma.jpg)

Enigma 密碼機是德軍在二次大戰時期所採用的加解密機器，這台密碼機曾經是希特勒口中無法破解的機器，但是在戰爭的後期，德軍的密碼幾乎都被英國破解了，因此後來很多關鍵性的戰役都因為機密被英軍知道而導致戰情逆轉，因此二次大戰除了是強大武器的戰爭之外，也是一個密碼學的戰爭。

![圖、Enigma 的旋轉輪](640px-Enigma-rotor-stack.jpg)

![圖、Enigma 的反射插板](640px-Enigma-plugboard.jpg)

對於想要瞭解 Enigma 密碼機的運作原理的人，可以參考下列文章以及其中的兩個影片，這兩個影片清楚地展示了 Enigma 的運作原理。

* [究竟圖靈是怎樣破解德軍的密碼系統 Enigma？](http://www.inside.com.tw/2015/03/03/how-did-alan-turing-figure-out-the-enigma-machine)
 * [YouTube: 158,962,555,217,826,360,000 (Enigma Machine) - Numberphile](https://www.youtube.com/watch?v=G2_Q9FoD-oQ)
 * [YouTube: Flaw in the Enigma Code - Numberphile](https://www.youtube.com/watch?v=V4V2bpZlqx8)

從上面影片中您可以看到，同樣一個字母透過 Enigma 會編成不同的字母。舉例而言， Y 可能這一次編為 Ｗ，下一次又邊為 Ｔ。而且兩個不同的字母，像是 Ｐ和 Ｚ，也有可能都被編成 S。

 Enigma 之所以會有這樣的編碼行為，是因為那些旋轉輪會在每打幾個字之後就旋轉一格，然後前面的輪子在某個時候又會帶動後面的輪子，這些旋轉輪的不同狀態讓同一個字母會被編成不同的碼。

於是整台 Enigma 最難以破解的地方，就是那些旋轉輪所代表的初始狀態，以及反射插板的插線方法，只要知道了這些初始狀態與插線方法，就等於是破解了 Enigma 。

但是、由於旋轉輪與插板的排列組合可能性眾多，因此要破解可沒有那麼容易，我們將在下文當中繼續討論 Enigma 的破解方法。

### 參考文獻
* [恩尼格瑪密碼機](http://zh.wikipedia.org/wiki/%E6%81%A9%E5%B0%BC%E6%A0%BC%E7%8E%9B%E5%AF%86%E7%A0%81%E6%9C%BA) 