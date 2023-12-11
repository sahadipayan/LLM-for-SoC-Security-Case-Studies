**Case Study 7: Security Assertion**

The importance of security properties in formal verification is of utmost importance as they play a critical role in guaranteeing the strength and dependability of complex systems. In this case study, our objective is to demonstrate the ability of GPT-3.5 to understand security properties, expressed in natural language and presented as SVAs. To accomplish this, we presented GPT-3.5 with a collection of security properties formulated in SVA and requested it to provide a general grasp of these assertions. Our goal was to discern whether the model could not only parse the technical syntax but also extract and convey the underlying intent of these assertions.

*Prompt 5.1:*
```systemverilog
property p1;
@(posedge clk)RSA binary.F SM ==
RSA binary.IDLE && rose(start)
| => (RSA binary.F SM == RSA binary.IN IT )
##1(RSA binary.F SM == RSA binary.LOAD1)
##1(RSA binary.F SM == RSA binary.LOAD2)
##1(RSA binary.F SM == RSA binary.M U LT )
##1(RSA binary.F SM == RSA binary.SQR)
##1(RSA binary.F SM == RSA binary.M U LT )
##1(RSA binary.F SM == RSA binary.SQR)
##1(RSA binary.F SM == RSA binary.M U LT )
##1(RSA binary.F SM == RSA binary.SQR)
##1(RSA binary.F SM ==
RSA binary.RESU LT );
endproperty
```
*Response to Prompt 5.1:*

As an example, Prompt 5.1 showcases one of the SVA-based security properties given to GPT-3.5 for a design of the controller of RSA shown in Figure 9. GPT-3.5 was asked what this property actually checks. “Response to Prompt 5.1” displays the final portion of the reply by GPT-3.5. From the response, it becomes evident that GPT-3.5 not only recognizes the structural components of the property but also grasps the semantic implications of the assertion. The response of the model accurately captures the essence of the property, highlighting its role in verifying the correct sequence of state transitions in the FSM. This shows the potential of GPT-3.5 as a tool for understanding and possibly helping design and verification of complex digital systems.
