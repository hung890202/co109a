### week10
code
<pre><code>
// This file is part of www.nand2tetris.org
// and the book "The Elements of Computing Systems"
// by Nisan and Schocken, MIT Press.
// File name: projects/04/Fill.asm

// Runs an infinite loop that listens to the keyboard input.
// When a key is pressed (any key), the program blackens the screen,
// i.e. writes "black" in every pixel;
// the screen should remain fully black as long as the key is pressed. 
// When no key is pressed, the program clears the screen, i.e. writes
// "white" in every pixel;
// the screen should remain fully clear as long as no key is pressed.

// Put your code here.
//16384到24575 為螢幕
//24576為鍵盤
@16383
D=A //將值先輸入D
@20
M=D //R20存入控制螢幕初始值
@21
M=D //R21存入控制螢幕初始值
@8192
D=A //存入計數器控制螢幕反白
M=D
@8193
M=D //存入計數器控制螢幕變黑
@24576
D=M 
@27
D;JEQ //當沒按鍵盤跳到迴圈(LOOP1反白)
@20   //當按鍵盤跳到迴圈(LOOP2反黑)[迴圈LOOP2開頭]
M=M+1 //A+1(16383+1......)
D=M   
A=D
M=-1 //反黑指令
@8193 
M=M-1 //計數器-1
D=M
@39  
D;JEQ //當計數器0跳出迴圈LOOP2
@15
0;JMP //回到LOOP2迴圈
@21 //當沒按鍵盤跳到迴圈(LOOP1反白)[迴圈LOOP2開頭]
M=M+1 //A+1(16383+1......)
D=M
A=D
M=0 //反白指令
@8192
M=M-1 //計數器-1
D=M
@39
D;JEQ //當計數器0跳出迴圈LOOP1
@27
0;JMP //回到LOOP1迴圈
@0
0;JMP //回到主迴圈指令1
</code></pre>
## picture
![picture](https://github.com/hung890202/co109a/blob/master/picture/24.jpg)
![picture](https://github.com/hung890202/co109a/blob/master/picture/25.jpg)