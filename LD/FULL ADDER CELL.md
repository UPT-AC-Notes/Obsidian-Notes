## FAC

```verilog
module fac(input x, input y, input c_in, output sum, output c_out);

assign sum = x ^ y ^ c_in;
assign c_out = (x & y) | (x & c_in) | (y & c_in);

endmodule
```

## FAC TESTBENCH

```verilog
module fac_tb(output reg x_tb, output reg y_tb, output reg c_in_tb, output sum_tb, output c_out_tb);

fac dut(.x(x_tb), .y(y_tb), .c_in(c_in_tb), .sum(sum_tb), .c_out(c_out_tb));

integer i;

initial begin

	{x_tb, y_tb, c_in_tb} = 3'b000; // set x, y, c_in to 0

	for(i = 0; i < 8; i = i + 1)
		#30 {x_tb, y_tb, c_in_tb} = i; 
		// delay 30 nanoseconds
		// set x, y, c_in to bits of i (000 to 111)

end

endmodule
```
