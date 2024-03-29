**Case Study 4: Security Evaluation in FSM Design**

In this group of case studies, we primarily perform investigations on the competence of GPT in two scenarios: 1) calculation of security metrics and 2) security assessment.

**Case Study IV-A:** Security Metric Calculation: In this case study, we primarily perform investigations on the competence of GPT in the detection of security rule violations. The security metric plays an important role in the generation of secure designs and vulnerability detection. Calculating a security metric meticulously at RTL-level demands a proper understanding of design and the ability of arithmetic calculation. In this light, we check the competence of GPT in the calculation of a security metric named Fault-Injection Feasibility (FIF) [57, 65] in this case. Drawing from the study [57], Kibria et al. set forth a security guideline: when a state transition occurs between two consecutive unprotected states, the Fault-Injection Feasibility (FIF) metric should be ‘0’. A design with a FIF metric of 1 for such states is deemed susceptible to fault injection. FIF metric can be calculated by 
$$FIF = \prod_{i=0}^{n-1} \left( (b_{x_i} \oplus b_{y_i}) + (b_{x_i} \cdot b_{p_i}) \right)$$

where bx<sub>i</sub>, by<sub>i</sub>, and bp<sub>i</sub> denote bits of the present, next, and protected states at the position i, respectively; the symbols ⊕, + and · represent the operations XOR, OR, and AND, respectively. From the equation, we observe that the calculation of the FIF metric is not a straightforward task. Given its complexity, we evaluated the proficiency of GPT-3.5 in deriving this particular security metric. To do this, our approach to guiding GPT-3.5 mirrored the principles of multi-step reasoning. Without making the decision in a single step, we methodically segmented the task into sequential phases. Each phase is intrinsically linked, with the result from one acting as the foundation for the subsequent. This systematic breakdown not only simplifies the process for the model but also given the intricate nature of the FIF metric, it potentially increases the accuracy and efficiency of the computation by model. By affording GPT-3.5 this structured pathway, we enhance its chances of producing a more reliable and accurate output.

To compute the FIF metric, we employ the subsequent three-step methodology:

- Listing all transitions that occur between unprotected states.
- For every such transition, identify the values of bxi, byi, and bpi for each respective bit position.
- Using Equation 1 to calculate the FIF metric for each of these listed state transitions.

We formulate three sequential prompts to execute the above-mentioned steps. The functionality of each prompt is discussed below:

- Prompt 3.1: In the initial phase of Prompt 3.1, we feed GPT-3.5 with the input design, identifying the protected state. We then guide the model to enumerate all state transitions and subsequently filter out those involving the protected state. Through this approach, we generate a list of state transitions with state encoding exclusive to unprotected states via prompting.
- Prompt 3.2: In the subsequent phase, the list of unprotected state transitions, along with their state encodings from the previous step, is introduced to Prompt 3.2. Initially, we explain the concept of FIF metric by presenting its corresponding equation and a brief description. This ensures that GPT-3.5 comprehends the exact information needed to compute this metric. To further aid understanding, we illustrate with a concise example, demonstrating how to derive the values of bx, by, and bp from a given state transition. Concluding the prompt, we task GPT-4 with computing the values for every state transition, instructing it to display the results in the prescribed format.
- Prompt 3.3: In this final step, we initialize the prompt with the definition of the FIF metric. This provides the model with the requisite background knowledge. To ensure that GPT-4 grasps the computational intricacies involved, we illustrate with a detailed example explaining the process of calculating the FIF metric using values of bx, by, and bp from a representative state transition. Building on this understanding, we then direct GPT-4 to process the information generated in the preceding steps. Tasked with calculating the FIF metric in alignment with the provided instructions and example, we underscore the importance of adhering to the specified tabular format.

*Listing 6:*
```systemverilog
always @(*) begin 
    next_state = s0; 
    case (curr_state)
        so: begin
            sbit = 0;
            if(start) next_state = s1;
            else next_state = s0;
        end
        s1: begin
            sbit = ctrl;
            if (ctrl) next_state = s4;
            else next_state = s2;
        end
        s2: begin
            next_state = s0;
        end
        s3: begin 
        if(finish) next_state = s0;
        else next_state=s3;
        end
        s4: begin
            if (sbit) next_state = s1;
            else next_state = s2;
        end
    endcase
end
endmodule
```
We meticulously devised this prompting strategy to compute the FIF metric across a large amount of FSM designs. For illustrative purposes, our chosen FSM design contains 7 states. The state transition graph (STG) of this design mirrors the one depicted in the following figure. ”Result” is considered as the protected state. The outcomes generated by Prompt 3.1, Prompt 3.2, and Prompt 3.3 elucidate the step-wise systematic approach through which GPT-3.5 adeptly tackles the computationally intensive task of determining the FIF metric. To ensure clarity in our demonstration, we opted to abstain from displaying outputs for each individual state transition. A critical observation from this case study is the indispensable role of multi-stage prompting coupled with a structured tabular data representation in meticulously calculating such security metrics. This led to two more prompt guidelines for the calculation of extensive SoC security tasks.
![image](https://github.com/sahadipayan/LLM-for-SoC-Security-Case-Studies/assets/89291347/77dec53e-c32d-4046-af06-80269982f1bc)



**Case Study IV-C:** Security Assessment through Open-ended Question: In the preceding investigations, namely Case Study IV-A, our emphasis was on providing detailed prompts to GPT-3.5, targeting security metric computations. This approach was necessitated due to the relative limitations of GPT-3.5 in executing advanced reasoning tasks. Contrarily, in the present study, we prefer conducting a security assessment on GPT-4 using open-ended questions, deliberately avoiding any predefined prompting strategies.

The FSM design under scrutiny in this study is presented in Listing 6. A thorough analysis of this design reveals that the state “s3” within the FSM stands isolated, devoid of any incoming transitions, making it an unreachable state. This configuration is not merely a design flaw; it poses a significant security risk. An attacker might leverage these states to communicate or transfer information without detection, thus compromising confidentiality. With this design as input, we prompted GPT-4, succinctly asking, “*Is there any issue with this design?*”. GPT-4’s feedback is visualized in the following figure. Impressively, GPT-4 identified a multitude of design issues, including the critical observation of the unreachable state. Beyond this primary concern, GPT-4 further pinpointed other design elements needing improvement, showcasing its comprehensive design evaluation capabilities.
![image](https://github.com/sahadipayan/LLM-for-SoC-Security-Case-Studies/assets/89291347/f37721e6-6918-44e4-b8c1-9b001dfe527f)


In our evaluations spanning various scenarios, including static deadlock, dynamic deadlock, absence of default statement, and race condition, we consistently employed open-ended questions with GPT-4. Remarkably, GPT-4 demonstrated a significantly higher success rate in precisely assessing security concerns, with the exception of dynamic deadlock situations. Through these case studies, it became evident that GPT-4 possesses the capability to deeply dissect and comprehend FSM designs, promptly identifying simple and not-so-complicated design imperfections. Furthermore, contrasting its predecessors, GPT-4 accentuates its advanced reasoning skills by adeptly responding to open-ended inquiries without the necessity of iterative prompting.
