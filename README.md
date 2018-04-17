# New_Datapath
module DatapathLEGv8 (/*ControlWord, status_out,*/ data, clock, reset);
	wire [96:0] ControlWord;
	inout [63:0] data;
	input clock, reset;
	wire [3:0] status;
	/*output*/ wire[4:0] status_out;

	wire [63:0] K;
	wire [4:0] SA, SB, DA;
	wire RegWrite, MemWrite;
	wire [63:0] RegAbus, RegBbus, A, B, B_out;
	wire [4:0] FS;
	wire C0;
	wire [63:0] ALU_output, MEM_output;
	wire EN_Mem, EN_ALU, EN_PC, EN_B, SL;
	wire Asel, Bsel, NS;
	wire [1:0] PS;
	wire [63:0]PC_in, PC_out;
	wire [63:0] in;

	assign {SA, SB, DA, RegWrite, MemWrite, FS, C0, EN_Mem, EN_ALU, Asel,Bsel,PS, EN_PC, EN_B, SL, NS, K} = ControlWord;
	
	assign RegBbus = Bsel ? K : B;
	assign RegAbus = Asel ? K : A;
	
// The constant input to the Program counter
//	Comes from constant on A bus. 
	assign in = RegAbus;
	
	assign B_out = EN_B ? B : 64'bz;

	RegFile32x64 regfile(A, B, data, DA, SA, SB, RegWrite, reset, clock);

	ALU_LEGv8_Improved alu (A, RegBbus, FS, C0, ALU_output, status);

	RAM256x64 data_mem (ALU_output, clock, B_out, MemWrite, MEM_output);
	
	ProgCounter PC (PC_out, PC_in, PS, in, clock, reset);
	
	statusRegister statReg (status_out, status, SL, reset, clock);
	
	ControlUnit CU (I, status_out, reset, clock, ControlWord);

	rom_case inst_mem (I, PC_out[17:2]);
	
	wire [31:0] I;
	
	assign data = EN_Mem ? MEM_output : 64'bz;
	assign data = EN_ALU ? ALU_output : 64'bz;
	assign data = EN_PC ? PC_in : 64'bz;

endmodule





