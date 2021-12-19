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
            This is also confirmed by the code
             ![2](https://user-images.githubusercontent.com/25670930/146623873-a9385ba3-3836-4d96-817e-c66d8f7bfc64.PNG). This is take from chrome_exe_main_win.cc and if u are currious to read the whole code it's located at chromium/src/chrome/app . Ok, going down with the execution we can see it calls MakeMainDllLoader() to call the dll loaded class after that it launches the "loader" aka it loads the chrome.dll and if it is necessary restarts it with the necessary command lines. To further analyse the Loader we need to understand it's code, which in the same directory, in the file mail_dll_loader_win.cc .Scrolling all the way down of the file we can see the call to MakeMainDllLoader which in terms calls based on the verion you have ChromeDllLoader or ChromiumDllLoader. ![3](https://user-images.githubusercontent.com/25670930/146624258-cbb87c7e-e48c-4fbf-9280-a390a5b37010.PNG) We can see that ChromiumDllLoader is a class which inherites from MainDllLoader. We can see from the definition that the class simply does is to load the dll based on the arguments passed to cmdline and process type![6](https://user-images.githubusercontent.com/25670930/146624860-99c0fb44-2ec4-4323-bc4b-e6849e423179.PNG). Analysing the Launch method we can understand some things that happen before chrome.dll starts: We can understand from this comment that "// Launching is a matter of loading the right dll and calling the entry point.
// Derived classes can add custom code in the OnBeforeLaunch callback." launching chrome is a matter of loading a bunch of dll which actually do the work. Secondly it takes the arguments passed to command line and further it initialises the sandbox services.![q](https://user-images.githubusercontent.com/25670930/146625301-e8aea9cf-4223-4487-b5f2-e7da492a75ca.PNG). First it checks to see if it's the browser who's calling the sandbox initialisation than it checks if the process which called the initialisation of the sandbox was called as a cloud print service. It also checks if  --no-sandbox was passed to the binary,which basically tells to the binary not to run in a sandbox. it checks if wether any of these were set and if either is true it calls the sanbox with the respective options. than in the end we get to ![8](https://user-images.githubusercontent.com/25670930/146625574-0d1a67c4-c0f8-4bbc-bed6-631bad59b5f8.PNG) which is what we are interested in, this representing the wrapper to calling chrome_main.
             *Now lets see what it does. we open chromium.dll in binja 
             Based on this picture from https://blogs.igalia.com/jaragunde/files/2019/03/chrome-init-sequence.png we have a rough ideea of what should we look after in binja
             This is how the graph looks in binja![research](https://user-images.githubusercontent.com/25670930/146561718-139d5a5a-4759-4a44-98c2-2efabee4cb3d.PNG) . To be able to better track it down look for a function called ChromeMain. This is chrome's main(aka core) logic.Inside of it we can see the afro-mention phases of how chrome.dll runs at startup:
             Here we can see the first it calls some functions such as sub_180001420,sub_180017020,sub_180017020 which do some checking to see some details regarding how chrome was installed/compiled. Using the source code we can track their atributes. First function,sub_180001420 corresponds to UmaHistogramEnumeration,next we have sub_180017020 which is InitializeFromPrimaryModule and third and last one sub_180017020 is chrome_main_delegate.
             We keep repeating chrome_main_delegate but we never defined what it's purpuse is. ChromeMainDelegate is a class which inherits from ContentMainDelegate and mainly provides startup related functions and process call processing. !In case you want you can implement a custom ContentMainDelegate interface to change the default behavior of the Content module, and use a custom ChromeMainDelegate class in Chromium to customize the behavior of the startup process. 
             
              ![Capture](https://user-images.githubusercontent.com/25670930/146572896-6d40dbf2-fcec-4a6a-a0eb-13a4f26aa7df.PNG). Inside of it there are some function calls which do some xor on some data regions andconcat it with some regitry to get some options regarding chrome startup ![Capture](https://user-images.githubusercontent.com/25670930/146581324-fdb4098f-fe78-49d4-b2a4-f7aab5c36be2.PNG). Also note that function sub_180001510 is creating a scopted pointer reference with two callbacks.Not really important thing that it does but it's interesting to learn what a BindStateBase  is. We will meet it a lot in the chrome's basecode.
              Now you may be wondering what do the last three lines, precisly ![Capture9](https://user-images.githubusercontent.com/25670930/146587641-6df35a9f-4d55-4f82-912a-6a3af09900a4.PNG) do. so first line basically creates a std::unique_ptr<> for Closures. WTF is a closure!? well if it were to cyte from mozila dev" closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time." It if were to use human language it's a function whiting a function which acceses a variable withing the outher function. Eg![1](https://user-images.githubusercontent.com/25670930/146588196-84634271-c7b4-48c9-ba40-a54b92e7f057.PNG) . So in essnce it makes shure that the closure will execute. and the other two lines just set some specific behavior for dumps when chrome crashes.InstallDetails::Get().VersionMismatch() patchuie aici
              Going futher it checks for it's version at runtime and in case it doesn't match it it crashes and we arrive at the command line parsing ![commandline](https://user-images.githubusercontent.com/25670930/146590951-0043de30-11ca-448a-8928-1a7268d21696.PNG)
              Stepping inside sub_184171800 we can see it is not that big ![commandline2](https://user-images.githubusercontent.com/25670930/146591178-c44dfc19-097b-4fa3-8e64-26c0c131a324.PNG) First it takes the arguments passed to the current process, than calls a function which takes a StringPiece basically a class wrapper for std::string but a little bit cooler and that function basically checks if the binary was called headless,compares is the name of the binary which was run is chrome , checks if USE_HEADLESS_CHROME is set and than goes further into next step which is content main.
              Based on the arguments provided , content main's job is to start the respective shell.
              Now in case we don't provide any arguments to the shell the execution flow goes to ![10](https://user-images.githubusercontent.com/25670930/146630552-d76a440c-c4ae-4673-92ce-bcb7e76e52bc.PNG). In order to understand what it does we need to verify to source of it. We can find it at /src/content/app/content_main.cc
              We will have to scroll all the way to the bottom and and there we will find the definition of the ContentMain function. There we see it calls two functions. One which initialises a ContentMainRunner which is the class that handles all the "process creation" in the context of creating the browser ipc sqli net and the rest. And the other one which basically check the type of subprocesses and feed it to the ContentMainRunner class. 
              Now let's take a step back in order to understand the codeflow. First let's examine the RunContentProcess as ContentMainRunner is quite complex.First step it does is to create a GlobalActivityTracker. Omg !!! you guess it google tracks you :))) Nah just kidding pls don't sue me Google. But on a serious note this track each thread which is a thread tracker and it does a couple of cute things for better debugging such as:![Capture](https://user-images.githubusercontent.com/25670930/146662120-10749d1b-041c-45c3-87d6-deff628da350.PNG) which in essence means that there will be a unique integer used as an identifier for each thread to better understand what cause the process to break.
              eg: ```
                  kTypeIdActivityTracker = 0x5D7381AF + 4,   // SHA1(ActivityTracker) v4
                  kTypeIdUserDataRecord = 0x615EDDD7 + 3,    // SHA1(UserDataRecord) v3
                  kTypeIdGlobalLogMessage = 0x4CF434F9 + 1,  // SHA1(GlobalLogMessage) v1
                  kTypeIdProcessDataRecord = kTypeIdUserDataRecord + 0x100,
                  ```
              tracking a process life stage which is defined as 
              ```
                  // The phases are generic and may have meaning to the tracker.
              PROCESS_PHASE_UNKNOWN = 0,
              PROCESS_LAUNCHED = 1,
              PROCESS_LAUNCH_FAILED = 2,
              PROCESS_EXITED_CLEANLY = 10,
              PROCESS_EXITED_WITH_CODE = 11,

              // Add here whatever is useful for analysis.
              PROCESS_SHUTDOWN_STARTED = 100,
              PROCESS_MAIN_LOOP_STARTED = 101,
              ```
              log asap when a process exits,save the information regarding the module aka when when a module(aka a componenet of chromium) is loaded and basically the same functinolity repeated but different for different threads and classes. In case you are interested you can find it at chromium/src/base/debug/activity_tracker.h 
              Afterwards checks if "Main()" was called before. Here i think they mean if the ChromeMain() was called before. They basically check if there were passed any command to the content process was called with any arguments.And in case it was , they make sure the browser process will  get the same args.They than check for platform specific and in case windows is found the initialise they own handler by doing CreateATLModuleIfNeeded, than it's the same story checking for platform specifics and and they pass the arguments using SetupCRT and we get to the part where we initialise the ipc so we can talk to the other processes after we have spawned them. Here is the code responsabile for that. We won't go into details about it , as we will come back later discussing more indepth about the ipc mechanism of chrome. Just know this for the moment, this is the mechanism which facilitate the talking between the mutli architecture processes of chrome. And in case you don't know for what stands IPC, it's for inter process communication.![mojo](https://user-images.githubusercontent.com/25670930/146662673-32045099-b6b5-4a6c-89ff-5dd6efc4297b.PNG). Than what it does it "fordwards" rather more than sets the arguments for ui using ui::RegisterPathProvider, call the tracker and gets the result using content_main_runner->Initialize(std::move(params)) ,create a parent cosole, which i guess what they mean is that they create a parent process does some more checking and than we get to the important part which is ![Capture2](https://user-images.githubusercontent.com/25670930/146662970-9d31f08d-2d66-4b71-b344-650ded989893.PNG) . Here we see a call to a function called IsSubprocess. ![Capture3](https://user-images.githubusercontent.com/25670930/146663018-b27e2f7e-b8fe-4c48-b4fe-c3b2e7ee75c7.PNG) what it does basically to avoid the old boilderplate code it checks in one function the type of the process it receives on command line  and in case an option was passed it chooses from the following it creates the respective process. 
         ```return type == switches::kGpuProcess ||
         type == switches::kPpapiPluginProcess ||
         type == switches::kRendererProcess ||
         type == switches::kUtilityProcess || type == switches::kZygoteProcess;```
        From there we get to content_main_runner->Run(); which in essence does all the magic. Let's examine it under the hood. So content_main_runner is of class ContentMainRunner duhh.. which can be found at content\app\content_main_runner_impl.cc
         To actually be able to understand the code flow for this we have to take it from the bottom to the top. And this being said , we scroll down again and we find that actually ContentMainRunner::Create() calls infact ContentMainRunnerImpl::Create(); which makes us realise that in fact we will have to search for ContentMainRunnerImpl definition not for ContentMainRunner, which we find in content_main_runner_impl.h in the same already mentioned folder.![Capture 4PNG](https://user-images.githubusercontent.com/25670930/146663921-1188df62-5aca-4fad-a081-3c338380d3f4.PNG)
We see that it inherits from ContentMainRunner and we can see it's core methods. Really there's not too much here as it's behaviour is mostly overwritten in content_main_runner_impl.cc
         Inside content_main_runner_impl.cc, inside run method it first does some checking using DCHECK(what the hell is this function?(```The CHECK() macro will cause an immediate crash if its condition is not met. DCHECK() is like CHECK() but is only compiled in when DCHECK_IS_ON is true (debug builds and some bot configurations, but not end-user builds).``` quotes from google docs :) ) to see if is_initialized,content_main_params_,is_shutdown_ are set.![Capture6](https://user-images.githubusercontent.com/25670930/146664265-9ba18dca-8f2b-46d2-b3a2-82583a9378c3.PNG) Afterwards is gets the arguments and determines the type from which i have mentioned earlier![Capture7](https://user-images.githubusercontent.com/25670930/146664279-d5b2dfbd-c293-4a07-b5d1-22306c592dfc.PNG) Than in case we can't find that we call InitializeFieldTrialAndFeatureList()  delegate_->PostFieldTrialInitialization(); and post is as message using mojo.![Capture8](https://user-images.githubusercontent.com/25670930/146664315-a9cccbec-58ce-46ce-8c4f-603cf1fc697f.PNG)



              
              
              
              
              


