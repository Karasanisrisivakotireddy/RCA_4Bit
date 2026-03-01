module rca_ila(
    input  wire       clk,
    input  wire [1:0] addr
);

    reg  [3:0] xm [0:3];
    reg  [3:0] ym [0:3];

    wire [3:0] xw, yw;
    wire [3:0] sumw;
    wire       cinw;
    wire       coutw;
    wire [4:0] resultw;   // NEW

    assign xw   = xm[addr];
    assign yw   = ym[addr];
    assign cinw = addr[0];

    RCA4 dut (
        .cy  (cinw),
        .x   (xw),
        .y   (yw),
        .s   (sumw),
        .cy4 (coutw)
    );

    // Combine sum and carry
    assign resultw = {coutw, sumw};

    initial begin
        xm[0] = 4'd7;  ym[0] = 4'd6;
        xm[1] = 4'd5;  ym[1] = 4'd9;
        xm[2] = 4'd5;  ym[2] = 4'd6;
        xm[3] = 4'd7;  ym[3] = 4'd8;
    end

    ila_0 ila_instance (
        .clk   (clk),
        .probe0(xw),     
        .probe1(yw),     
        .probe2(cinw),   
        .probe3(resultw)   // 5-bit
    );

endmodule




`timescale 1ns / 1ps

module sum(output s, input a, b, c);
    wire t;
    xor x1(t, a, b);
    xor x2(s, t, c);
endmodule

module carry(output cy_out, input a, b, cin);
    wire t1, t2, t3;

    and g1(t1, a, b);
    and g2(t2, b, cin);
    and g3(t3, cin, a);
    or  g4(cy_out, t1, t2, t3);
endmodule

module FA(output s, cout, input a, b, c);
    sum   u1(s, a, b, c);
    carry u2(cout, a, b, c);
endmodule

module RCA4(
    input        cy,
    input  [3:0] x,
    input  [3:0] y,
    output [3:0] s,
    output       cy4
);
    wire [2:0] c;

    FA fa0(s[0], c[0], x[0], y[0], cy);
    FA fa1(s[1], c[1], x[1], y[1], c[0]);
    FA fa2(s[2], c[2], x[2], y[2], c[1]);
    FA fa3(s[3], cy4 , x[3], y[3], c[2]);
endmodule

module rca_ila(
    input  wire       clk,
    input  wire [1:0] addr
);

    reg  [3:0] xm [0:3];
    reg  [3:0] ym [0:3];

    wire [3:0] xw, yw;
    wire [3:0] sumw;
    wire       cinw;
    wire       coutw;

    assign xw   = xm[addr];
    assign yw   = ym[addr];
    assign cinw = addr[0];

   
    RCA4 dut (
        .cy  (cinw),
        .x   (xw),
        .y   (yw),
        .s   (sumw),
        .cy4 (coutw)
    );

   
    initial begin

        xm[0] = 4'd7;  ym[0] = 4'd6;
        xm[1] = 4'd5;  ym[1] = 4'd9;
        xm[2] = 4'd5;  ym[2] = 4'd6;
        xm[3] = 4'd7;  ym[3] = 4'd8;
    end



    ila_0 ila_instance (
        .clk   (clk),
        .probe0(xw),     
        .probe1(yw),     
        .probe2(cinw),   
        .probe3(sumw),   
        .probe4(coutw)
    );

endmodule

module rca_ila_TB;
reg clk;
    reg [1:0] addr;
  

    rca_ila koti (
        .addr(addr),.clk(clk)
    );

 
    initial begin
        addr = 2'b00; #20;
        addr = 2'b01; #20;
        addr = 2'b10; #20;
        addr = 2'b11; #20;

        $finish;
    end

endmodule
