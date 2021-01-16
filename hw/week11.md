### week11
## memory
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/05/Memory.hdl

/**
 * The complete address space of the Hack computer's memory,
 * including RAM and memory-mapped I/O. 
 * The chip facilitates read and write operations, as follows:
 *     Read:  out(t) = Memory[address(t)](t)
 *     Write: if load(t-1) then Memory[address(t-1)](t) = in(t-1)
 * In words: the chip always outputs the value stored at the memory 
 * location specified by address. If load==1, the in value is loaded 
 * into the memory location specified by address. This value becomes 
 * available through the out output from the next time step onward.
 * Address space rules:
 * Only the upper 16K+8K+1 words of the Memory chip are used. 
 * Access to address>0x6000 is invalid. Access to any address in 
 * the range 0x4000-0x5FFF results in accessing the screen memory 
 * map. Access to address 0x6000 results in accessing the keyboard 
 * memory map. The behavior in these addresses is described in the 
 * Screen and Keyboard chip specifications given in the book.
 */

CHIP Memory {
    IN in[16], load, address[15];
    OUT out[16];

    PARTS:
    // Put your code here:
    Not(in=address[14],out=NR);
    And(a=NR,b=load,out=RAML);
    And(a=address[14],b=load,out=ScrL);
    RAM16K(in=in,load=RAML,address=address[0..13],out=Rout);
    Screen(in=in,load=ScrL,address=address[0..12],out=Sout);
    Keyboard(out=Kout);
    Mux16(a=Sout,b=Kout,sel=address[13],out=SKout);
    Mux16(a=Rout,b=SKout,sel=address[14],out=out);
}
</code></pre>
## picture
![picture](https://github.com/hung890202/co109a/blob/master/picture/S__233971727.jpg)