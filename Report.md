# 数字系统设计作业报告
* 学号：518021910628
* 姓名：吴小茜

<!-- 设计模块，测试模块，波形图，显示输出，设计说明 -->

## EXP2-1
#### 设计/测试模块
```
// File: wavegen.v

`timescale 10 ns / 1 ns

module wavegen;

    reg clk;

    initial begin
        clk = 1'b0;
        #2 clk = 1'b1;
        #1 clk = 1'b0;
        #9 clk = 1'b1;
        #10 clk = 1'b0;
        #2 clk = 1'b1;
        #3 clk = 1'b0;
        #5 clk = 1'b1;
	#20 $stop;
    end

endmodule
```
#### 仿真结果
![exp1.png](Simulation_results/exp1.png)

## EXP2-2
#### 设计模块
```
// File: encoder8x3.v 
module Encoder8x3( output reg[2:0] code, input [7:0] data ); 
 always @(*) 
 begin 
 case ( data ) 
 8'b0000_0001: code = 3'd0; 
 8'b0000_0010: code = 3'd1; 
 8'b0000_0100: code = 3'd2; 
 8'b0000_1000: code = 3'd3; 
 8'b0001_0000: code = 3'd4; 
 8'b0010_0000: code = 3'd5; 
 8'b0100_0000: code = 3'd6; 
 8'b1000_0000: code = 3'd7; 
 default: code = 3'bx; 
 endcase 
 end 
endmodule 
```
#### 测试模块
```
// File: tb_encoder8x3.v

`include "encoder8x3.v"

module tb_Encoder8x3;

    parameter STEP = 9;

    wire [2:0] code;
    reg [7:0] data;
    integer k;

    Encoder8x3 a(code, data);

    initial begin
        data = 8'b0000_0001;
        for ( k=1; k<STEP; k=k+1 )
            #1 data = data << 1;
    end

    initial begin
        $monitor("At time %4t, data=%b, code=%d",
                    $time, data, code);
    end

endmodule
```
#### 仿真结果
![exp2-1.png](Simulation_results/exp2-1.png)
![exp2-2.png](Simulation_results/exp2-2.png)

## EXP2-3
### a) 2选1多路选择器
#### 设计模块
```
// File: mux2x1.v
module mux2x1( dout, sel, din );
 output dout;
 input sel;
 input [1:0] din;
 bufif1 b2( dout, din[1], sel );
 bufif0 b1( dout, din[0], sel );
endmodule
```

#### 仿真模块
```
// File: tb_mux2x1.v
`timescale 10ns / 1ns
`include "mux2x1.v"

module tb_mux2x1();
 reg [2:0] p_data;
 wire p_dout;
 integer i;
 initial begin
 p_data = 3'bx;
 #5 p_data = 3'b0;
 for ( i = 1; i < 9; i = i + 1 )
  #5 p_data = p_data + 3'b1;
 end

 mux2x1 m0( .dout( p_dout ), .sel(p_data[2]), .din(p_data[1:0]) );
 initial
 $monitor( "At time %t, dout=%1b, sel=%1b, din=%2b", $time, p_dout, p_data[2], p_data[1:0]);

endmodule
```

#### 仿真结果
![exp3-1.png](Simulation_results/exp3-1.png)
![exp3-2.png](Simulation_results/exp3-2.png)

### b) 4选1多路选择器
#### 设计模块
```
// File: mux4x1.v

`include "mux2x1.v"

module mux4x1(dout, sel, din);

    output dout;
    input [1:0] sel;
    input [3:0] din;

    wire ta, tb;

    mux2x1 a(ta, sel[0], din[1:0]),
           b(tb, sel[0], din[3:2]),
           c(dout, sel[1], {tb, ta});

endmodule
```

#### 仿真模块
```
// File: tb_mux4x1.v
`timescale 10ns / 1ns
`include "mux4x1.v"

module tb_mux4x1();
 reg [5:0] p_data;
 wire p_dout;
 integer i;
 initial begin
 p_data = 6'bx;
 #5 p_data = 6'b0;
 for ( i = 1; i < 65; i = i + 1 )
  #5 p_data = p_data + 6'b1;
 end

 mux4x1 m0( .dout( p_dout ), .sel(p_data[5:4]), .din(p_data[3:0]) );
 initial
 $monitor( "At time %t, dout=%1b, sel=%2b, din=%4b", $time, p_dout, p_data[5:4], p_data[3:0]);

endmodule
```

#### 仿真结果
![exp3-3.png](Simulation_results/exp3-3.png)
![exp3-4.png](Simulation_results/exp3-4.png)
![exp3-5.png](Simulation_results/exp3-5.png)
![exp3-6.png](Simulation_results/exp3-6.png)

## EXP2-4
#### 设计模块——利用门原语设计
```
// File: comb_str.v
module comb_str(output Y, input A,B,C,D);
 wire M1,M2,M3,M4;
 not u1(M1,D);
 and u4(M4,B,C,M1);
 or u3(M3,A,D);
 not u2(M2,M3);
 and u5(Y,M2,M4);
endmodule
```
#### 设计模块——利用连续赋值语句设计
```
//File comb_dataflow.v
module comb_dataflow(output Y, input A,B,C,D);
 assign Y=(~D&B&C)&~(A|D);
endmodule
```
#### 设计模块——利用过程语句设计
```
//File: comb_behavior.v
module comb_behavior(output reg Y, input A,B,C,D);
 always@(*) begin
  Y = ~(A | D) & (B & C & ~D);
 end
endmodule;
```
#### 设计模块——利用UDP设计
```
// File: comb_prim.v
primitive comb_prim(output Y, input A,B,C,D);
 table
 //a b c d : y
   0 0 ? ? : 0;
   0 1 0 ? : 0;
   0 1 1 0 : 1;
   0 1 1 1 : 0;
   1 ? ? ? : 0;
 endtable
endprimitive
```
#### 测试模块
```
// File: testbench_comb.v
`timescale 10ns / 1ns
`include "comb_str.v"
`include "comb_dataflow.v"
`include "comb_behavior.v"
`include "comb_prim.v"

module testbench_comb();
 wire Y1,Y2,Y3,Y4;
 reg A,B,C,D;
 integer k;

 comb_str a(Y1, A, B, C, D);
 comb_dataflow b(Y2, A, B, C, D);
 comb_behavior c(Y3, A, B, C, D);
 comb_prim d(Y4, A, B, C, D);

 initial begin
  {A,B,C,D}=4'b0;
  for (k=1;k<16;k=k+1)
   #1 {A, B, C, D} = {A, B, C, D} + 1'b1;
  #1 {A,B,C,D}=4'b0;
 end

 initial begin
  $monitor("At time %4t, A=%b, B=%b, C=%b, D=%b, Y1=%b, Y2=%b, Y3=%b, Y4=%b", 
  $time, A, B, C, D, Y1, Y2, Y3, Y4);
 end

endmodule
```
#### 仿真结果
![exp4-1.png](Simulation_results/exp4-1.png)
![exp4-2.png](Simulation_results/exp4-2.png)

## EXP2-5
#### 设计模块
```
// File: comb_Y1.v
module comb_Y1(output Y, input A, B, C);
 assign Y = (~A & ~B & C) | (~A & B & ~C) |
            (A & ~B & ~C) | (A & ~B & C);
endmodule
```
```
// File: comb_Y2.v

module comb_Y2(Y, A, B, C, D);

    output Y;
    input A, B, C, D;

    assign Y = (~A & B & ~C & ~D) | (~A & B & ~C & D) |
                (~A & B & C & ~D) | (~A & B & C & D) |
                (A & ~B & C & D) | (A & B & ~C & ~D) |
                (A & B & ~C & D);

endmodule
```
#### 测试模块
```
// File: tb_comb_Y1.v
`timescale 10ns / 1ns
`include "comb_Y1.v"

module tb_comb_Y1;

    parameter STEP = 8;
    integer k;

    wire Y;
    reg A, B, C;

    comb_Y1 a(Y, A, B, C);

    initial begin
        {A, B, C} = 3'b0;
        for ( k=1; k<STEP; k=k+1 )
            #1 {A, B, C} = {A, B, C} + 1'b1;
    end

    initial begin
        $monitor("At time %4t, A=%b, B=%b, C=%b, Y=%b",
                    $time, A, B, C, Y);
    end

endmodule
```
```
// File: tb_comb_Y2.v
`timescale 10ns / 1ns
`include "comb_Y2.v"

module tb_comb_Y2;

    parameter STEP = 16;
    integer k;

    wire Y;
    reg A, B, C, D;

    comb_Y2 a(Y, A, B, C, D);

    initial begin
        {A, B, C, D} = 4'b0;
        for ( k=1; k<STEP; k=k+1 )
            #1 {A, B, C, D} = {A, B, C, D} + 1'b1;
    end

    initial begin
        $monitor("At time %4t, A=%b, B=%b, C=%b, D=%b, Y=%b",
                    $time, A, B, C, D, Y);
    end

endmodule
```
#### 仿真结果
![exp5-1.png](Simulation_results/exp5-1.png)
![exp5-2.png](Simulation_results/exp5-2.png)
![exp5-3.png](Simulation_results/exp5-3.png)
![exp5-4.png](Simulation_results/exp5-4.png)

## EXP2-6
#### 设计模块
```
//File: ones_count.v
module ones_count(output reg [3:0] count, input [7:0] dat_in);
 integer k;
 always@(dat_in)begin
  count=4'b0;
  for (k=0;k<8;k=k+1)
   count = count+dat_in[k];
 end
endmodule
```
#### 测试模块
```
// File: tb_ones_count.v
`timescale 10ns / 1ns
`include "ones_count.v"

module tb_ones_count;

    parameter STEP = 256;
    integer k;

    wire [3:0] count;
    reg [7:0] dat_in;

    ones_count a(count, dat_in);

    initial begin
        dat_in = 8'b0;
        for ( k=1; k<STEP; k=k+1 )
            #1 dat_in = dat_in + 1'b1;
    end

    initial begin
        $monitor("At time %4t, dat_in=%b, count=%b",
                    $time, dat_in, count);
    end

endmodule
```
#### 仿真结果
![exp6-1.png](Simulation_results/exp6-1.png)
![exp6-2.png](Simulation_results/exp6-2.png)
![exp6-3.png](Simulation_results/exp6-3.png)
![exp6-4.png](Simulation_results/exp6-4.png)
![exp6-5.png](Simulation_results/exp6-5.png)
![exp6-6.png](Simulation_results/exp6-6.png)
![exp6-7.png](Simulation_results/exp6-7.png)
![exp6-8.png](Simulation_results/exp6-8.png)
![exp6-9.png](Simulation_results/exp6-9.png)

## EXP2-7
#### 设计模块
```
//File: dec_counter.v
module dec_counter(output reg [3:0] count, input clk, reset);
 always@(posedge clk) begin
 if(reset) count<=4'b0;
 else begin
  case (count)
  4'd0: count <= 4'd1;
  4'd1: count <= 4'd2;
  4'd2: count <= 4'd3;
  4'd3: count <= 4'd4;
  4'd4: count <= 4'd5;
  4'd5: count <= 4'd6;
  4'd6: count <= 4'd7;
  4'd7: count <= 4'd8;
  4'd8: count <= 4'd9;
  4'd9: count <= 4'd10;
  4'd10: count <= 4'd0;
  default: count <= 4'b0;
  endcase
 end
 end

endmodule
```
#### 测试模块
```
// File: tb_dec_counter.v
`include "dec_counter.v"

module tb_dec_counter;

    wire [3:0] count;
    reg clk, reset;

    dec_counter a(count, clk, reset);

    initial begin
        clk = 1'b0;
        forever #20 clk = ~clk;
    end

    initial begin
        reset = 1'b0;
        forever #150 reset = ~reset;
    end

    initial begin
        $monitor("At time %4t, reset=%b, count=%d",
                    $time, reset, count);
    end

endmodule
```
#### 仿真结果
![exp7-1.png](Simulation_results/exp7-1.png)
![exp7-2.png](Simulation_results/exp7-2.png)

## EXP2-8
#### 设计模块
#### 测试模块
#### 仿真结果