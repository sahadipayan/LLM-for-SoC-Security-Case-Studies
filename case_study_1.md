**Case Study 1: Design Generation**

We start the discussion with a very simple example of small design generation in Verilog through LLM. We directed a simple prompt at GPT-3.5, seeking the Verilog representation of a Moore FSM consisting of three states. A portion of the outcome, showcased in the **Listing 1**, exhibits a syntax error. We can notice that in the design, the 'state_out' output signal has conflicting definitions as both wire and reg. Although syntax error is an unusual event in GPT-3.5-generated Verilog designs, a closer look at large-scale outputs reveals occasional syntax issues. For example, sometimes GPT-3.5 generated design contains syntactical problems. Usage of obsolete constructs, missing parenthesis for multiple statements, or the insertion of inappropriate or extraneous punctuations are some of them.

*Listing 1:*
```verilog

module MooreFSM(
  input wire clk,
  input wire reset,
  (output wire reg state_out)
);

  // Define the states
  parameter S0 = 2'b00;
  parameter S1 = 2'b01;
  parameter S2 = 2'b10;
  
  // Define the next state and output for each state
  reg [1:0] current_state, next_state;
  
  always @(posedge clk or posedge reset) begin
    if (reset)
      current_state <= S0; // Reset to initial state
    else
      current_state <= next_state; // Move to the next state
  end
```



A very effective solution to prevent such syntactical error in GPT-generated design is to use the self-scrutiny mechanism of LLM itself. For example, in this case, we used a self-checking prompt that forces the model to review its own output. We added a simple self-debugging question at the end of the prompt saying, "Is there any issue regarding syntax, coding style, and synthesis? If yes, correct the problems". By doing so, the model becomes its own first line of defense, potentially identifying and rectifying issues before the output reaches the end user. This self-checking mechanism serves as an internal audit, improving the reliability and integrity of the generated design. A portion of the refined output design is shown in *Listing 2*, and it exhibits no syntactical error as before. As discussed, this idea of self-checking has been successful in reasoning tasks. However, it should be mentioned that although this self-scrutiny is valuable, it is not an absolute panacea. There may still be higher-level logical or design errors that this immediate review does not capture. But by integrating the self-evaluative capabilities of LLM into the design generation process, we can obviously establish a more robust initial filter, ensuring the production of higher-quality and more reliable outputs. Such observation motivates us to formulate our first prompting guideline for design generation using LLM.

*Listing 2:*
```verilog
module MooreFSM(
  input wire clk,
  input wire reset,
  output wire reg state_out
);

  // Define the states
  parameter S0 = 2'b00;
  parameter S1 = 2'b01;
  parameter S2 = 2'b10;
  
  // Define the next state and output for each state
  reg [1:0] current_state, next_state;
  
  always @(posedge clk) begin
    if (reset)
      current_state <= S0; // Reset to initial state
    else
      current_state <= next_state; // Move to the next state
  end
  
  always @(current_state) begin
    case (current_state)
      S0: begin
        // Define the next state and output for S0
        next_state <= S1;
        state_out <= 0; // Set output to 0 in S0
      end
``` 

**Prompt Guideline 1:** *Incorporate a self-scrutiny step at the end of the prompt to ensure rigorous revision and rectification of potential design or coding issues.*

Following this guideline serves a dual purpose. Firstly, such a self-regulatory mechanism streamlines the output, making it more aligned with standard coding practices. Secondly, it instills an added layer of confidence in the generated result.
