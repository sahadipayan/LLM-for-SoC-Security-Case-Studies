**Case Study 3: Vulnerability Detection in RISC-V SoCs**

To examine vulnerabilities at the SoC level and evaluate their detection and security implications using GPT-4, we utilized two SoCs based on the RISC-V architecture: PULPissimo and CVA6. PULPissimo employs a 4-stage, in-order, single-issue core, while CVA6 features a 6-stage, in-order, single-issue CPU with a 64-bit RISC-V instruction set. A commonality between them is the integrated debug module (JTAG). For the sake of this case study, our attention is primarily riveted on detecting vulnerabilities present within the debug modules of these designs. In the case of the PULPissimo SoC, the TAP controller contains the following two vulnerabilities:

- An incorrect implementation of the password-checking mechanism for JTAG lock/unlock
- The advanced debug unit examines 31 out of 32 bits of the password

In order to detect the vulnerabilities, we employed two distinct methods: blind testing and contextual testing.

- Blind testing: As its name suggests, it places the GPT model in a position where it is devoid of any explicit context. This method challenges the inherent understanding and reasoning capabilities of the model. By presenting only the debug module design, we aim to assess whether the model can, on its own, pinpoint potential vulnerabilities without any prior hints or guiding prompts.
- Contextual testing: It presents the model with a more structured framework for evaluation. Here, the GPT model is fed with prompts that provide context, aiding its understanding and analysis. The context serves as a lens, directing the focus of the model toward specific areas or features of the design. In short, this method evaluates the ability of the model to leverage provided information for a more nuanced and targeted vulnerability assessment.


For the blind test, we utilized a basic and straightforward prompt, outlined in Prompt 2.1. As anticipated, the GPT-4 model could not identify any weaknesses within the design, given that we did not include details regarding the context of the vulnerabilities of the design in the prompt. On the other hand, in the contextual testing using Prompt 2.2, the model’s response, as showcased in Response to Prompt 2.2, was more discerning. As evident in Response to Prompt 2.2, the model exhibited an enhanced aptitude for vulnerability detection and managed to identify the second vulnerability related to the bit-checking of the password. However, this increased context did not guarantee full vulnerability detection, as GPT-4 overlooked an earlier and more intricate vulnerability. This suggests that while contextual clues certainly bolster the analytical prowess of the model, they do not make it perfect. The balance between prompt specificity and model inference remains a pivotal consideration in vulnerability assessment.

*Prompt 2.1:*
```
$Input Design$
Does this module have any security issues? Describe where and why.
```
*Prompt 2.2:*
```
$Input Design$
Policy 1. Password-checking logic in the debug unit should be correct.
Policy 2. It should check all bits of the password.
Policy 3. The debug register should not be accessed in halt mode.
Check if any of these policies are violated in the module. If violated, why and where? Give a response in the following pattern:
Policy #: violated or not, explanation: if violated, why and where?, line no: violated line
```
*Response to Prompt 2.2:*
```
Policy 1: Not violated; explanation: The code contains a password-checking logic in the debug unit. It checks
if the password is correct and only allows access when the password is correct. The password-checking logic
is correctly implemented in the TAP State Machine.
Policy 2: Violated, explanation: The code does not check all bits of the password. It only checks for a
specific value (32’h0001 FFFF) and if the tdi_o is equal to the pass[bitindex]. This means that not all bits
of the password are being checked, line no: 193-198.
```
*Prompt 2.3:*
```
$Input Design$
Policy 1. Password-checking logic in the debug unit should be correct.
Check if this policy is violated in the module. If violated, why, and where?
```
*Response to Prompt 2.3:*
```
The DMI JTAG module described in your code seems to implement a kind of password checking via the PassChk and PassChkValid states and the HMAC module, but there may be a potential violation of policy 1 - the password checking logic in the debug unit should be correct. In your code, when the module enters the PassChkValid state, it checks whether the calculated HMAC hash is equal to pass_hash. If they are equal, it sets pass_check to 1 (true), else it sets it to 0 (false). This procedure is secure in itself. However, the issue lies in the fact that pass_check does not appear to block operations in case of an incorrect password. This value (pass_check) should be used to restrict access when the password does not match, but in your module, its usage is unclear, or it is not used as expected. It is set in the PassChkValid state but does not seem to prevent access in case of an incorrect password.
```
Our investigation persisted by employing a similar debug module, albeit with a distinct implementation utilized in the CVA6 SoC. This debug module also features a password-checking mechanism akin to the previous one. If valid, the pass_check signal of the design signifies the successful completion of the password-checking process. The vulnerability within this design is integrated in a manner that permits the debug module to remain unlocked indefinitely after its initial unlocking. This occurs due to the failure to reset the pass_check signal during the reset phase. We provided the design along with a little context to GPT-4, as shown in Prompt 2.3. The response depicted in Response to Prompt 2.3 initiates an intriguing discussion. While unable to pinpoint the precise location of the vulnerability (within the reset condition), the GPT model brought to light another significant flaw in the design regarding the password-checking mechanism - a flaw previously unknown to us. The code snippet featured in Listing 5 showcases the GPT’s response, revealing that the password-checking condition invariably leads to an unlocked debug session, regardless of whether the password matches successfully or not. However, with a more comprehensive prompt, complete with instructions on implementing the vulnerability, the GPT model was able to identify the actual flaw in the design.

*Listing 5:*
```verilog
PassChkValid: begin
    if(hashValid) begin 
        if(hash == pass_hash) begin 
            pass_check = 1'b1;
        end else begin
            pass_check = 1'b0;
        end
        state_d = Idle;
    end else begin
       state_d = PassChkValid;
    end  
end
```

Both investigations emphasize the efficacy of a detailed examination of an SoC, demonstrating that microscopical scrutiny yields a more accurate security assessment of the SoC design. The main challenge lies in the token constraints inherent to the GPT interface, which require us to present smaller designs or segments of designs as prompts to encapsulate a complete hardware functionality. In this context, a viable approach to provide the model with a comprehensive context involves providing the module under examination (e.g., debug module). Furthermore, it should be noted that GPT-4 can assist in uncovering previously unidentified design flaws that could be challenging to detect using conventional verification methods, significantly when constrained by tight time-to-market requirements. Through this case study, we point out two more prompt guidelines for security assessment.
