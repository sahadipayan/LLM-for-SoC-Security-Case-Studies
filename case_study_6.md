**Case Study 6: Testbench Generation**

In the domain of security verification, the development of testbenches is often time-consuming, especially when dealing with substantial designs. Therefore, we present an investigation into the creation of test benches for security verification using ChatGPT. These automatically generated testbenches are aimed at detecting CWEs and other potential security vulnerabilities within a design. The principal objective of these generated testbenches is to identify counterexamples, which demonstrate violations of targeted CWEs or security rules.

To harness ChatGPT-generated test benches for security verification, it is crucial to validate their syntactic and semantic correctness, as well as their ability to detect security weaknesses. Consequently, we generated a set of examples to evaluate these testbenches. We pursued a comparative analysis by generating two versions of test cases: one incorporating the security weakness and the other devoid of it. Subsequently, we executed simulations of the design and the corresponding testbenches using Modelsim.

We first consider CWE-835, which highlights the security frailties associated with an unreachable loop. To expose this security vulnerability, a test case generated within a testbench should illustrate entry into the loop without any possibility of exit. One example of the generated Verilog file of the Testbench by ChatGPT is demonstrated in Listing 9. In the context of CWE-835, the testbench successfully identified the presence of the unreachable loop and effectively flagged it within the printed message.

*Listing 9:*
```systemverilog
module testbench;    
    reg KR, DS, clock, reset;
    wire [2:0] current_state, next_state;
    
    FSM uut (.KR(KR), .DS(DS), 
    .clock(clock), .reset(reset),
     .current_state(current_state),
     .next_state(next_state));
    
    integer i;
    
    initial begin
      clock = 0;
      forever #5 clock = ~clock;
    end
    
    initial begin
      reset = 1;
      #10 reset = 0;
    end
    
    initial begin
      // Wait for the FSM to reach the WAIT_KEY state
      while(current_state != 3'h1) begin
        #5;
      end
      
      // Generate input sequences to test for CWE 835
      KR = 1; DS = 0; // should transition to WAIT_KEY
      #5 KR = 0;
      #5 KR = 1; DS = 1; // should transition to 
      //INITIAL_ROUND
      #5 KR = 0;
      #5 KR = 1; DS = 1; // should transition to DO_ROUND
      #5 KR = 0;
      #5 KR = 1; DS = 1; // should transition to FINAL_ROUND
      #5 KR = 0;
      #5 KR = 1; DS = 1; // should transition to WAIT_DATA
      #5 KR = 1; DS = 0; // should transition to WAIT_KEY
      
      // Check for CWE 835 by ensuring that the testbench 
      //does not get stuck in a loop
      for(i = 0; i < 10; i = i + 1) begin
        #5;
        if(current_state == 3'h1) begin
          $display("CWE 835 detected!");
          $finish;
        end
      end
      
      $display("No CWE 835 detected.");
      $finish;
    end

endmodule
```
Turning to the CWE570 (expression is always false) scenario, where an always-false expression was examined, the testbench printed message shows that the final state was never reached, but the conditions to reach that state were not satisfied. As it is shown in the following figure, the reset is constantly active, making it impossible to transition to the next state. Therefore, the printed message that shows that the state was never reached is not a counterexample.
![image](https://github.com/sahadipayan/LLM-for-SoC-Security-Case-Studies/assets/89291347/5dc0f675-6240-45bd-b5b8-aaa512232b55)


There are indeed several advantages to employing ChatGPT for the automated generation of testbenches, particularly when compared to the manual authoring of these testbenches. LLM-generated testbenches offer notable advantages, especially in scenarios involving large and intricate designs. They are tailored specifically to a given design, enhancing efficiency. Furthermore, they exhibit versatility in language translation, with the ability to seamlessly transition between languages such as Verilog, SystemVerilog, and iVerilog, facilitated by straightforward prompts. Additionally, this approach accommodates design and verification engineers who may not possess expertise in a particular language, empowering them to craft testbenches in that language.

Nevertheless, our assessments have unveiled certain challenges associated with these automatically generated testbenches. Initial attention is required to ensure the syntactic and semantic correctness of the testbench. Subsequently, the design and associated testbench necessitate simulation in a separate software environment, such as Siemens ModelSim. Following this, a thorough analysis of the test output waveforms is essential to ascertain the successful detection or non-detection of vulnerabilities.

Our case studies revealed minor syntactic and semantic issues, including clock generation errors, design instantiation problems, and the instantiation of non-I/O port signals. Addressing these issues is feasible; however, a comprehensive assessment is necessary to gauge the test bench’s effectiveness in confirming the presence of security vulnerabilities. Although ChatGPT-generated test benches showed promising results, their overall capacity to detect security vulnerabilities remains inadequate. This observation parallels the conclusions drawn from a previous study on ChatGPT-generated test benches for verification of functionality in ChipGPT; it revealed challenges in their creation and the need for modifications. These limitations are rooted in the insufficient availability of suitable training data within the test bench and verification code domain.
