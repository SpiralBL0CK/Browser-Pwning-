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

And such we define the following goals in order to be able to get started with browser hacking

 | Goals  | 
| ------------- | 
| Learn,map,master the internal structures used by V8 engine and the key points of Chrome architecture | 
| Learn and master the basics procedures necessary for browser exploitation  | 

# Learn,map,master the internal structures used by V8 engine and the key points of Chrome architecture 
  At the first stage of the project it's necessary to gather as much knowledge regarding Chrome Architecture and how each componenet interact with each other. In order to better understand this we need to the Chromium Project into multiple subcomponenets such that we are able to isolate everything and analyse it properly. More precisely in how many subcomponents each of the following components breaks 
  V8 Architecture
  Chromium Architecture
  Blink Architecture
  

Now the most logical step for a first step is understanding chromium architecture.
   So ok we want to exploit the browser, but what happends when we first start the browser? Well after clicking the chromium executable , the executable starts some processes.
   The order and their name is as follows:
   
     First one is called browser process. What does this process do ?
          * well it is the central coordinator of all the processes, and operates at the highest privilege level available to the browser. It is also know as the startup process.
          * it is also the main process
          * it's lifespan is throught the browser's entire lifetime.
          * it controls features such as the address bar, bookmarks and the back/forward/reload buttons.
          * it does not trust the data given to it by any of the other processes
          * if another process would like to do a more highly privilleged action such as interracting with the UI,filesystem storage or network it does that for the respective process and it handles back the results to that process.
          
          Now that we briefly know what it does it is time to go more indepth on it:
            *So the way Chromium works on Windows at least, is that it compiles the files into a dll , and after that it loads it into memory. So the core logic of Chroium browser is in chromium.dll.
             *Now lets see what it does. we open chromium.dll in binja 
             Based on this picture from https://blogs.igalia.com/jaragunde/files/2019/03/chrome-init-sequence.png we have a rough ideea of what should we look after in binja
             This is how the graph looks in binja![research](https://user-images.githubusercontent.com/25670930/146561718-139d5a5a-4759-4a44-98c2-2efabee4cb3d.PNG) . To be able to better track it down look for a function called ChromeMain. This is chrome's main(aka core) logic.Inside of it we can see the afro-mention phases of how chrome.dll runs at startup:
             Here we can see the first it calls some functions such as sub_180001420,sub_180017020,sub_180017020 which do some checking to see some details regarding how chrome was installed/compiled. Ater that we can see a call to a function which i renamed ChromeMainDelegate.
              ![Capture](https://user-images.githubusercontent.com/25670930/146572896-6d40dbf2-fcec-4a6a-a0eb-13a4f26aa7df.PNG). Inside of it there are some function calls which do some xor on some data regions andconcat it with some regitry to get some options regarding chrome startup ![Capture](https://user-images.githubusercontent.com/25670930/146581324-fdb4098f-fe78-49d4-b2a4-f7aab5c36be2.PNG). Also note that function sub_180001510 is creating a scopted pointer reference with two callbacks.Not really important thing that it does but it's interesting to learn what a BindStateBase  is. We will meet it a lot in the chrome's basecode.
              Now you may be wondering what do the last three lines, precisly ![Capture9](https://user-images.githubusercontent.com/25670930/146587641-6df35a9f-4d55-4f82-912a-6a3af09900a4.PNG) do. so first line basically creates a std::unique_ptr<> for Closures. WTF is a closure!? well if it were to cyte from mozila dev" closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time." It if were to use human language it's a function whiting a function which acceses a variable withing the outher function. Eg![1](https://user-images.githubusercontent.com/25670930/146588196-84634271-c7b4-48c9-ba40-a54b92e7f057.PNG) . So in essnce it makes shure that the closure will execute. and the other two lines just set some specific behavior for dumps when chrome crashes.



             


