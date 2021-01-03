<font face="宋体">

# DSD复习笔记
## Chap2 Verilog 基本概念
* 模块的结构
```verilog
module 模块名(端口列表/端口声明列表);
    端口声明；
    参数声明；
    wire, reg ... 变量声明；
    assign [#delay] LEFT=RIGHT；
    initial, always ... 行为语句；
    低层模块实例语句；
    任务和函数；
endmodule
```
* 利用层次命名引用对象
* 模块内并发执行的语句：
  * assign, 每条连续赋值语句的执行顺序依赖于发生在变量a和b上的事件，**assign不要忘记写！** 多个可以共用assign用逗号连接不过最好分开写
  * always，initial只能用于建模和仿真，**只有reg类型可以在initial/always中被赋值而wire不可以**，**所有initial/always语句在0时刻并发执行**，**顺序过程的延迟时间是累加的**
  * 结构方式：门原语，用户定义原语UDP，模块实例
* 时间：
  * #2指两个时间单位
  * `timescale 1ns/100ps，定义延时单位为1ns，精度为100ps(1ps=10^(-12)s), #2.24表示2.2ns
* 模块内input和模块外output，只能用net类型，inout在模块内外都只能用net
- [ ] 端口位宽匹配, ppt-P26，更高位补零？是否有补1的情况？
- [ ] 4位行波进位计数器，用到了T触发器，D触发器，ppt-P30
  
## Chap3 Verilog语言基础
* 标识符：第一个字母必须是字母或者下划线，区分大小写，不能是关键字
* 基本值：x,z不区分大小写，0x1z和0X1Z相同，**wire默认=z，reg默认=x**
* 常量：整数，实数，字符串
* 整数表示法：十进制数格式，或基数格式：**自己写结果时候基数格式要写全**
  * [size]'[s/S][base][value],
  * d/D-10, b/B-2, o/O-8, h/H-16，缺省为10进制，
  * 值x/z以及十六进制中a-f不区分大小写，
  * 数值部分不能为负，4'd-2(x)
  * 8'h2A，8'和h之间不能有空格
  * 位宽不能使用表达式‘=
  * 补最高位：4'bX0=4'bxxx0, 4'hZ=4'bzzzz
  * 整数中？可以替代z
  * 下划线，不能作为首字符，不能出现在位宽和进制处，8’b_0101_1100(x)
  * 默认位宽和机器有关，10, 'bx
* 实数
  * 6.(x), 小数点两侧必须有一位数字
  * 科学计数法
* 字符串
  * 双引号
  * 不能多行书写
  * 每个字母用8位ASCII值表示‘=
* 参数parameter表示常量，可以重载,表达式右边必须是常数表达式，只能是数字和已经定义过的参数，用于定义延迟时间和变量宽度
```verilog
comb #(.N(8)) m_comb(...);
```
* 局部参数localparam，值不能重载修改，状态机的状态编码
* tri，线网类型，多个驱动源
```verilog
tri y;
assign y=a&b;
assign y=a^b;
// a&b=01x, a^b=11z,->y=x1x
//0-0=0,1-1=1,0-z=0,1-z=1,z-z=z,ow=x
```
* reg
  * 变量的值被解释为无符号数，负数以二进制补码保存
  ```verilog
  reg [3:0] tag;
  tag=-2; //保存的是4'd14=4'b1110, 0010->1101->1110
  ```
  * 声明为有符号类型变量
  ```verilog
  reg [4:0] x;
  reg [4:0] y;
  initial begin
   x=5;
   $display("x=%5b",x); //x=00101
   y=-x;
   $display("y=%5b",y) //y=11011, 补码，00101->11010->11011
   $display("y=%d",y) //y=27, 默认解释为无符号整数
  end 

  reg [4:0] x;
  reg signed [4:0] y; //signed 的位置在wire/reg 和位宽之间
  initial begin
   x=5;
   $display("x=%5b",x); //x=00101
   y=-x;
   $display("y=%5b",y) //y=11011, 补码，00101->11010->11011
   $display("y=%d",y) //y=-5
  end 
  ```
* 部分位选择，高低顺序一致
  ```verilog
  reg [255:0] data1;
  reg [0:255] data2;
  reg [7:0] byte;
  byte = data1[31-:8]; //31:24
  byte = data1[24+:8]; //31:24
  byte = data2[31-:8]; //24:31
  byte = data2[24+:8]; //24:31
  ```
* 寄存器类型：reg,integer,real,time,realtime
* 存储器类型memory，寄存器数组，可以用索引进行部分位选择
```verilog
parameter WSIZE=64, MSIZE=64;
reg [WSIZE-1:0] mem[MSIZE-1:0];
reg [7:0] mem[0:63]; //64x8为存储器
mem = 0; //x
mem[1] = 0; 
```
* 数组，声明各种类型的数组：reg,integer,time,real,realtime,wire，区分位宽和数组维度，不能对整个数组赋值，不能对数组一整行赋值
* 表达式：操作数和操作符
* 算术运算符，+-*/%
  * / 整数除法，截断小数部分
  * % 取模，两个操作数均为整数，*求出与第一个操作符号相同的余数，两个绝对值做运算*
  * **如果算术运算符中的任意操作数是x或z，则整个结果为x**
  ```verilog
  5'b0_10x1+5'b0_1111=5'bx_xxxx;
  5'b1_z010+5'b0_0010=5'bx_xxxx;
  ```
  * **独立表达式中，运算结果的长度由最长的操作数决定；赋值语句中，由左边目标的长度决定**
  ```verilog
  wire[4:1] a,b;
  wire[5:1] c;
  wire[6:1] d;
  wire[8:1] F;
  if((a+c)+(b+d)) //(a+c)结果为6位，因为看整个表达式
  F=(a+c)+(b+d); //中间结果(a+c)结果为8位
  ```
  * 无符号数和有符号数
    * 无符号数：wire,reg；有符号数：integer, reg signed, wire signed
    * 表达式有符号/无符号只依赖于操作数，不依赖于左边被赋值变量
    * 十进制数是有符号数
    * 带符号基数格式，4'sd12
    * 位选择和部分位选择是无符号数
    ```verilog
    reg [5:0] bar;
    bar = -4'd12; // 001100->110011->110100->52

    integer tab;
    tab = -4'd12; // 32bit，同样存储补码
    ```
* 位运算符
  * **~ 按位取反**
  * & 按位与
  * | 按位或
  * **^ 按位异或**
  * **^~ 按位同或**
  * **若操作数长度不同，长度较小的操作数在最左边添0**
  * **位运算规则**
    * ~x=x, ~z=x
    * & 两个操作数中有一个是0，结果为0(包括出现x/z)，1&1=1，其余都为x
    * | 两个操作数中有一个是1，结果为1(包括出现x/z)，0|0=0，其余都为x
    * ^, ^~, 两个操作数中出现x/z，则结果为x
* 逻辑运算符
  * **用于条件判断，总是相当于1位二进制位运算**
  * **&&， ||， ！**
  * **对于向量的逻辑运算，非0向量作为1处理**
  ```verilog
  reg flag;
  reg [3:0] val_a, val_b;
  reg [3:0] a,b,c,d,e,f;

  inital begin
    val_a = 4'b0110; //比如在initial/always语句中赋值
    val_b = 4'b0000;
    c = !val_a; //c=0000
    d = !val_b; //d=0001, 高位填充0
    flag=1'bx;
    e=!flag; //e=000x
    f=a+e; //f=xxxx
  end
  ```
* 关系运算符
  * **如果操作数中有一位为x或z，那么结果为x**
* 相等关系运算符
  * ==, !=, ===, !==
  * **如果操作数中有一位为x或z，那么结果为x**
  * **如果操作数长度不等，长度较小的操作数左边添0补位**
  ```verilog
  data='b11x0;
  addr='b11x0;
  data==addr //x
  data===addr //1
  ```
* 移位运算符
  * 如果右侧操作数的值为x/z，移位操作结果为x
  * 用0填补移出的空位
* 拼接/连接运算符
  * 可以用作左值
  ```verilog
  assign bus0={bus[3:0],bus0[7:4]};
  {bus0,5} //x, 不允许使用非定长数
  ```
* 重复复制
  ```verilog
  abus = {3{4'b1011}};
  ```
* 缩减/归约运算符
  * &, ~&, |, ~|, ^, ~^, 这里带~的运算符得到相反结果
  * 如果某一位为x/z，则结果为x，可以用^确定向量中是否有位不确定
  ```verilog
  `timescale 1ns/1ns
  module logic_eval();
    reg [3:0] a;
    reg [3:0] data;
    initial begin
      a = 4'b0;        //d=4'b0000
      #5 a = 4'b1;     //d=4'b0001
      #5 a = 4'bz0x1;  //d=4'b0001
      #5 a = 4'b1z00;  //d=4'b0001
      #5 a = 4'b0zx0;  //d=4'b000x
      #5 a = 4'b0xx0;  //d=4'b000x
      #5 a = 4'bx;     //d=4'b000x
      #5 a = 4'bz;     //d=4'b000x
    end
    always @(*) data = !(!a);
    //逻辑运算符！先对向量做按位或|缩减?
    initial
    $monitor("At %2t time, a=%4b, data=%4b", $time, a, data);
  endmodule
  ```
* 条件运算符
  ```verilog
  data_out = (a) ? 4'b110x : 4'b1000;
  //如果 a 为真，则 data_out = 4’b110x
  //如果 a 为假，则 data_out = 4’b1000
  //如果 a 为 x，则 data_out = 4’b1x0x
  //如果 a 为 z，则 data_out = 4’b1x0x, 求同存异
  ```
* 运算优先级

## Chap4 门级建模

  </font>
