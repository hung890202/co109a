### week14
## Makefile
code
<pre><code>
CC := gcc -std=c99
CPPC := g++ -std=c++11
TARGET = asm vm dasm

all: $(TARGET)

asm: asm.cpp
	$(CPPC) $^ -o $@

vm: vm.c
	$(CC) $^ -o $@

dasm: dasm.c
	$(CC) $^ -o $@

clean:
	rm -f *.o *.exe $(TARGET)
</code></pre>
## vm.c

CODE
<pre><code>
#include 
#include 
#include 
#include 
#include 
#include 

#define SIZE(list) (sizeof(list)/sizeof(list[0]))
#define DEBUG(...) printf(__VA_ARGS__)
#define BIT(i) (0x0001<<(i))

int imTop = 0;
uint16_t im[32768];
int16_t  m[65536];

void run(uint16_t *im, int16_t *m) {
  int16_t D = 0, A = 0, PC = 0;
  uint16_t I = 0;
  uint16_t a, c, d, j;
  while (1) {
    int16_t aluOut = 0, AM = 0;
    if (PC >= imTop) { 
      DEBUG("exit program !\n");
      break;
    }
    I = im[PC];
    DEBUG("PC=%04X I=%04X", PC, I);
    PC ++;
    if ((I & 0x8000) == 0) { // A 指令
      A = I;
    } else { // C 指令
      a = (I & 0x1000) >> 12;
      c = (I & 0x0FC0) >>  6;
      d = (I & 0x0038) >>  3;
      j = (I & 0x0007) >>  0;

      AM = (a == 0) ? A : m[A];
      switch (c) { // 處理 c1..6, 計算 aluOut
        case 0x2A: aluOut = 0;      break; // "0",   "101010"
        case 0x3F: aluOut = 1;      break; // "1",   "111111"
        case 0x3A: aluOut = -1;     break; // "-1",  "111010"
        case 0x0C: aluOut = D;      break; // "D",   "001100"
        case 0x30: aluOut = AM;     break; // "AM",  "110000"
        case 0x0D: aluOut = D ^ 0xFFFF; break; // "!D",  "001101"
        case 0x31: aluOut = AM ^ 0xFFFF; break; // "!AM", "110001"
        case 0x0F: aluOut = -D;     break; // "-D",  "001111"
        case 0x33: aluOut = -AM;    break; // "-AM", "110011"
        case 0x1F: aluOut = D + 1;  break; // "D+1", "011111"
        case 0x37: aluOut = AM + 1; break; // "AM+1","110111"
        case 0x0E: aluOut = D - 1;  break; // "D-1", "001110"
        case 0x32: aluOut = AM - 1; break; // "AM-1","110010"
        case 0x02: aluOut = D + AM; break; // "D+AM","000010"
        case 0x13: aluOut = D - AM; break; // "D-AM","010011"
        case 0x07: aluOut = AM - D; break; // "AM-D","000111"
        case 0x00: aluOut = D & AM; break; // "D&AM","000000"
        case 0x15: aluOut = D | AM; break; // "D|AM","010101"
        default: assert(0);
      }
      if (d & BIT(2)) A = aluOut;
      if (d & BIT(1)) D = aluOut;
      if (d & BIT(0)) m[A] = aluOut;
      switch (j) {
        case 0x0: break;                          // 
        case 0x1: if (aluOut >  0) PC = A; break; // JGT
        case 0x2: if (aluOut == 0) PC = A; break; // JEQ
        case 0x3: if (aluOut >= 0) PC = A; break; // JGE
        case 0x4: if (aluOut <  0) PC = A; break; // JLT
        case 0x5: if (aluOut != 0) PC = A; break; // JNE
        case 0x6: if (aluOut <= 0) PC = A; break; // JLE
        case 0x7: PC = A; break;                  // JMP
      }
    }
    DEBUG(" A=%04X D=%04X m[A]=%04X=%04d", A, D, m[A], m[A]);
    if ((I & 0x8000) != 0) DEBUG(" a=%X c=%02X d=%X j=%X", a, c, d, j);
    DEBUG("\n");
  }
}

// run: ./vm 
int main(int argc, char *argv[]) {
  char *binFileName = argv[1];
  FILE *binFile = fopen(binFileName, "rb");
  imTop = fread(im, sizeof(uint16_t), 32768, binFile);
  fclose(binFile);
  run(im, m);
}
</code></pre>
## dasm.c

CODE
<pre><code>
#include 
#include 
#include 
#include 
#include 
#include 

#define SIZE(list) (sizeof(list)/sizeof(list[0]))
#define DEBUG(...) printf(__VA_ARGS__)
#define BIT(i) (0x0001<<(i))

int imTop = 0;
uint16_t im[32768];
char *jTable[] = {"","JGT","JEQ","JGE", "JLT", "JNE", "JLE", "JMP"};
char *dTable[] = {"", "M", "D",  "MD",  "A",   "AM",  "AD",  "AMD"};

void disasm(uint16_t *im, int16_t imTop) {
  uint16_t I = 0;
  uint16_t a, c, d, j;
  char AM = ' ', op[10] = "";
  for (int PC = 0; PC> 12;
      c = (I & 0x0FC0) >>  6;
      d = (I & 0x0038) >>  3;
      j = (I & 0x0007) >>  0;
      AM = (a == 0) ? 'A' : 'M';
      switch (c) { // 處理 c1..6, 計算 aluOut
        case 0x2A: sprintf(op, "0");         break; // "0",   "101010"
        case 0x3F: sprintf(op, "1");         break; // "1",   "111111"
        case 0x3A: sprintf(op, "-1");        break; // "-1",  "111010"
        case 0x0C: sprintf(op, "D");         break; // "D",   "001100"
        case 0x30: sprintf(op, "%c", AM);    break; // "AM",  "110000"
        case 0x0D: sprintf(op, "!D");        break; // "!D",  "001101"
        case 0x31: sprintf(op, "!%c", AM);   break; // "!AM", "110001"
        case 0x0F: sprintf(op, "-D");        break; // "-D",  "001111"
        case 0x33: sprintf(op, "-%c", AM);   break; // "-AM", "110011"
        case 0x1F: sprintf(op, "D+1");       break; // "D+1", "011111"
        case 0x37: sprintf(op, "%c+1", AM);  break; // "AM+1","110111"
        case 0x0E: sprintf(op, "D-1");       break; // "D-1", "001110"
        case 0x32: sprintf(op, "%c-1", AM);  break; // "AM-1","110010"
        case 0x02: sprintf(op, "D+%c", AM);  break; // "D+AM","000010"
        case 0x13: sprintf(op, "D-%c", AM);  break; // "D-AM","010011"
        case 0x07: sprintf(op, "%c-D", AM);  break; // "AM-D","000111"
        case 0x00: sprintf(op, "D&%c", AM);  break; // "D&AM","000000"
        case 0x15: sprintf(op, "D|%c", AM);  break; // "D|AM","010101"
        default: assert(0);
      }
      char head[100]="", command[100];
      if (d != 0) sprintf(head, "%s=", dTable[d]);
      if (op[0] != '\0') strcat(head, op);
      if (j != 0) 
        sprintf(command, "%s;%s", head, jTable[j]);
      else
        sprintf(command, "%s", head);
      printf("%s\n", command);
    }
  }
}

// run: ./disasm 
int main(int argc, char *argv[]) {
  char *binFileName = argv[1];
  FILE *binFile = fopen(binFileName, "rb");
  imTop = fread(im, sizeof(uint16_t), 32768, binFile);
  fclose(binFile);
  disasm(im, imTop);
  return 0;
}
</code></pre>
## asm.c

CODE
<pre><code>
#include 
#include 

typedef struct {
    char cnd[];
    char val[];
    char name[];
    int num;
}map;

map dMap[8] = 
{[0]={.cnd = "", .val = "000"},
 [1]={.cnd = "M", .val = "001"},
 [2]={.cnd = "D", .val = "010"},
 [3]={.cnd = "MD", .val = "011"},
 [4]={.cnd = "A", .val = "100"},
 [5]={.cnd = "AM", .val = "101"},
 [6]={.cnd = "AD", .val = "110"},
 [7]={.cnd = "AMD", .val = "111"},}

map cMap[28] =
{[0]={.cnd = "0", .val = "0101010"},
 [1]={.cnd = "1", .val = "0101010"},
 [2]={.cnd = "-1", .val = "0101010"}, 
 [3]={.cnd = "D", .val = "0101010"},
 [4]={.cnd = "A", .val = "0101010"},
 [5]={.cnd = "!D", .val = "0101010"},
 [6]={.cnd = "!A", .val = "0101010"},
 [7]={.cnd = "-D", .val = "0101010"},
 [8]={.cnd = "-A", .val = "0101010"},
 [9]={.cnd = "D+1", .val = "0101010"},
 [10]={.cnd = "A+1", .val = "0101010"},
 [11]={.cnd = "D-1", .val = "0101010"},
 [12]={.cnd = "A-1", .val = "0101010"},
 [13]={.cnd = "D+A", .val = "0101010"},
 [14]={.cnd = "D-A", .val = "0101010"},
 [15]={.cnd = "A-D", .val = "0101010"},
 [16]={.cnd = "D&A", .val = "0101010"},
 [17]={.cnd = "D|A", .val = "0101010"},
 [18]={.cnd = "M", .val = "0101010"},
 [19]={.cnd = "!M", .val = "0101010"},
 [20]={.cnd = "-M", .val = "0101010"},
 [21]={.cnd = "M+1", .val = "0101010"},
 [22]={.cnd = "M-1", .val = "0101010"},
 [23]={.cnd = "D+M", .val = "0101010"},
 [24]={.cnd = "D-M", .val = "0101010"},
 [25]={.cnd = "M-D", .val = "0101010"},
 [26]={.cnd = "D&M", .val = "0101010"},
 [27]={.cnd = "D|M", .val = "0101010"}}

map jMap[8] = 
{[0]={.cnd = "", .val = "000"},
 [1]={.cnd = "JGT", .val = "001"},
 [2]={.cnd = "JEQ", .val = "010"},
 [3]={.cnd = "JGE", .val = "011"},
 [4]={.cnd = "JLT", .val = "100"},
 [5]={.cnd = "JNE", .val = "101"},
 [6]={.cnd = "JLE", .val = "110"},
 [7]={.cnd = "JMP", .val = "111"}}

map symMap[23] = 
{[0]={.name = "R0", .num = 0},
 [1]={.name = "R1", .num = 1},
 [2]={.name = "R2", .num = 2},
 [3]={.name = "R3", .num = 3},
 [4]={.name = "R4", .num = 4},
 [5]={.name = "R5", .num = 5},
 [6]={.name = "R6", .num = 6},
 [7]={.name = "R7", .num = 7},
 [8]={.name = "R8", .num = 8},
 [9]={.name = "R9", .num = 9},
 [10]={.name = "R10", .num = 10},
 [11]={.name = "R11", .num = 11},
 [12]={.name = "R12", .num = 12},
 [13]={.name = "R13", .num = 13},
 [14]={.name = "R14", .num = 14},
 [15]={.name = "R15", .num = 15},
 [16]={.name = "SCREEN", .num = 16384},
 [17]={.name = "KBD", .num = 24576},
 [18]={.name = "SP", .num = 0},
 [19]={.name = "LCL", .num = 0},
 [20]={.name = "ARG", .num = 2},
 [21]={.name = "THIS", .num = 3},
 [22]={.name = "THAT", .num = 4}}

void symDump(map *inputmap){
    printf("=================symMap===================\n");
    for(int i=0;i 0; i--) {
        *bin-- = (a & 1) + '0';
        a >>= 1;
    }
    return bin;
}

int bin2int(char *bin) {
  return (int) strtol(bin, NULL, 2);
}

void code2binary(char *code, char *binary) {
  if (code[0]=='@') { // A 指令： ＠number || @symbol
    int address;
    int match = sscanf(code, "@%d", &address);
    if (match == 1)
      int2bin(address, binary, 16);
    else {
      char symbol[100];
      match = sscanf(code, "@%s", symbol);
      string sym(symbol);
      auto it = symMap.find(sym);
      if (it == symMap.end()) { // 宣告變數
        symMap[sym] = varTop; // 新增一個變數
        address = varTop++;
      } else { // 已知變數 (標記) 位址
        address = it->second; // 取值。
      }
      int2bin(address, binary, 16);
    }
  } else { // C 指令
    char d[10], comp[100], j[10];
    string dcode, ccode, jcode;
    if (strchr(code, '=') != NULL) { // d=comp
      sscanf(code, "%[^=]=%s", d, comp);
      string ds(d), comps(comp);
      dcode = dMap[ds];
      ccode = cMap[comps];
      sprintf(binary, "111%s%s000", ccode.c_str(), dcode.c_str());
    } else {
      sscanf(code, "%[^;];%s", comp, j); // comp;j
      string comps(comp), js(j);
      ccode = cMap[comps];
      jcode = jMap[js];
      sprintf(binary, "111%s000%s", ccode.c_str(), jcode.c_str());
    }
  }
}

void pass2(char* inFile, char* hackFile, char* binFile) {
  printf("============= PASS2 ================\n");
  char line[100], binary[17];
  FILE *fp = fopen(inFile, "r"); // 開啟組合語言檔
  FILE *hfp = fopen(hackFile, "w"); // 開啟輸出的 .hack 二進位字串檔案
  FILE *bfp = fopen(binFile, "wb"); // 開啟輸出的 .bin 二進位檔
  int address = 0;
  while (fgets(line, sizeof(line), fp)) { // 一行一行讀
    char *code = parse(line); // 取得該行的程式碼部分
    if (strlen(code)==0) continue;
    if (line[0] == '(') { // 這行是符號 ex: (LOOP)
      printf("%s\n", line); // 印出該符號
    } else {
      code2binary(code, binary); // 將指令編碼為二進位字串 string
      uint16_t b = bin2int(binary); // 將二進位字串 string 轉成 int16
      printf("%02X: %-20s %s %04x\n", address, code, binary, b);
      fprintf(hfp, "%s\n", binary); // 輸出 .hack 的二進位字串檔
      fwrite(&b, sizeof(b), 1, bfp); // 輸出 .bin 的二進位檔
      address ++;
    }
  }
  fclose(fp);
  fclose(hfp);
  fclose(bfp);
}

void assemble(char *file)
{
    char inFile[100], hackFile[100], binFIle[100];
    sprintf(inFile , "%s.asm", file);
    sprintf(hackFile, "%s.hack", file);
    sprintf(binFile, "%s.bin", file);
    +// symDump(&symMap);
    pass1(inFile);
    // symDump(&symMap);
    pass2(inFile, hackFile, binFile);
}
int main(int argc, char *argv[])
{
    assemble(argc[]);
}
</code></pre>
## ./asm MaxL(執行結果)
<pre><code>

============= PASS1 ================
00:@R0
01:D=M
02:@R1
03:D=D-M
04:@OUTPUT_FIRST    
05:D;JGT
06:@R1
07:D=M
08:@OUTPUT_D        
09:0;JMP
10:(OUTPUT_FIRST)
symbol:OUTPUT_FIRST address=10
10:@R0
11:D=M
12:(OUTPUT_D)
symbol:OUTPUT_D address=12
12:@R2
13:M=D
14:(INFINITE_LOOP)
symbol:INFINITE_LOOP address=14
14:@INFINITE_LOOP
15:0;JMP
============= PASS2 ================
00: @R0                  0000000000000000 0000
01: D=M                  1111110000010000 fc10
02: @R1                  0000000000000001 0001
03: D=D-M                1111010011010000 f4d0
04: @OUTPUT_FIRST        0000000000001010 000a
05: D;JGT                1110001100000001 e301
06: @R1                  0000000000000001 0001
07: D=M                  1111110000010000 fc10
08: @OUTPUT_D            0000000000001100 000c
09: 0;JMP                1110101010000111 ea87
(OUTPUT_FIRST)
0A: @R0                  0000000000000000 0000
0B: D=M                  1111110000010000 fc10
(OUTPUT_D)
0C: @R2                  0000000000000010 0002
0D: M=D                  1110001100001000 e308
(INFINITE_LOOP)
0E: @INFINITE_LOOP       0000000000001110 000e
0F: 0;JMP                1110101010000111 ea87
</code></pre>
## ./asm RectL
<pre><code>

============= PASS1 ================
00:@0
01:D=M
02:@23
03:D;JLE
04:@16
05:M=D
06:@16384
07:D=A
08:@17
09:M=D
10:@17
11:A=M
12:M=-1
13:@17
14:D=M
15:@32
16:D=D+A
17:@17
18:M=D
19:@16
20:MD=M-1
21:@10
22:D;JGT
23:@23
24:0;JMP
============= PASS2 ================
00: @0                   0000000000000000 0000
01: D=M                  1111110000010000 fc10
02: @23                  0000000000010111 0017
03: D;JLE                1110001100000110 e306
04: @16                  0000000000010000 0010
05: M=D                  1110001100001000 e308
06: @16384               0100000000000000 4000
07: D=A                  1110110000010000 ec10
08: @17                  0000000000010001 0011
09: M=D                  1110001100001000 e308
0A: @17                  0000000000010001 0011
0B: A=M                  1111110000100000 fc20
0C: M=-1                 1110111010001000 ee88
0D: @17                  0000000000010001 0011
0E: D=M                  1111110000010000 fc10
0F: @32                  0000000000100000 0020
10: D=D+A                1110000010010000 e090
11: @17                  0000000000010001 0011
12: M=D                  1110001100001000 e308
13: @16                  0000000000010000 0010
14: MD=M-1               1111110010011000 fc98
15: @10                  0000000000001010 000a
16: D;JGT                1110001100000001 e301
17: @23                  0000000000010111 0017
18: 0;JMP                1110101010000111 ea87
</code></pre>
## ./asm PongL (顯示部分)
<pre><code>

============= PASS1 ================
00:@256
01:D=A
02:@SP
03:M=D
04:@133
05:0;JMP
06:@R15
07:M=D
08:@SP
09:AM=M-1
10:D=M
11:A=A-1
12:D=M-D
13:M=0
14:@END_EQ
15:D;JNE
16:@SP
17:A=M-1
18:M=-1
...
============= PASS2 ================
00:@256                  0000000100000000
01:D=A                   1110110000010000
02:@SP                   0000000000000000
03:M=D                   1110001100001000
04:@133                  0000000010000101
05:0;JMP                 1110101010000111
06:@R15                  0000000000001111
07:M=D                   1110001100001000
08:@SP                   0000000000000000
09:AM=M-1                1111110010101000
10:D=M                   1111110000010000
11:A=A-1                 1110110010100000
12:D=M-D                 1111000111010000
13:M=0                   1110101010001000
14:@END_EQ               0000000000010011
15:D;JNE                 1110001100000101
16:@SP                   0000000000000000
17:A=M-1                 1111110010100000
18:M=-1                  1110111010001000
...
</code></pre>
## ./vm PongL.asm (顯示部分)
<pre><code>

PC=058F I=0D31 A=0D31 D=%04 m[A]=0000=0000
PC=0590 I=440A A=440A D=%04 m[A]=0000=0000
PC=0591 I=4D3D A=4D3D D=%04 m[A]=0000=0000
PC=0592 I=0A0D A=0A0D D=%04 m[A]=0000=0000
PC=0593 I=3540 A=3540 D=%04 m[A]=0000=0000
PC=0594 I=0A0D A=0A0D D=%04 m[A]=0000=0000
PC=0595 I=3D4D A=3D4D D=%04 m[A]=0000=0000
PC=0596 I=0D44 A=0D44 D=%04 m[A]=0000=0000
PC=0597 I=400A A=400A D=%04 m[A]=0000=0000
PC=0598 I=0D33 A=0D33 D=%04 m[A]=0000=0000
PC=0599 I=440A A=440A D=%04 m[A]=0000=0000
PC=059A I=4D3D A=4D3D D=%04 m[A]=0000=0000
PC=059B I=0A0D A=0A0D D=%04 m[A]=0000=0000
PC=059C I=3040 A=3040 D=%04 m[A]=0000=0000
PC=059D I=0A0D A=0A0D D=%04 m[A]=0000=0000
PC=059E I=4D41 A=4D41 D=%04 m[A]=0000=0000
...
</code></pre>
## ./vm RectL.asm (顯示部分)
<pre><code>

PC=0000 I=2F2F A=2F2F D=%04 m[A]=0000=0000
PC=0001 I=5420 A=5420 D=%04 m[A]=0000=0000
PC=0002 I=6968 A=6968 D=%04 m[A]=0000=0000
PC=0003 I=2073 A=2073 D=%04 m[A]=0000=0000
PC=0004 I=6966 A=6966 D=%04 m[A]=0000=0000
PC=0005 I=656C A=656C D=%04 m[A]=0000=0000
PC=0006 I=6920 A=6920 D=%04 m[A]=0000=0000
PC=0007 I=2073 A=2073 D=%04 m[A]=0000=0000
PC=0008 I=6170 A=6170 D=%04 m[A]=0000=0000
PC=0009 I=7472 A=7472 D=%04 m[A]=0000=0000
PC=000A I=6F20 A=6F20 D=%04 m[A]=0000=0000
PC=000B I=2066 A=2066 D=%04 m[A]=0000=0000
PC=000C I=7777 A=7777 D=%04 m[A]=0000=0000
PC=000D I=2E77 A=2E77 D=%04 m[A]=0000=0000
PC=000E I=616E A=616E D=%04 m[A]=0000=0000
PC=000F I=646E A=646E D=%04 m[A]=0000=0000
</code></pre>