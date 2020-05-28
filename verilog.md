### 握手信号的写法

data_in

data_in_valid

data_store_ready 进行握手信号。

```
reg tx_en;
reg tx_done;

always@(posedge clk or negedge rst_n)begin
	if(!rst_n)begin
		reg_data <= 16'd0;
		data_store_ready <= 1'b1;
	end
	else if(data_in_valid)begin
		reg_data <= data_in;
		data_store_ready <= 1'b0;
		tx_en <= 1'b1;
	end
	else if(tx_done)begin
		data_store_ready <= 1'b1;
		tx_en <= 1'b0;
	end
end
```



### ROM读取

```
always@(posedge clk_1K or negedge rst_n)begin
	if(!rst_n)begin
		addra <= 10'd0;
	end
	else begin
		addra <= addra + 1;
	end
end

cos1_rom cos_high (
  .clka(cnt_1k),    // input wire clka
  .addra(addra),  // input wire [9 : 0] addra
  .douta(cos_high_data)  // output wire [15 : 0] douta
);

cos2_rom cos_low (
  .clka(cnt_1k),    // input wire clka
  .addra(addra),  // input wire [9 : 0] addra
  .douta(cos_low_data)  // output wire [15 : 0] douta
);

```

### I2s的发送

```
module i2s_tx #(
	parameter AUDIO_DW	= 32
)(
	input			sclk,
	input			rst,

	// Prescaler for lrclk generation from sclk should hold the number of
	// sclk cycles per channel (left and right).
	input [AUDIO_DW-1:0]	prescaler,

	output reg		lrclk,
	output reg		sdata,

	// Parallel datastreams
	input [AUDIO_DW-1:0]	left_chan,
	input [AUDIO_DW-1:0]	right_chan
);

reg [AUDIO_DW-1:0]		bit_cnt;
reg [AUDIO_DW-1:0]		left;
reg [AUDIO_DW-1:0]		right;

always @(negedge sclk)
	if (rst)
		bit_cnt <= 1;
	else if (bit_cnt >= prescaler)
		bit_cnt <= 1;
	else
		bit_cnt <= bit_cnt + 1;

// Sample channels on the transfer of the last bit of the right channel
always @(negedge sclk)
	if (bit_cnt == prescaler && lrclk) begin
		left <= left_chan;
		right <= right_chan;
	end
// left/right "clock" generation - 0 = left, 1 = right
always @(negedge sclk)
	if (rst)
		lrclk <= 1;
	else if (bit_cnt == prescaler)
		lrclk <= ~lrclk;

always @(negedge sclk)
	sdata <= lrclk ? right[AUDIO_DW - bit_cnt] : left[AUDIO_DW - bit_cnt];

endmodule
```

