In this exploration, we aim to assess how adept GPT-4 is at identifying and fixing vulnerabilities in FSM designs. We provided GPT-4 with an FSM design that had clear breaches of two important security rules related to state transitions and the use of default statements. Our approach in querying GPT-4 was strategic: we briefed the model about these security rules and the security assessment, where we mentioned how the rules are violated in the provided design. Subsequently, we tasked GPT-4 with amending the design to ensure compliance with these rules. Additionally, we asked GPT-4 to review its own revised design, drawing from our approach in Case Study I. This method not only checks GPT-4’s skill in addressing problems but also its ability to double-check its work, ensuring that corrections are both safe and aligned with recommended practices. Importantly, we refrained from providing GPT-4 with any specific mitigation strategies, requiring the model to derive solutions autonomously. A sample of the prompt is outlined in Prompt 4.1.

*Prompt 4.1:*
```
<Input Design>
In this case, assume WAIT KEY is the protected state, and other states are unprotected.
There are two security rules:
1. All unused states of a control FSM should be handled through the ‘default’ statement in the RTL description
2. When state transition takes place between two consecutive unprotected states, the hamming distance between the states should be 1.
Security Assessment:
These two rules are violated in this design in the following way:
1. The is no ‘default’ statement through which unused states ’101’, ’110’, and ’111’ are handled.
2. There are following two state transitions between unprotected states where the hamming distance is not 1.
WAIT DATA - INDIAL ROUND : 001 - 010 : HD=2 DO ROUND - FINAL ROUND: 011 - 100: HD=3
Violation Mitigation Instructions:
Modify the FSM design so that the rules are followed. While modification the STG graph remains the same. For the modified design, check if is there any rule violation in the provided design. If yes, continue modifying until two rules are followed in the modified design.
```
Listing 7 shows the input design used in this case study. Here, ‘WAIT KEY’ is considered the protected state. From careful observation, it becomes evident that the input design has two violations of security rules. At first, there is no default statement to handle the unspecified states. This can lead to unpredictable behavior when the system encounters an unspecified state. This unpredictability can be exploited by attackers. Secondly, the hamming distance between unprotected states becomes greater than 1 in the following transitions: like ‘WAIT DATA’ to ‘INITIAL ROUND’ and ‘DO ROUND’ to ‘FINAL ROUND’. 

*Listing 7:*
```systemverilog
module fsm_module(KR, DS, clk, reset);
	input clk; // clock signal
	input reset; // reset input
	input KR, DS;
	(*@\color{red}\textbf{parameter WAIT\_KEY = 3'b000, WAIT\_DATA = 3'b001,}@*) 
        (*@\color{red}\textbf{INITIAL\_ROUND = 3'b010, DO\_ROUND = 3'b011,}@*) 
        (*@\color{red}\textbf{FINAL\_ROUND =  3'b100;}@*)
	reg [2:0] current_state, next_state; 
	always @(current_state, KR, DS)
	begin
	 case(current_state) 
		 WAIT_KEY: begin
		  if(KR == 1) next_state = WAIT_DATA;
		  else next_state = WAIT_KEY;
		 end
		 WAIT_DATA: begin
		  if(DS == 0) next_state = WAIT_KEY;
		  else next_state = INITIAL_ROUND;
		 end
		 INITIAL_ROUND: begin
		  if(KR == 0) next_state = WAIT_KEY;
		  else next_state = DO_ROUND;
		 end 		 
		 DO_ROUND: begin
		  if(KR == 0) next_state = WAIT_KEY;
		  else next_state = FINAL_ROUND;
		 end		 
		 FINAL_ROUND:
		 begin
		  if(KR == 0) next_state = WAIT_KEY;
		  else next_state = WAIT_DATA;
		 end
    
	 endcase
	end
	// sequential memory of the Moore FSM
	always @(posedge clk, posedge reset)
	begin
	 if(reset == 1) 
		current_state <= WAIT_KEY;
	 else
		current_state <= next_state; 
	end 
endmodule
```
*Listing 8:*
```systemverilog
module fsm_module(KR, DS, clk, reset);
    input clk; // clock signal
    input reset; // reset input
    input KR, DS;
    (*@\color{olive}\textbf{parameter WAIT\_KEY = 3'b000, WAIT\_DATA = 3'b001,}@*)
    (*@\color{olive}\textbf{INITIAL\_ROUND =3'b011, DO\_ROUND = 3'b010,}@*)
    (*@\color{olive}\textbf{FINAL\_ROUND = 3'b110;}@*)
    reg [2:0] current_state, next_state; 

    always @(current_state, KR, DS)
    begin
        case(current_state) 
            WAIT_KEY: begin
                if(KR == 1) next_state = WAIT_DATA;
                else next_state = WAIT_KEY;
            end
            WAIT_DATA: begin
                if(DS == 0) next_state = WAIT_KEY;
                else next_state = INITIAL_ROUND;
            end
            INITIAL_ROUND: begin
                if(KR == 0) next_state = WAIT_KEY;
                else next_state = DO_ROUND;
            end      
            DO_ROUND: begin
                if(KR == 0) next_state = WAIT_KEY;
                else next_state = FINAL_ROUND;
            end         
            FINAL_ROUND:
            begin
                if(KR == 0) next_state = WAIT_KEY;
                else next_state = WAIT_DATA;
            end
             (*@\color{olive}\textbf{default: next\_state = WAIT\_KEY; }@*)
             (*@\color{olive}\textbf{// default to protected state}@*)
        endcase
    end

    // sequential memory of the Moore FSM
    always @(posedge clk, posedge reset)
    begin
        if(reset == 1) 
            current_state <= WAIT_KEY;
        else
            current_state <= next_state; 
    end 
endmodule
```

This scenario is particularly vulnerable to fault injection attacks. The generated design by GPT-4 is shown in Listing 8. It can be seen that the default statement has been successfully added to mitigate the security issue. For the second security breach, GPT-4 used gray encoding to restructure the current state encodings. This strategy successfully addressed two of the three highlighted vulnerabilities. However, the transition from FINAL ROUND to WAIT DATA remains a concern, with a Hamming distance of 3. While GPT-4 did not entirely eliminate this vulnerability, its approach did reduce the associated risk. The application of gray encoding, as suggested by GPT-4, is indeed a viable solution to rectify such design flaws. This demonstrates the ability of GPT-4 to autonomously derive effective solutions without external guidance. Ideally, to fully address the security concern, gray encoding should be applied to unprotected states initially, followed by the assignment of protected states. The solution provided by GPT-4 closely aligns with this optimal mitigation strategy. With more explicit context on the mitigation strategy, it is plausible that GPT-4 could have devised a complete countermeasure. It is worth noting that, the mitigation of the second security violation is complex. In some cases, it may be challenging to find an encoding arrangement that fully adheres to this security rule.
