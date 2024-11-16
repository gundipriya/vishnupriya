# Tiny Tapeout
<details>
 <summary>Verilog codes</summary>

 <details>
 <summary>ALU_top</summary>
           <br />
           
            module alu_top (
            input  wire [7:0] ui_in,    
            output wire [7:0] uo_out,   
            input  wire [7:0] uio_in,   
            output wire [7:0] uio_out,  
            output wire [7:0] uio_oe,   
            input  wire       ena,      
            input  wire       clk,     
            input  wire       rst_n     
            );
            
            reg [3:0] in1,in2;
            reg [2:0] sel;
            reg [7:0] alu_out_reg;
            wire [7:0] alu_out;
          
            // All output pins must be assigned. If not used, assign to 0.
          //  assign uo_out  = ui_in + uio_in;  // Example: ou_out is the sum of ui_in and uio_in
            assign uio_out = 0;
            assign uio_oe  = 0;
          //input reg
          always @(posedge clk) begin
            if(!rst_n) begin
             in1 <= 4'b0;
             in2 <= 4'b0;
             sel <= 3'b0;
            end
            else begin
             in1 <= ui_in[3:0];
             in2 <= ui_in[7:4];
             sel <= uio_in[2:0];
            end
          end
          
            alu submodule(.a(in1),.b(in2),.alu_sel(sel),.result(alu_out));
          
           //output reg
          always @(posedge clk)begin
            if(!rst_n)begin
              alu_out_reg <=8'b0;
            end
            else begin
              alu_out_reg <= alu_out;
            end
          end
          
            assign uo_out = alu_out_reg;
            // List all unused inputs to prevent warnings
            wire unused = &{ena,uio_in[7:3],1'b0};
          
          endmodule

   
          
          
          module alu (
              input [3:0] a,            
              input [3:0] b,            
              input [2:0] alu_sel,      
              output reg [7:0] result   
          );
          
          always @(*) begin
          
              case (alu_sel)
                  3'b000: result = a + b;                        // Addition
                  3'b001: result = a - b;                        // Subtraction
                  3'b010: result = {4'b0000, (a & b)};           // Bitwise AND 
                  3'b011: result = {4'b0000, (a | b)};           // Bitwise OR 
                  3'b100: result = {4'b0000, (a ^ b)};           // Bitwise XOR 
                  3'b101: result = {~b, ~a};                     // Bitwise NOT 
                  3'b110: result = a * b;                        // Multiplication
                  3'b111: begin                                  // Division
                      if (b != 0) begin
                          result = {4'b0000, (a / b)};           
                      end else begin
                          result = 8'b00000000;                  
                      end
                  end
                  default: result = 8'b00000000;                 // Default case
              endcase
          end
          
          
          endmodule
 </details>
 <details>
 <summary>ALU_top_tb</summary>
           <br />
  
                   //`timescale 1ns/1ps
        
        module alu_top_tb;
        
        
          reg [7:0] ui_in;
          reg [7:0] uio_in;
          reg clk, rst_n;
          wire [7:0] uo_out;
          wire [7:0] uio_out;
          wire [7:0] uio_oe;
        
        
          alu_top dut (
            .ui_in(ui_in),
            .uo_out(uo_out),
            .uio_in(uio_in),
            .uio_out(uio_out),
            .uio_oe(uio_oe),
            .ena(1'b1),     
            .clk(clk),
            .rst_n(rst_n)
          );
        
          
          always #5 clk = ~clk;
        
        covergroup Alu_coverage @(posedge clk);
           input_a: coverpoint ui_in[3:0];
        
           input_b: coverpoint ui_in[7:4];
        
           alu_op: coverpoint uio_in[2:0] {
                   bins add = {3'b000};
                   bins sub = {3'b001};
                   bins mul ={3'b110};
                   bins div ={3'b111};
                   bins and_op={3'b010};
                   bins or_op={3'b011};
                   bins xor_op={3'b100};
                   bins not_op={3'b101};
            }
        
          result: coverpoint uo_out;
        
          cr1:cross input_a,input_b;
        endgroup
        
        Alu_coverage alu_cov;
        
          initial begin
          alu_cov = new;
            clk = 0;
            rst_n = 0;
            ui_in = 8'd0;
            uio_in = 8'd0;

    #10 rst_n = 1;

    repeat (1000) begin
      @(posedge clk);
      // Randomize inputs
      ui_in = $random;
      uio_in[2:0] = $random;  
      uio_in[7:3] = 5'b0;     
      #10;
    end

    
    $finish;
  end
  initial
    begin
      $dumpfile("dump.vcd");
      $dumpvars();
    end

endmodule
