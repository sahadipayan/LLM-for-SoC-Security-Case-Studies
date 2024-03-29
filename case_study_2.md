In this study, we explore the proficiency of GPT-3.5 in embedding vulnerabilities into hardware designs. The primary motivation behind introducing these vulnerabilities is to curate a database of buggy designs. Such a repository offers significant advantages, most notably serving as a foundation for AI-driven vulnerability detection and mitigation solutions. Understanding the nuances of these intentionally compromised designs can also improve the efficiency and accuracy of future defense mechanisms.

In this case, we focus on inserting vulnerabilities into FSM design. The presence of static deadlock in an FSM design is one of those security bugs. A static deadlock in an FSM can lead to denial of service (DoS), unintended data leakage, and exploitable system inconsistencies, compromising the overall security and reliability of the system. Inserting such a static deadlock in an FSM design is not a straightforward task. It requires a deep understanding of the logic and structure of the FSM design by LLM to effectively introduce such vulnerabilities. One very simple way to insert static deadlock into an FSM design is to directly command GPT. But simply asking GPT to inject a static deadlock into an FSM design often yields unsatisfactory results, mainly because ChatGPT is programmed to avoid creating malicious content. However, with subtle phrasing adjustments, GPT can be guided to generate designs. However, this seemingly direct method often results in unsuccessful attempts, especially in the case of GPT-3.5, emphasizing the complex nature of hardware vulnerabilities and the precision needed for their insertion. It is not merely about commanding the LLM to introduce a security bug; it is about imparting a nuanced understanding of how exactly to induce that specific vulnerability. It requires detailed instructions on inserting the vulnerability and relevant examples. The idea of including reference examples in the prompt is reminiscent of one-shot or few-shot learning in the context learning paradigm. Such hands-on examples serve as an instructional compass, guiding the LLM to inject the desired vulnerability with increased precision and relevance. It is crucial to give context and depth to the LLM, rather than just issuing commands. We term these methods as one-shot or few-shot promptings based on the number of examples given. The concept of one-shot prompting is depicted in the following figure:

![concept of one-hot prompting](https://github.com/sahadipayan/LLM-for-SoC-Security-Case-Studies/assets/89291347/222790b6-5416-497b-b630-0438a2ba6913)


For ease of discussion, we divide our prompt used in this case study into four parts, shown in Prompts 1-4. Here, we discuss the functions of these prompts:

- Prompt 1.1: Starting with Prompt 1.1, we present the input design and outline the scope of the task to GPT. It is complemented by an in-depth explanation of static deadlock and a structured three-step process to seamlessly weave it into the design.
- Prompt 1.2: Next, in Prompt 1.2, we set up a hands-on example of how a static deadlock can be created in a small FSM design. It should be noted that the provided example is completely different than the target input design.
- Prompt 1.3: Prompt 1.3 is dedicated to refining the model-generated design. Many of these post-processing steps mentioned in Prompt 1.3 are not necessarily, in general. Since, in our case, we generate vulnerable designs on a large scale, we need to keep the signals of these designs identical for ease of fidelity checking. Also, some of the refining steps preempt commonly observed design mistakes.
- Prompt 1.4: Prompt 1.4 performs a self-review and prints out the output design. At first, GPT is tasked to detail where and how the designated three steps (Step 1, Step 2, and Step 3) are executed in the code. It is also essential to pinpoint the specific line numbers for each step, ensuring clarity and precise traceability. Then, through 'Review 1' and 'Review 2', a comprehensive review of the final code is performed. Prompt 1.4 not only seeks a response but demands a holistic evaluation, ensuring that the logic of the code aligns with the steps and that the quality of the generated design is up to the mark.

*Prompt 1.1:*
```
Your task is to perform the following actions:
Now, read the following Verilog code delimited by <>
Code: <Input Design>
Modify the code by introducing/adding a static deadlock state to the existing state transitions in the case statement.
Static deadlock refers to a situation when the FSM enters to that state from another state,cit will not be
able to come out from that state. To do this,
Step 1: From the parameter list, first select a state from the state transition graph.
Step 2: Change its state transition in the combinational block so that it connects to a new state called deadlock_state.
Step 3: Add new deadlock_state state in the case statement that has a self-connecting loop.
```

*Prompt 1.2:*
```verilog
//For example, 
// Before deadlock:

    parameter X=3'b000, Y=3'b001, Z=3'b011; 
     case(current_state) 
         X: begin
            next_state=Y;
         end
 
        Y: begin
          next_state=Z;
        end
 
        Z: begin
          next_state=X;
        end
        
     endcase
```
```verilog
//After deadlock: 

   case(current_state) 
        X: begin
         if (start)
         next_state=Y;
         else
            next_state=deadlock_state;
         end
        Y: begin
          next_state=Z;
        end 
        Z: begin
          next_state=X; 
        end           
        deadlock_state: begin
          next_state=deadlock_state; 
        end
    endcase

//Here, when X transits to deadlock_state, FSM cannot get out of it.
```

*Prompt 1.3:* 
```
Now implement the deadlock in the provided code. Always implement a deadlock state in the case statement. Do not modify the sequential block. Take care of the following steps:
1) Do not use semicolon(;) after “end” keyword
2) For multiple statements, always use begin..end
3) Use parameter instead of local parameter
4) Put a semicolon after the declaration of reg
5) Put a semicolon at the end of each statement
6) Mark the clock signal as “clk” and reset the signal as “rst”
7) Make the module name “fsm_module”
```
*Prompt 1.4:*
```
When giving a response, only write in the following
format delimited by [ ]. Make sure that all three steps are followed.
Explanation: Where and how have Step 1, Step 2, and Step 3 been followed in the code? Tell me the line no
also where step 1, 2, and 3 has been implemented.
Review 1: Have you implemented Step 2 in the case statement block? If not, rewrite the whole code by case
statement block as shown in the provided code.
Review 2: Is there any issue regarding syntax, coding style, and synthesis? If so, correct the problems.
[ code: < modified code > ]
```

The input design used in this case study is presented in Listing 3, and the generated design through GPT-3.5 is outlined in Listing 4. Upon analysis, it is evident that static deadlock has been inserted successfully by GPT-3.5. To simplify cross-referencing and traceability, the resultant design has incorporated commentary, marking the specific lines where each of the three steps from Prompt 1.1 has been executed. This feature, in itself, showcases the attention to detail by GPT-3.5 and its ability to provide both the solution and its explanation concurrently. In a similar fashion, using simple natural language description through detailed prompting, we successfully inserted security different vulnerabilities into FSM designs. Because of such success, we formulated another prompting guideline for vulnerability insertion using LLM. Without detailed instructions and examples absent in the prompt, the success rate of task completion (vulnerability insertion) by GPT-3.5 drops significantly. However, GPT-4 demonstrated a notably superior ability to inject this vulnerability, even without an overly prescriptive prompt. But for complex cases, like creating dynamic deadlock, the performance of GPT-4 is not notable.

*Listing 3:*
```verilog
module fsm_module (
  input clk,
  input reset,
  input coin,
  input productSelected,
  output reg dispenseItem
);

  parameter IDLE = 3'b000;
  parameter ACCEPTING_COINS = 3'b001;
  parameter PRODUCT_SELECTED = 3'b010;
  parameter DISPENSING_ITEM = 3'b011;

  reg [2:0] current_state;
  reg [2:0] next_state;

  always @(posedge clk or posedge reset) begin
    if (reset) begin
      current_state <= IDLE;
    end else begin
      current_state <= next_state;
    end
  end

  always @(*) begin
    case (current_state)
      IDLE:
        begin
          dispenseItem  = 0;
          next_state     = ACCEPTING_COINS;
          if (productSelected) begin
            next_state = PRODUCT_SELECTED;
          end
        end
      ACCEPTING_COINS:
        begin
          dispenseItem  = 0;
          next_state     = ACCEPTING_COINS;
          if (coin) begin
            next_state = PRODUCT_SELECTED;
          end
          if (productSelected) begin
            next_state = PRODUCT_SELECTED;
          end
        end
      PRODUCT_SELECTED:
        begin
          dispenseItem  = 0;
          next_state     = DISPENSING_ITEM;
          if (!productSelected) begin
            next_state = ACCEPTING_COINS;
          end
        end
      DISPENSING_ITEM:
        begin
          dispenseItem  = 1;
          next_state     = IDLE;
        end
      default:
        begin
          dispenseItem  = 0;
          next_state     = IDLE;
        end
    endcase
  end
endmodule
```
*Listing 4:*
```verilog
module fsm_module (
  input clk,
  input reset,
  input coin,
  input productSelected,
  output reg dispenseItem
);

  parameter IDLE = 3'b000;
  parameter ACCEPTING_COINS = 3'b001;
  parameter PRODUCT_SELECTED = 3'b010;
  parameter DISPENSING_ITEM = 3'b011;
  parameter DEADLOCK_STATE = 3'b100; 
  //adding new state

  reg [2:0] current_state;
  reg [2:0] next_state;

  always @(posedge clk or posedge reset) begin
    if (reset) begin
      current_state <= IDLE;
    end else begin
      current_state <= next_state;
    end
  end

  always @(current_state) begin
    case (current_state)
      IDLE:
        begin
          dispenseItem  = 0;
          if (productSelected) begin
            next_state = PRODUCT_SELECTED;
          end
          else begin
            next_state = DEADLOCK_STATE; 
            //step 2: connecting IDLE to deadlock_state
          end
        end
      ACCEPTING_COINS:
        begin
          dispenseItem  = 0;
          next_state     = ACCEPTING_COINS;
          if (coin) begin
            next_state = PRODUCT_SELECTED;
          end
          if (productSelected) begin
            next_state = PRODUCT_SELECTED;
          end
        end
      PRODUCT_SELECTED:
        begin
          dispenseItem  = 0;
          next_state     = DISPENSING_ITEM;
          if (!productSelected) begin
            next_state = ACCEPTING_COINS;
          end
        end
      DISPENSING_ITEM:
        begin
          dispenseItem  = 1;
          next_state     = IDLE;
        end
      DEADLOCK_STATE: //step 3: adding deadlock state
        begin
          dispenseItem = 0;
          next_state = DEADLOCK_STATE;
        end
      default:
        begin
          dispenseItem  = 0;
          next_state     = IDLE;
        end
    endcase
  end
endmodule
```
It should be mentioned that during this experiment, we intentionally kept the temperature parameter 0, which makes the process very deterministic. While detailed prompts often simplify the vulnerability insertion process and increase the precision of the task to a great extent, there is a noticeable scope of the output becoming merely a replication of the provided example, rather than a genuine incorporation of the desired vulnerability. Furthermore, there is a significant challenge in guaranteeing the fidelity of the generated design. In other words, post-generation, we are faced with the task of verifying whether the intended vulnerability has indeed been seamlessly and accurately integrated into the design. This introduces an additional layer of complexity and reinforces the need for rigorous validation mechanisms.

As we mentioned before, the objective of this task is to form a vulnerable design dataset, which can be helpful for developing future AI solutions for vulnerability detection and mitigation. Formation of such a dataset manually can be very tiresome and time-consuming. But such quality of GPT can become a double-edged sword. With this capability, a malicious entity would not require profound knowledge about intricate hardware design nuances, breaking a fundamental assumption in many security threat models. Instead, they can utilize LLMs to simplify the complexity of embedding harmful vulnerabilities. This democratization of vulnerability insertion could drastically shift the landscape of SoC security, making it imperative for stakeholders to establish stringent ethical guidelines and prevent the LLM tools from generating malicious designs.
