# Browser-Pwning
A proper well structured documentation for getting started with chrome pwning &amp; v8 pwning 

# Structure of document
| How this doc is organised  | 
| ------------- | 
| 1. Motivation | 
| 2. Actual Study Material | 


# Motivation

Browsers are one of today’s most used pieces of technology. On every off the shelf computer, if we just plug and play we will see a browser installed. That’s why from an attacker’s perspective and a threat model perspective it’s very rewaradable if the attacker is able to compromise the browser through a malicious page. Given the upper arguments I choose to study Google’s Javascript Engine, particularly v8. 

Given the gigantic nature of v8’s project,I choose as a starting point the interpreter, namely d8.Although extensive research has been done already on d8, we hope to at least find one bug and if not be able to move forward with research on browser exploitation since v8 provides entry ground for the basic exploit development strategies used on browser exploitation .

 Another reason why I choose v8 as a target is that it’s used across multiple browsers. 
 
 If we look under the hood we can see that the engine is used in MicrosoftEdge too, so there is a multiple bug bounty award chance. For the underlying operating system   the researcher will be using a mix of windows and linux since there’s no restriction in terms of obtaining a shell since v8 bugs allow code execution through wasm pages and it’s not bound to any platform specific. 
 
 
Unfortunately, while an exploited bug in v8 would result in code execution, we won’t be able to execute any code due to the sandbox, and as such we would get code execution in the context of the renderer,which would not allow us to execute code on the machine. For that we would need another exploit for the sandbox,thus we would need a full chain in order to exploit the system.
 
 | Goals  | 
| ------------- | 
| Learn,map,master the internal structures used by V8 engine and the key points of Chrome architecture | 
| Learn and master the basics necessary for browser exploitation  | 
