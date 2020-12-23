# 数字系统设计作业报告
* 学号：518021910628
* 姓名：吴小茜

<!-- 设计模块，测试模块，波形图，显示输出，设计说明 -->

## EXP2-1
#### 设计/测试模块
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
//File comb_dataflow.v
module comb_dataflow(output Y, input A,B,C,D);
 assign Y=(~D&B&C)&~(A|D);
endmodule
```
#### 设计模块——利用过程语句设计
```verilog
//File: comb_behavior.v
module comb_behavior(output reg Y, input A,B,C,D);
 always@(*) begin
  Y = ~(A | D) & (B & C & ~D);
 end
endmodule;
```
#### 设计模块——利用UDP设计
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
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
```verilog
// File: comb_str_2.v 
`include "mux2x1.v" 
module comb_str( y, sel, a, b, c, d); 
 output y; 
 input sel; 
 input a, b, c, d; 
 wire s0, s1; 
 nand #3 u1( s0, a, b); 
 nand #4 u2( s1, c, d); 
 
 mux2x1 m0( y, sel, {s1,s0}); 
endmodule 

```
#### 测试模块
```verilog
// File: tb_comb_str_2.v

`include "comb_str_2.v" 
`define STOP 32 

module tb_comb_str(); 
 reg [4:0] px; 
 wire py; 
 integer i; 
 
 initial begin 
 px = 5'b0; 
 for ( i = 1; i < `STOP; i = i + 1 ) begin 
 #5 px = px + 1'b1; 
 end 
 end 
 comb_str m1(.y(py),.sel(px[4]),.a(px[3]),.b(px[2]),.c(px[1]),.d(px[0])); 
 
 initial 
 $monitor( "At time %t, y=%1b, sel=%1b, a=%1b, b=%1b, c=%1b, d=%1b",  $time, py, px[4], px[3], px[2], px[1], px[0] ); 
endmodule 
```
#### 仿真结果
![exp8-1.png](Simulation_results/exp8-1.png)
![exp8-2.png](Simulation_results/exp8-2.png)
![exp8-3.png](Simulation_results/exp8-3.png)

## EXP2-9
#### 设计模块
```verilog
//File: LFSR.v
module LFSR(q, clk, rst_n, load, din);

    output reg [1:26] q;    // 26 bit data output.
    input clk;              // Clock input.
    input rst_n;            // Synchronous reset input.
    input load;             // Synchronous load input.
    input [1:26] din;       // 26 bit parallel data input.

    always @ ( posedge clk ) begin
        if (!rst_n) q <= 26'b0;
        else begin
            if (load) q <= |din ? din : 26'b1;
            else begin
                if (q == 26'b0) q <= 26'b1;
                else begin
                    q[10:26] <= q[9:25];
                    q[9] <= q[8] ^ q[26];
                    q[8] <= q[7] ^ q[26];
                    q[3:7] <= q[2:6];
                    q[2] <= q[1] ^ q[26];
                    q[1] <= q[26];
                end
            end
        end
    end

endmodule
```
#### 测试模块
```verilog
//File: tb_LFSR.v

`include "LFSR.v" 

module tb_LFSR(); 
 wire [1:26] p_q; 
 reg [1:26] p_din; 
 reg p_clk, p_rst_n, p_load; 
 
 LFSR u0(.q(p_q), .clk(p_clk), .rst_n(p_rst_n), .load(p_load), .din(p_din)); 
 
 initial begin 
  p_clk = 0; 
  forever #5 p_clk = ~p_clk; 
 end 

 initial begin 
  p_rst_n = 0; 
  #8 p_rst_n = 1; 
 end 
 
 initial begin 
  p_load = 0; 
  p_din = 26'd0; 
 
 #88 p_load = 1'b1; 
 p_din = 26'd4668_5317; // 26'b10_1100_1000_0101_1100_1000_0101 
 #10 p_load = 1'b0; 
 
 #80 p_load = 1'b1; 
 p_din = 26'd0; 
 #10 p_load = 1'b0; 
 end 
 
 initial begin 
 $monitor("At time t=%4t, rst_n=%b, load=%b, din=%d, q=%d", 
 $time, p_rst_n, p_load, p_din, p_q); 
 end 
endmodule 
```
#### 仿真结果
![exp9-1.png](Simulation_results/exp9-1.png)
![exp9-2.png](Simulation_results/exp9-2.png)

## EXP2-10
#### 设计模块
```verilog
//File: filter.v
module filter(sig_out, clock, reset, sig_in); 
 output sig_out; 
 input clock, reset, sig_in; 
 
 reg [3:0] data; 
 reg dout; 
 
 always @(posedge clock) begin 
 if (!reset) begin 
 data <= 4'b0; 
 dout <= 1'b0; 
 end 
 else begin 
 data <= {data[2:0], sig_in}; 
 
 case ( { &data[3:1], &(~data[3:1])} ) 
 2'b01: dout <= 1'b0; 
 2'b10: dout <= 1'b1; 
 2'b11: dout <= ~dout; 
 default: ; 
 endcase 
 end 
 end 
 assign sig_out = dout; 
endmodule 
```
#### 测试模块
```verilog
//File: tb_filter.v

`include "filter.v" 
module tb_filter; 
 wire p_o; 
 reg p_clk, p_rst, p_i; 
 
 filter u0(.sig_out(p_o), .clock(p_clk), .reset(p_rst), .sig_in(p_i)); 
 
 initial begin 
 p_clk = 0; 
 forever #5 p_clk = ~p_clk; 
 end 
 
 initial begin 
 p_rst = 1'b0; 
 #23 p_rst = 1'b1; 
 end 
 
 integer k; 
 reg [31:0] din; 
 initial begin 
 din = 32'b1101_0010_0000_0111_1101_1111_0000_0001; 
 #10; 
 for (k=0; k<32; k=k+1) 
 #10 p_i = din[k]; 
 end 
 
 initial 
 $monitor( "At time %t, reset=%b, sig_in=%b, sig_out=%b", 
 $time, p_rst, p_i, p_o); 
endmodule 
```
#### 仿真结果
![exp10-1.png](Simulation_results/exp10-1.png)
![exp10-2.png](Simulation_results/exp10-2.png)

## EXP2-11
#### 设计模块
```verilog
// File: counter8b_updown.v 
module counter8b_updown(output [7:0] count, input clk, reset, dir); 
 reg [7:0] count0,  count1;  
 always @( posedge clk or negedge reset ) 
 begin 
  if ( reset == 1'b0 ) begin 
   count0 <= 8'b0;  
   count1 <= 8'b1111_1111;  
  end 
 else if ( dir ) 
  count0 <= count0 + 1;  
 else 
  count1 <= count1 - 1;  
 end 
assign count = (dir) ?  count0 : count1; 
endmodule 
```
#### 测试模块
```verilog
// File: tb_counter8b_updown.v 

`include "counter8b_updown.v" 

module tb_counter8b_updown();  
 wire [7:0] p_cnt;  
 reg p_clk,  p_rst,  p_dir;  

 initial begin 
  p_clk = 0;  
  forever #5 p_clk = ~p_clk;  
 end 

 initial begin 
  #5 p_rst = 1'b0;  
  #2 p_dir = 1'b1;  
  #10 p_rst = 1'b1;  
  #80 p_rst = 1'b0;  
  #2 p_dir = 1'b0;  
  #10 p_rst = 1'b1;  
  #100 p_dir = 1'b1;  
 end 

 counter8b_updown m0(.count(p_cnt), .clk(p_clk), .reset(p_rst), .dir(p_dir) );
 initial 
 $monitor("At time %t, <----> count=%b, dir=%b" ,  $time,  p_cnt,  p_dir ); 

endmodule 
```
#### 仿真结果
![exp11-1.png](Simulation_results/exp11-1.png)
![exp11-2.png](Simulation_results/exp11-2.png)


## EXP2-12
#### 设计模块
<!-- 这里用ANSI-C风格传递参数似乎会出问题 -->
<!-- ~操作，~8`b1111_1111 -> 9'b1_0000_0000-->
<!-- 组合逻辑，case语句阻塞赋值 -->
```verilog
module ALU(c_out, sum, oper, a, b, c_in);

    output reg [7:0] sum;
    output reg c_out;
    input [2:0] oper;
    input [7:0] a;
    input [7:0] b;
    input c_in;

    always @ ( * ) begin
        case (oper)
            3'b000: {c_out, sum} <= a + b + c_in;   // and
            3'b001: {c_out, sum} <= a + ~b + c_in;  // subtract
            3'b010: {c_out, sum} <= ~a + b + ~c_in; // subtract_a
            3'b011: {c_out, sum} <= {1'b0, a | b};  // or_ab
            3'b100: {c_out, sum} <= {1'b0, a & b};  // and_ab
            3'b101: {c_out, sum} <= {1'b0, ~a & b}; // not_ab
            3'b110: {c_out, sum} <= {1'b0, a ^ b};  // exor
            3'b111: {c_out, sum} <= {1'b0, a ~^ b}; // exnor
            default: {c_out, sum} <= 9'bx;
        endcase
    end
endmodule
```
#### 测试模块
```verilog
// File: tb_ALU.v

`include "ALU.v"

module tb_ALU;

    parameter STEP = 8;
    integer k;

    wire c_out, sum;
    reg [2:0] oper;
    reg [7:0] a;
    reg [7:0] b;
    reg c_in;

    ALU c(c_out, sum, oper, a, b, c_in);

    initial begin
        oper = 3'b0;
        for ( k=1; k<STEP; k=k+1 )
            #5 oper = oper + 3'b1;
    end

    initial begin
        a = 8'b0;
        forever #5 a = {a[7:0], $random % 2};
    end

    initial begin
        b = 8'b0;
        forever #10 b = {b[7:0], $random % 2};
    end

    initial begin
        c_in = 1'b0;
        forever #15 c_in = $random % 2;
    end

    initial begin
        $monitor("At time %4t, a=%b, b=%b, c_in=%b, oper=%b, sum=%b, c_out=%b",
                    $time, a, b, c_in, oper, sum, c_out);
    end

endmodule
```
#### 仿真结果
![exp12-1.png](Simulation_results/exp12-1.png)
![exp12-2.png](Simulation_results/exp12-2.png)


## EXP2-13
#### 设计模块
```verilog
// File: shift_counter.v
module shift_counter(count, clk, reset);
 output [7:0] count;
 input clk;
 input reset;
 parameter CNTSUP = 17;
 reg [4:0] cnt;
 always @( posedge clk ) begin
  if ( reset )
   cnt <= 5'b0;
  else 
  begin
  if (cnt < CNTSUP)
   cnt <= cnt + 1'b1;
  else
   cnt <= 5'b0;
  end
 end

 function [7:0] val( input [4:0] c );
 begin
 case ( c)
  0: val = 8'b0000_0001;
  1: val = 8'b0000_0001;
  2: val = 8'b0000_0001;
  3: val = 8'b0000_0001;
  4: val = 8'b0000_0010;
  5: val = 8'b0000_0100;
  6: val = 8'b0000_1000;
  7: val = 8'b0001_0000;
  8: val = 8'b0010_0000;
  9: val = 8'b0100_0000;
  10: val = 8'b1000_0000;
  11: val = 8'b0100_0000;
  12: val = 8'b0010_0000;
  13: val = 8'b0001_0000;
  14: val = 8'b0000_1000;
  15: val = 8'b0000_0100;
  16: val = 8'b0000_0010;
  17: val = 8'b0000_0001;
  default:val = 8'b0000_0001;
 endcase
 end
 endfunction
 assign count = val(cnt);
endmodule
```
#### 测试模块
```verilog
`include "shift_counter.v"
`define PULSE 5
module tb_shift_counter;
 wire [7:0] p_cnt;
 reg p_clk;
 reg p_rst;
 initial begin
  p_rst = 1'b1;
  #25 p_rst = 1'b0;
 end

 initial begin
 p_clk = 1'b0;
 forever #`PULSE p_clk = ~p_clk;
 end

 shift_counter u0(.count(p_cnt), .clk(p_clk), .reset(p_rst));
 initial
 $monitor( "At time %4t, reset=%b, count=%b", $time, p_rst, p_cnt );
endmodule
```
#### 仿真结果
![exp13-1.png](Simulation_results/exp13-1.png)
![exp13-2.png](Simulation_results/exp13-2.png)

## EXP2-14
#### 设计模块
```verilog
// File: sram.v

module sram(dout, din, addr, wr, rd, cs);

    output [7:0] dout;
    input [7:0] din;
    input [7:0] addr;
    input wr, rd, cs;

    reg [7:0] sram [255:0];
    reg [7:0] data;

    assign dout = (cs && !rd) ? data : 8'bz;

    always @ ( posedge wr ) begin
        if (cs) sram[addr] <= din;
    end

    always @ ( negedge rd ) begin
        if (cs) data <= sram[addr];
    end

endmodule
```
#### 测试模块
```verilog
// File: tb_sram.v

`include "sram.v"

module tb_sram;

    wire [7:0] dout;
    reg [7:0] din;
    reg [7:0] addr;
    reg wr, rd, cs;

    sram a(dout, din, addr, wr, rd, cs);

    initial begin
        cs = 1'b0;
        #10 cs = 1'b1;
        #10 cs = 1'b0;
        #10 cs = 1'b1;
    end

    initial begin
        din = 8'b1010_0101;
        addr = 8'b0101_1010;
        #100 addr = 8'b1010_0101;
    end

    initial begin
        wr = 1'b0;
        rd = 1'b1;
        forever begin
            #10;
            wr = $random % 2;
            rd = $random % 2;
        end
    end

    initial begin
        $monitor("At time %4t, din=%b, addr=%b, cs=%b, wr=%b, rd=%b, dout=%b",
                    $time, din, addr, cs, wr, rd, dout);
    end

endmodule
```
#### 仿真结果
![exp14-1.png](Simulation_results/exp14-1.png)
![exp14-2.png](Simulation_results/exp14-2.png)

## EXP2-15
#### 设计模块
```verilog
// File: seq_detect.v

module seq_detect(flag, din, clk, rst_n);

    output reg flag;
    input din, clk, rst_n;

    parameter S10 = 9'b0_0000_0001; // IDLE(0)
    parameter S11 = 9'b0_0000_0010; // A(0)
    parameter S12 = 9'b0_0000_0100; // B(0)
    parameter S13 = 9'b0_0000_1000; // C(0)
    parameter S14 = 9'b0_0001_0000; // D(1)

    parameter S20 = 9'b0_0000_0001; // IDLE(0)
    parameter S21 = 9'b0_0010_0000; // E(0)
    parameter S22 = 9'b0_0100_0000; // F(0)
    parameter S23 = 9'b0_1000_0000; // G(0)
    parameter S24 = 9'b1_0000_0000; // H(1)

    reg [8:0] state1;   // 1101
    reg [8:0] state2;   // 0110
    reg flag1, flag2;

    always @ ( * ) begin
        flag <= flag1 | flag2;
    end

    always @ ( negedge clk ) begin
        if (!rst_n) begin
            flag1 <= 1'b0;
            state1 <= S10;
        end else begin
            flag1 <= (state1 == S14) ? 1'b1 : 1'b0;
            case (state1)
                S10: state1 <= (din) ? S11 : S10;
                S11: state1 <= (din) ? S12 : S10;
                S12: state1 <= (din) ? S12 : S13;
                S13: state1 <= (din) ? S14 : S10;
                S14: state1 <= (din) ? S12 : S10;
                default: begin state1 <= S10; flag1 <= 1'b0; end
            endcase
        end
    end

    always @ ( negedge clk ) begin
        if (!rst_n) begin
            flag2 <= 1'b0;
            state2 <= S20;
        end else begin
            flag2 <= (state2 == S24) ? 1'b1 : 1'b0;
            case (state2)
                S20: state2 <= (din) ? S20 : S21;
                S21: state2 <= (din) ? S22 : S21;
                S22: state2 <= (din) ? S23 : S21;
                S23: state2 <= (din) ? S20 : S24;
                S24: state2 <= (din) ? S22 : S21;
                default: begin state2 <= S20; flag2 <= 1'b0; end
            endcase
        end
    end

endmodule
```
#### 测试模块
```verilog
// File: tb_seq_detect.v

`include "seq_detect.v"

module tb_seq_detect;

    parameter STEP = 32;
    integer k;

    wire flag;
    reg [31:0] data;
    reg din,clk, rst_n;

    seq_detect a(flag, din, clk, rst_n);

    initial begin
        clk = 1'b0;
        forever #10 clk = ~clk;
    end

    initial begin
        rst_n = 1'b0;
        #50 rst_n = 1'b1;
    end

    initial begin
        data = 32'b1100_0110_0100_0110_1010_0100_1010_0010;
        for ( k=1; k<STEP; k=k+1 ) begin
            #20;
            din = data[31];
            data = data << 1;
        end
    end

endmodule
```
#### 仿真结果
![exp15-1.png](Simulation_results/exp15-1.png)
#### 设计说明--状态转移图
![exp15-2.png](Simulation_results/exp15-2.png)

## EXP2-16
#### 设计模块
```verilog
```
#### 测试模块
```verilog
```
#### 仿真结果
