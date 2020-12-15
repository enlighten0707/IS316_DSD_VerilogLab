# 数字系统设计作业报告
* 学号：518021910628
* 姓名：吴小茜

<!-- 设计模块，测试模块，波形图，显示输出，设计说明 -->

### EXP2-1
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

### EXP2-2
#### 设计模块
```
// File: encoder8x3.v 
module encoder8x3( output reg[2:0] code, input [7:0] data ); 
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

module tb_encoder8x3;

    parameter STEP = 8;

    wire [2:0] code;
    reg [7:0] data;
    integer k;

    encoder8x3 a(code, data);

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
