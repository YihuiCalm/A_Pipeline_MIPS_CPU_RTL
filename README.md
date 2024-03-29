# **Pipeline-MIPS-CPU**
* **Description**: Implementation of A Pipeline MIPS CPU  
* **HDL Language**: Verilog  
* **Target Device**:   
* **Tool Version**:  Synopsys VCS O-2018.09, Verdi O-2018.09-SP2
***  
## **MIPS32 Instructions**  
This design only realized basic MIPS32 instructions below:  
<table>
    <tr>
        <td align="center">Type</td>
        <td align="center">Instruction</td>
        <td align="center">[31:26]</td>
        <td align="center">[25:21]</td>
        <td align="center">[20:16]</td>
        <td align="center">[15:11]</td>
        <td align="center">[10:6]</td>
        <td align="center">[5:0]</td>
        <td align="center">Function</td>
    </tr>
    <tr>
        <td align="center">R</td>
        <td align="center">add</td>
        <td align="center">000000</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center">rd</td>
        <td align="center">0</td>
        <td align="center">100000</td>
        <td align="center">rd = rs + rt</td>
    </tr>
    <tr>
        <td align="center">R</td>
        <td align="center">sub</td>
        <td align="center">000000</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center">rd</td>
        <td align="center">0</td>
        <td align="center">100010</td>
        <td align="center">rd = rs - rt</td>
    </tr>
    <tr>
        <td align="center">R</td>
        <td align="center">and</td>
        <td align="center">000000</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center">rd</td>
        <td align="center">0</td>
        <td align="center">100100</td>
        <td align="center">rd = rs &amp; rt</td>
    </tr>
    <tr>
        <td align="center">R</td>
        <td align="center">or</td>
        <td align="center">000000</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center">rd</td>
        <td align="center">0</td>
        <td align="center">100101</td>
        <td align="center">rd = rs | rt</td>
    </tr>
    <tr>
        <td align="center">R</td>
        <td align="center">slt</td>
        <td align="center">000000</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center">rd</td>
        <td align="center">0</td>
        <td align="center">101010</td>
        <td align="center">rd = rs &lt; rt? 1: 0</td>
    </tr>
    <tr>
        <td align="center">I</td>
        <td align="center">lw</td>
        <td align="center">100011</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center"  colspan="3">immediate</td>
        <td align="center">rt = mem(rs + imm)</td>
    </tr>
    <tr>
        <td align="center">I</td>
        <td align="center">sw</td>
        <td align="center">101011</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center"  colspan="3">immediate</td>
        <td align="center">mem(rs+imm)=rt</td>
    </tr>
    <tr>
        <td align="center">I</td>
        <td align="center">beq</td>
        <td align="center">000100</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center"  colspan="3">immediate</td>
        <td align="center">PC=rs==rt? PC+4+imm*4: PC+4</td>
    </tr>
    <tr>
        <td align="center">J</td>
        <td align="center">J</td>
        <td align="center">000010</td>
        <td align="center" colspan="5">address</td>
        <td align="center">PC=address</td>
    </tr>
</table>  
For there types of instructions, they have formats of 32-bits below:  
<table>
    <tr>
        <td align="center">Type</td>
        <td align="center">[31:26]</td>
        <td align="center">[25:21]</td>
        <td align="center">[20:16]</td>
        <td align="center">[15:11]</td>
        <td align="center">[10:6]</td>
        <td align="center">[5:0]</td>
    </tr>
    <tr>
        <td align="center">R</td>
        <td align="center">op</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center">rd</td>
        <td align="center">sa</td>
        <td align="center">func</td>
    </tr>
    <tr>
        <td align="center">I</td>
        <td align="center">op</td>
        <td align="center">rs</td>
        <td align="center">rt</td>
        <td align="center"  colspan="3">immediate</td>
    </tr>
    <tr>
        <td align="center">J</td>
        <td align="center">op</td>
        <td align="center"  colspan="5">address</td>
    </tr>
</table>

***  
## **Pipeline CPU Architecture**  
The Architecture Design references the [CSE378](https://courses.cs.washington.edu/courses/cse378/) of UW:
![image](https://github.com/YihuiCalm/Pipeline_MIPS_CPU/assets/96307958/344fa823-eade-4bb6-9528-d14af178b149)
This image only shows the basic pipeline structure without the hazard units.
## **Hazard**
* **Structural Hazard**: To avoid the conflict of reading and writing data on the register at the same cycle, the register will be written at negative edge of `clk`, and the reading process will occur at the positive edge. 
* **Data Hazard**: To avoid the data hazard caused by the `lw` instruction, "bubbles" will be inserted into the pipeline, which means the ID/EX register will be reset for a cycle when this kind of hazard happens. Data from the latest stage will be forwarded for other kinds of hazards.
* **Control Hazard**: The pipeline will flush when a `beq` or a `j` happens to avoid the control hazard.


***
## **Verilog Module Design**
Only the top module is shown here, the specific module design could be found in the [source](https://github.com/YihuiCalm/Pipeline_MIPS_CPU/tree/main/source) folder:
```verilog
`timescale 1ns / 1ps

module CPU_top(
    input clk,
    input rst_n,
    input [31:0] instruction,

    output [31:0] inst_address
);

    // Operation parameters
    parameter ADD = 4'b0001;
    parameter SUB = 4'b0010;
    parameter AND = 4'b0011;
    parameter OR = 4'b0100;
    parameter SLT = 4'b0101;
    parameter LW = 4'b0110;
    parameter SW = 4'b0111;
    parameter BEQ = 4'b1000;
    parameter J = 4'b1001; 

    // Internal signals
    wire [31:0] instruction_IF, instruction_ID; 
    wire [31:0] inst_addr_IF, inst_addr_ID, inst_addr_EX;
    wire [31:0] jump_inst_addr_EX, jump_inst_addr_MEM;
    wire [31:0] shifted_address_EX;
    wire [31:0] shifted_address_MEM;
    wire shift_enable, jump_enable;

    wire [4:0] read_register_1_ID, read_register_1_EX;
    wire [4:0] read_register_2_ID, read_register_2_EX;
    wire reg_write_enable_WB, reg_write_enable_MEM;
    wire [31:0] reg_write_data, reg_read_data_1_ID, reg_read_data_1_EX, reg_read_data_2_ID, reg_read_data_2_EX;
    wire [4:0] reg_write_address_1_EX, reg_write_address_2_EX, reg_write_address_EX, reg_write_address_MEM, reg_write_address_WB;

    wire [31:0] extended_immi_ID, extended_immi_EX;
    
    wire [3:0] op_type_ID, op_type_EX, op_type_MEM, op_type_WB;
    wire [31:0] alu_result_EX, alu_result_MEM, alu_result_WB;

    wire [31:0] write_mem_data_MEM;
    wire mem_write_enable;
    wire mem_read_enable;
    wire [31:0] mem_read_data_MEM, mem_read_data_WB;

    wire stall, flush;

    // Instantiation
    IF2ID_reg inst_IF2ID_reg(
        .clk(clk),
        .rst_n(rst_n),
        .stall(stall),
        .instruction_next(instruction_IF),
        .inst_address_next(inst_addr_IF),
        .instruction(instruction_ID),
        .inst_address(inst_addr_ID)
    );
    
    register_MEM inst_register(
        .clk(clk),
        .rst_n(rst_n),
        .read_register_1(read_register_1_ID),
        .read_register_2(read_register_2_ID),
        .write_register(reg_write_address_WB),
        .write_data(reg_write_data),
        .write_enable(reg_write_enable_WB),
        .register_1(reg_read_data_1_ID),
        .register_2(reg_read_data_2_ID)
    );
    
    Control_unit inst_Countrol_unit(
        .instruction(instruction_ID),
        .op_type(op_type_ID)
    );

    ID2EX_reg inst_ID2EX_reg(
        .clk(clk),
        .rst_n(rst_n),
        .stall(stall),
        .flush(flush),
        .op_type_next(op_type_ID),
        .address_next(inst_addr_ID),
        .register_1_next(reg_read_data_1_ID),
        .register_2_next(reg_read_data_2_ID),
        .extended_immi_next(extended_immi_ID),
        .reg_write_address_1_next(instruction_ID[20:16]),
        .reg_write_address_2_next(instruction_ID[15:11]),
        .jump_address_next({{6{1'b0}},instruction_ID[25:0]}),
        .register_1_addr_next(read_register_1_ID),
        .register_2_addr_next(read_register_2_ID),
        
        .op_type(op_type_EX),
        .address(inst_addr_EX),
        .register_1(reg_read_data_1_EX),
        .register_2(reg_read_data_2_EX),
        .extended_immi(extended_immi_EX),
        .reg_write_address_1(reg_write_address_1_EX),
        .reg_write_address_2(reg_write_address_2_EX),
        .jump_address(jump_inst_addr_EX),
        .register_1_addr(read_register_1_EX),
        .register_2_addr(read_register_2_EX)
    );
        
    ALU 
    #(
        .ADD (ADD ),
        .SUB (SUB ),
        .AND (AND ),
        .OR  (OR  ),
        .SLT (SLT ),
        .LW  (LW  ),
        .SW  (SW  ),
        .BEQ (BEQ ),
        .J   (J   )
    )
    inst_ALU(
        .op_type               (op_type_EX               ),
        .reg_write_enable_MEM  (reg_write_enable_MEM  ),
        .reg_write_enable_WB   (reg_write_enable_WB   ),
        .reg_write_address_WB  (reg_write_address_WB  ),
        .reg_write_address_MEM (reg_write_address_MEM ),
        .reg_read_data_1_EX    (reg_read_data_1_EX    ),
        .reg_read_data_2_EX    (reg_read_data_2_EX    ),
        .read_register_1_EX    (read_register_1_EX    ),
        .read_register_2_EX    (read_register_2_EX    ),
        .extended_immi_EX      (extended_immi_EX      ),
        .reg_write_data        (reg_write_data        ),
        .alu_result_MEM        (alu_result_MEM        ),
        .result                (alu_result_EX                )
    );

    EX2MEM_reg inst_EX2MEM_reg(
        .clk(clk),
        .rst_n(rst_n),
        .flush(flush),
        .op_type_next(op_type_EX),
        .shifted_address_next(shifted_address_EX),
        .alu_result_next(alu_result_EX),
        .write_mem_data_next(reg_read_data_2_EX),
        .write_reg_address_next(reg_write_address_EX),
        .jump_address_next(jump_inst_addr_EX),
        
        .op_type(op_type_MEM),
        .shifted_address(shifted_address_MEM),
        .alu_result(alu_result_MEM),
        .write_mem_data(write_mem_data_MEM),
        .write_reg_address(reg_write_address_MEM),
        .jump_address(jump_inst_addr_MEM)
    );

    MEM2WB_reg u_MEM2WB_reg(
        .clk(clk),
        .rst_n(rst_n),
        .flush(flush),
        .op_type_next(op_type_MEM),
        .read_data_next(mem_read_data_MEM),
        .alu_result_next(alu_result_MEM),
        .write_reg_address_next(reg_write_address_MEM),
        .op_type(op_type_WB),
        .read_data(mem_read_data_WB),
        .alu_result(alu_result_WB),
        .write_reg_address(reg_write_address_WB)
    );

    data_MEM inst_data_MEM(
        .clk(clk),
        .rst_n(rst_n),
        .read_addr(alu_result_MEM),
        .write_addr(alu_result_MEM),
        .write_data(write_mem_data_MEM),
        .read_enable(mem_read_enable),
        .write_enable(mem_write_enable),
        .read_data(mem_read_data_MEM)
    );
    
    PC inst_PC(
        .clk(clk),
        .rst_n(rst_n),
        .stall(stall),
        .old_inst_addr(inst_address),
        .shift_inst_addr(shifted_address_MEM),
        .jump_inst_addr(jump_inst_addr_MEM),
        .shift_enable(shift_enable),
        .jump_enable(jump_enable),
        .inst_addr(inst_address)
    );

    // Internal signal logic
    assign shift_enable = ((op_type_MEM==BEQ)&(alu_result_MEM==32'd1));
    assign jump_enable = (op_type_MEM==J);
    assign inst_addr_IF = inst_address;
    assign instruction_IF = instruction;

    assign read_register_1_ID = instruction_ID[25:21];
    assign read_register_2_ID = instruction_ID[20:16];

    assign extended_immi_ID = {{16{instruction_ID[15]}},instruction_ID[15:0]};

    assign reg_write_address_EX = (op_type_EX==LW)? reg_write_address_1_EX: reg_write_address_2_EX;
    assign shifted_address_EX = inst_addr_EX + 32'd4 + (extended_immi_EX<<2);

    assign reg_write_enable_MEM = ((op_type_MEM==ADD)|(op_type_MEM==SUB)|(op_type_MEM==AND)|(op_type_MEM==OR)|(op_type_MEM==SLT)|(op_type_MEM==LW))? 1'b1: 1'b0;
    assign mem_write_enable = (op_type_MEM==SW);
    assign mem_read_enable  = (op_type_MEM==LW);
   
    assign reg_write_enable_WB = ((op_type_WB==ADD)|(op_type_WB==SUB)|(op_type_WB==AND)|(op_type_WB==OR)|(op_type_WB==SLT)|(op_type_WB==LW))? 1'b1: 1'b0;
    assign reg_write_data = (op_type_WB==LW)? mem_read_data_WB: alu_result_WB;

    assign stall = (op_type_EX==LW)&((reg_write_address_EX==read_register_1_ID)|(reg_write_address_EX==read_register_2_ID));
    assign flush = (shift_enable|jump_enable);

endmodule
```
***
## **Simulation**
The testbench MIPS code is as followed:
```verilog
mem[0] = 32'h8C010000; // lw $1 0($0)
mem[1] = 32'h8C020001; // lw $2 1($0)
mem[2] = 32'h00221820; // add $3, $1, $2
mem[3] = 32'h00602020; // add $4, $3, $0
mem[4] = 32'h00622824; // and $5, $3, $2
mem[5] = 32'h00623025; // or $6, $3, $2
mem[6] = 32'h00812022; // sub $4, $4, $1 
mem[7] = 32'h10810001; // beq $4, $1, 1
mem[8] = 32'h08000018; // j 6
mem[9] = 32'h08000024; // j 9
```

The makefile is at [prj_file](https://github.com/YihuiCalm/A_Pipeline_MIPS_CPU/tree/main/prj_file), with the command of `make all`, you can compile with VCS and run the simulation with Verdi. The program will end in a dead loop. The simulation result is as followed:  
![Screenshot (16)](https://github.com/YihuiCalm/Pipeline_MIPS_CPU/assets/96307958/f2c03874-956b-4bb9-9a84-0ebfb09c6e58)  
The testbench output:
```
=========================================TESTBENCH===========================================
*********************************************************************************************
|        IF        |       ID        |       EX        |       MEM        |       WB        |
*********************************************************************************************
|                  |                 |                 |                  |                 |
|  lw $1 0($0)     |       ---       |       ---       |       ---        |       ---       |
|  lw $2 1($0)     |       LW        |       ---       |       ---        |       ---       |
|  add $3, $1, $2  |       LW        |       LW        |       ---        |       ---       |
|-------STALL------|------STALL------|       LW        |       LW         |       ---       |
|  add $4, $3, $0  |       ADD       |       ---       |       LW         |       LW        |
|  and $5, $3, $2  |       ADD       |       ADD       |       ---        |       LW        |
|  or $6, $3, $2   |       AND       |       ADD       |       ADD        |       ---       |
|  sub $4, $4, $1  |       OR        |       AND       |       ADD        |       ADD       |
|  beq $4, $1, 1   |       SUB       |       OR        |       AND        |       ADD       |
|  j 6             |       BEQ       |       SUB       |       OR         |       AND       |
|  j 9             |       J         |       BEQ       |       SUB        |       OR        |
|  j 9             |       J         |       J         |       BEQ        |       SUB       |
|-------FLUSH------|------FLUSH------|------FLUSH------|-------FLUSH------|------FLUSH------|
|  sub $4, $4, $1  |       ---       |       ---       |       ---        |       ---       |
|  beq $4, $1, 1   |       SUB       |       ---       |       ---        |       ---       |
|  j 6             |       BEQ       |       SUB       |       ---        |       ---       |
|  j 9             |       J         |       BEQ       |       SUB        |       ---       |
|-------FLUSH------|------FLUSH------|------FLUSH------|-------FLUSH------|------FLUSH------|
|  j 9             |       ---       |       ---       |       ---        |       ---       |
|  j 9             |       J         |       ---       |       ---        |       ---       |
|  j 9             |       ---       |       J         |       ---        |       ---       |
|-------FLUSH------|------FLUSH------|------FLUSH------|-------FLUSH------|------FLUSH------|
|  j 9             |       ---       |       ---       |       ---        |       ---       |
|  j 9             |       J         |       ---       |       ---        |       ---       |
==========================================FINISH=============================================
```






