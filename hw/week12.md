### week12
## cpu
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/05/CPU.hdl

/**
 * The Hack CPU (Central Processing unit), consisting of an ALU,
 * two registers named A and D, and a program counter named PC.
 * The CPU is designed to fetch and execute instructions written in 
 * the Hack machine language. In particular, functions as follows:
 * Executes the inputted instruction according to the Hack machine 
 * language specification. The D and A in the language specification
 * refer to CPU-resident registers, while M refers to the external
 * memory location addressed by A, i.e. to Memory[A]. The inM input 
 * holds the value of this location. If the current instruction needs 
 * to write a value to M, the value is placed in outM, the address 
 * of the target location is placed in the addressM output, and the 
 * writeM control bit is asserted. (When writeM==0, any value may 
 * appear in outM). The outM and writeM outputs are combinational: 
 * they are affected instantaneously by the execution of the current 
 * instruction. The addressM and pc outputs are clocked: although they 
 * are affected by the execution of the current instruction, they commit 
 * to their new values only in the next time step. If reset==1 then the 
 * CPU jumps to address 0 (i.e. pc is set to 0 in next time step) rather 
 * than to the address resulting from executing the current instruction. 
 */

CHIP CPU {

    IN  inM[16],         // M value input  (M = contents of RAM[A])
        instruction[16], // Instruction for execution
        reset;           // Signals whether to re-start the current
                         // program (reset==1) or continue executing
                         // the current program (reset==0).

    OUT outM[16],        // M value output
        writeM,          // Write to M? 
        addressM[15],    // Address in data memory (of M)
        pc[15];          // address of next instruction

    PARTS:
    // Put your code here:
    //把instruction分成16個，方便書寫程式
    Or16(a=false, b=instruction, out[15]=isC, out[12]=a, out[11]=c1, out[10]=c2, out[9]=c3, out[8]=c4, out[7]=c5, out[6]=c6, out[5]=d1, out[4]=d2, out[3]=d3, out[2]=j1, out[1]=j2, out[0]=j3);
    Mux16(a=instruction, b=aluOut, sel=isC, out=CorA);

    //Aregister-c
    And(a=d1, b=isC, out=Cd1);
    Not(in=isC, out=NC);
    Or(a=NC, b=Cd1, out=loadA);
    ARegister(in=CorA, load=loadA, out=A, out[0..14]=addressM);

    Mux16(a=A, b=inM, sel=a, out=AorM);

    //Dregister-c
    And(a=isC, b=d2, out=loadD);
    DRegister(in=aluOut, load=loadD, out=D);

    ALU(x=D, y=AorM, zx=c1, nx=c2, zy=c3, ny=c4, f=c5, no=c6, out=outM, out=aluOut, zr=zr, ng=ng);
    
    //writeM-c
    And(a=isC, b=d3, out=writeM);

    //PC-c
    Or(a=zr, b=ng, out=ZRorNG); //is input zero or negative
    Not(in=ZRorNG, out=GT); //input is not zero and negative (greater than)
    And(a=GT, b=j3, out=JGT);
    And(a=zr, b=j2, out=JEQ);
    Or(a=JEQ, b=JGT, out=JGE);
    And(a=ng, b=j1, out=JLT);
    Or(a=JLT, b=JGE, out=Pass);
    And(a=isC, b=Pass, out=forPC);
    PC(in=A, load=forPC, inc=true, reset=reset, out[0..14]=pc);
}
</code></pre>
## picture
![picture](https://github.com/hung890202/co109a/blob/master/picture/S__233971728.jpg)