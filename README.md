# Tiny Tapeout
<details>
 <summary>Verilog codes</summary>
**ALU_top**<br />

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
