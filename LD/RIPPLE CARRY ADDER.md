- 4-bit adder based on full adder cell
## RCA

```verilog
module rca(input [3:0]x, input [3:0]y, input c_in, output [3:0]sum, output c_out);

// c0 is c_in
// the wires c1, c2, c3 are the carry connecting the output carry to the next FAC as an input carry

wire c1, c2, c3;

fac fac0(.x(x[0], .y(y[0]), .c_in(c_in), .s(s[0]), .c_out(c1));
		 
fac fac0(.x(x[1], .y(y[1]), .c_in(c1), .s(s[1]), .c_out(c2));
		 
fac fac0(.x(x[2], .y(y[2]), .c_in(c2), .s(s[2]), .c_out(c3));
		 
fac fac0(.x(x[3], .y(y[3]), .c_in(c3), .s(s[3]), .c_out(c_out));
		 
endmodule
```

## RCA TESTBENCH

```verilog
module rca_tb(output reg [3:0]x, output reg [3:0]y, output reg c_in, output [3:0]sum, output c_out);

rca dut(.x(x), .y(y), .c_in(c_in), .sum(sum), .c_out(c_out));

integer i;

initial begin

	{x, y, c_in} = 9'b000000000; 
	// x, y set to 0000 on 4 bits
	// c_in to 0 on 1 bit

	for (i = 0; i < 512; i = i + 1)
		#30 {x, y, c_in} = i;
		
end

endmodule
```