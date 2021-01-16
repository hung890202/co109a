### week7
## 1.ram8
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/03/a/RAM8.hdl

/**
 * Memory of 8 registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then 
 * the in value is loaded into the memory location specified by address 
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM8 {
    IN in[16], load, address[3];
    OUT out[16];

    PARTS:
    // Put your code here:
    DMux8Way(in=load,sel=address,a=R1,b=R2,c=R3,d=R4,e=R5,f=R6,g=R7,h=R8);
    Register(in=in,load=R1,out=r0);
    Register(in=in,load=R2,out=r1);
    Register(in=in,load=R3,out=r2);
    Register(in=in,load=R4,out=r3);
    Register(in=in,load=R5,out=r4);
    Register(in=in,load=R6,out=r5);
    Register(in=in,load=R7,out=r6);
    Register(in=in,load=R8,out=r7);
    Mux8Way16(a=r0,b=r1,c=r2,d=r3,e=r4,f=r5,g=r6,h=r7,sel=address,out=out);
}
</code></pre>
## 2.ram64
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/03/a/RAM64.hdl

/**
 * Memory of 64 registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then 
 * the in value is loaded into the memory location specified by address 
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM64 {
    IN in[16], load, address[6];
    OUT out[16];

    PARTS:
    // Put your code here:
    DMux8Way(in=load,sel=address[3..5],a=R0,b=R1,c=R2,d=R3,e=R4,f=R5,g=R6,h=R7);
    RAM8(in=in,load=R0,address=address[0..2],out=r0);
    RAM8(in=in,load=R1,address=address[0..2],out=r1);
    RAM8(in=in,load=R2,address=address[0..2],out=r2);
    RAM8(in=in,load=R3,address=address[0..2],out=r3);
    RAM8(in=in,load=R4,address=address[0..2],out=r4);
    RAM8(in=in,load=R5,address=address[0..2],out=r5);
    RAM8(in=in,load=R6,address=address[0..2],out=r6);
    RAM8(in=in,load=R7,address=address[0..2],out=r7);
    Mux8Way16(a=r0,b=r1,c=r2,d=r3,e=r4,f=r5,g=r6,h=r7,sel=address[3..5],out=out);

}

</code></pre>
## 3.ram512
code
<pre><code>
// This file is part of the materials accompanying the book 
// "The Elements of Computing Systems" by Nisan and Schocken, 
// MIT Press. Book site: www.idc.ac.il/tecs
// File name: projects/03/b/RAM512.hdl

/**
 * Memory of 512 registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then 
 * the in value is loaded into the memory location specified by address 
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM512 {
    IN in[16], load, address[9];
    OUT out[16];

    PARTS:
    // Put your code here:
    DMux8Way(in=load,sel=address[6..8],a=R0,b=R1,c=R2,d=R3,e=R4,f=R5,g=R6,h=R7);
    RAM64(in=in,load=R0,address=address[0..5],out=r0);
    RAM64(in=in,load=R1,address=address[0..5],out=r1);
    RAM64(in=in,load=R2,address=address[0..5],out=r2);
    RAM64(in=in,load=R3,address=address[0..5],out=r3);
    RAM64(in=in,load=R4,address=address[0..5],out=r4);
    RAM64(in=in,load=R5,address=address[0..5],out=r5);
    RAM64(in=in,load=R6,address=address[0..5],out=r6);
    RAM64(in=in,load=R7,address=address[0..5],out=r7);
    Mux8Way16(a=r0,b=r1,c=r2,d=r3,e=r4,f=r5,g=r6,h=r7,sel=address[6..8],out=out);
}
</code></pre>
## 4.RAM4K
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/03/b/RAM4K.hdl

/**
 * Memory of 4K registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then 
 * the in value is loaded into the memory location specified by address 
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM4K {
    IN in[16], load, address[12];
    OUT out[16];

    PARTS:
    // Put your code here:
    DMux8Way(in=load,sel=address[9..11],a=R0,b=R1,c=R2,d=R3,e=R4,f=R5,g=R6,h=R7);
    RAM512(in=in,load=R0,address=address[0..8],out=r0);
    RAM512(in=in,load=R1,address=address[0..8],out=r1);
    RAM512(in=in,load=R2,address=address[0..8],out=r2);
    RAM512(in=in,load=R3,address=address[0..8],out=r3);
    RAM512(in=in,load=R4,address=address[0..8],out=r4);
    RAM512(in=in,load=R5,address=address[0..8],out=r5);
    RAM512(in=in,load=R6,address=address[0..8],out=r6);
    RAM512(in=in,load=R7,address=address[0..8],out=r7);
    Mux8Way16(a=r0,b=r1,c=r2,d=r3,e=r4,f=r5,g=r6,h=r7,sel=address[9..11],out=out);
}
</code></pre>
## ram16k
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/03/b/RAM16K.hdl

/**
 * Memory of 16K registers, each 16 bit-wide. Out holds the value
 * stored at the memory location specified by address. If load==1, then 
 * the in value is loaded into the memory location specified by address 
 * (the loaded value will be emitted to out from the next time step onward).
 */

CHIP RAM16K {
    IN in[16], load, address[14];
    OUT out[16];

    PARTS:
    // Put your code here:
    DMux4Way(in=load,sel=address[12..13],a=R0,b=R1,c=R2,d=R3);
    RAM4K(in=in,load=R0,address=address[0..11],out=r0);
    RAM4K(in=in,load=R1,address=address[0..11],out=r1);
    RAM4K(in=in,load=R2,address=address[0..11],out=r2);
    RAM4K(in=in,load=R3,address=address[0..11],out=r3);
    Mux4Way16(a=r0,b=r1,c=r2,d=r3,sel=address[12..13],out=out);
}
</code></pre>
## picture
![picture](https://github.com/hung890202/co109a/blob/master/picture/S__233971722.jpg)
![picture](https://github.com/hung890202/co109a/blob/master/picture/S__233971723.jpg)
![picture](https://github.com/hung890202/co109a/blob/master/picture/S__233971724.jpg)
![picture](https://github.com/hung890202/co109a/blob/master/picture/S__233971725.jpg)