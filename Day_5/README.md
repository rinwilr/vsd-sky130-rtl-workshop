# Day 5 - Optimization in synthesis

if - statement
Used to create priority logic
Syntax:
if <cond>
begin
end
else
begin
end

if <cond>
else if <cond>
else if <cond>
else

Caution with if:
Inferred Latches(Bad coding style) - due to incomplete "if"

Counter:
always @(posedge clk,posedge reset)
begin
if(reset)
count<=3'b000;
else if(en)
count<=count+1;
end
COMBINATIONAL CIRCUITS SHOULD NOT HAVE INFERRED LATCHES

Case Statement
must be a reg variable

reg y
always(*)
begin
case(sel)
2'b00:
begin
end
2'b01:
begin
end
endcase
end

Caveats with case
1. Incomplete case leads to inferred latches
   Solution: codecase with default
 2.   
3. 
