## Transmisia de date prin intermediul VGA (video graphics array)


Folosim clock-ul de 25 MHz pentru a minimiza erorile de transmisie



|     |     |     |     |     |     |     |     |
| --- | --- | --- | --- | --- | --- | --- | --- |
|     |     |     |     |     |     |     |     |


**Pixel Clock = Total Pixels per Line  x  Total Lines per Frame  x  Refresh Rate**

```verilog
// Horizontal timing     hEND = 400 -> hDisplay = 320 , rest 80
     localparam hEND        = hDisp + hFp + hPulse + hBp; 
     localparam hSyncStart  = hDisp + hFp;
     localparam hSyncEnd    = hDisp + hFp + hPulse;
             
// Vertical timing      vEND = 285 -> vDisplay = 240, rest 45
     localparam vEND        = vDisp + vFp + vPulse + vBp;
     localparam vSyncStart  = vDisp + vFp;
     localparam vSyncEnd    = vDisp + vFp + vPulse;
     
     reg [9:0] hc; // Horizontal Counter
     reg [9:0] vc; // Vertical Counter
     
     always@(posedge i_clk or negedge i_rstn)
        begin
            if(!i_rstn) begin
                hc      <= 0;
                vc      <= 0;
            end
            else begin
                if(hc == hEND-1)
                begin
                    hc <= 0;
                    if(vc == vEND-1)
                    vc <= 0; 
                    else
                        vc <= vc + 1'b1;
                end 
                else
                    hc <= hc + 1'b1; 
            end
        end 
        
     // Output (x,y) coordinates of the pixel and timing signals
     assign o_x_counter = hc;
     assign o_y_counter = vc;
     assign o_video     = ((hc >= 0) && (hc < hDisp) && (vc >= 0)  && (vc < vDisp)); 
     assign o_hsync     = ~((hc >= hSyncStart) && (hc < hSyncEnd));
     assign o_vsync     = ~((vc >= vSyncStart) && (vc < vSyncEnd)); 
```
