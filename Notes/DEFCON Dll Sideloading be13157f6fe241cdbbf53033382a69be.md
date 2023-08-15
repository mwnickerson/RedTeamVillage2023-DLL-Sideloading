# DEFCON Dll Sideloading

### Other Tools

[https://www.redteam.cafe/red-team/dll-sideloading/dll-sideloading-not-by-dllmain](https://www.redteam.cafe/red-team/dll-sideloading/dll-sideloading-not-by-dllmain)

[https://github.com/shantanu561993/DLL-Sideload](https://github.com/shantanu561993/DLL-Sideload)

ExportDumper.exe - [https://github.com/iilegacyyii/ExportDumper](https://github.com/iilegacyyii/ExportDumper)

giant list of sideloads - [https://github.com/XForceIR/SideLoadHunter/tree/main/SideLoads](https://github.com/XForceIR/SideLoadHunter/tree/main/SideLoads)

---

# Slide notes

## **We Found One! Now what?**

- Create a testing Dll that will be named as the missing one
- Place it in the writable path of the missing Dll
- Run the exe and look for signs that the Dll was sideloaded by the application
    - Message Box opens
    - program crashes
- Application will call DllMain
    - DllMain is an optional entry-point function for Dlls
    - THis function is called when a Dll is loaded or unloaded by a program
- If program is trying to borrow functions from the original Dll and ours doesn’t have them
- It breaks
- ⚠️ This can alert the user
- To be stealthy we can proxy the functions the program **needs** to the original Dll using pragma comments
    - which tells the linker during compilation that these are dependencies

## Confirm Sideload

- Since its not a console app you can’t `printf` to test
- Compile a Dll that creates a MessageBox on the screen
- Crash is good sign
- MessageBox is a good sign

## Proxy Dll Function calls

- To view the exported functions of a Dll
    - Load it into CFF explorer (in VM)
- Write down all the exported function names
- Add #pragma comment headers for each function
- Our Dll will refer to the linked functions when they are called by the Program

## Pragma who?

- Tells the linker to link libraries as dependencies during compilation
- /export:function name is the function name exported in your Dll
- C:\og\dll.function name is the decorated or full path of the function you want to export it to
- @1 is the ordinal value of the function, basically alternate way of referring to it inside the Dll

## Creating a malicious Dll

- Install
    - Visual Studio
    - C++ Development Kit
- Launch and select C++ Dynamic Link Library
- Call a target function inside of DllMain in the `DLL_PROCESS_ATTACH` **case**
- Create a new thread for that target function
- Be careful not to deadlock the DllMain function
- Certain actions perfocan “deadlock” a process
- Create a Dll that executes when its loaded (DllMain)
    - DllMain entry point -  is triggered *when it (Dll) is loaded or unloaded using the **[LoadLibrary](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya)** and **[FreeLibrary](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-freelibrary)** functions.*
- DllMain has a few different *******reasons******* for being called:
    - DLL_PROCESS_ATTACH
    DLL_THREAD_ATTACH
    - DLL_THREAD_DETACH
    - DLL_PROCESS_DETACH

`DLL_PROCESS_ATTACH` is the initialization function that is called when the Dll is loaded into a new process.

- Your function should execute, but you might break the process

![Untitled](DEFCON%20Dll%20Sideloading%20be13157f6fe241cdbbf53033382a69be/Untitled.png)

# Proxying Functions

`#pragma comment( lib, "emapi" )` - search for the EMAPI.LIB library while linking 

- ⭐These directives are often used to instruct the linker to perform certain actions or link specific libraries during the linking phase of compilation.
- When the binary calls the original Dll functions, our malicious Dll proxies functions to the legitimate one.
- Therefore not breaking the host process

```jsx
#pragma comment(linker, "/export:GetFileVersionInfoA=C:\\Windows\\System32\\version.GetFileVersionInfoA,@1")
```

---

- With a staged Meterpreter payload

## Creating Simple Dll Shellcode Loader

- Create shellcode execution function
- Start new thread with target function in `DllMain` →`DLL_PROCESS_ATTACH`
- Add pragma comments
- Generate shellcode with msfvenom (EXITFUNC thread)
- Many things can/should be done to increase the evasive-ness of this loader for actual engagements

## Putting it all together

- Package all together for a phishing scenario
- Drop Dll for a stealthier scenario
- should be unnoticable to user and execute desired code

## Beyond Calc.exe

- Becareful not to deadlock host process
    - certain functions (LoadLibrary) inside DllMain can deadlock
- Match C2 protocols with sideloaded App
- comment out a proxy and create your own definition for that function (instead of DllMain execution)

## Automating the process

DLLSideloader - [https://github.com/Flangvik/DLLSideloader](https://github.com/Flangvik/DLLSideloader)

SharpDLLProxy - [https://github.com/Flangvik/SharpDllProxy](https://github.com/Flangvik/SharpDllProxy)

comments.py -

[https://github.com/shantanu561993/DLL-Sideload/blob/main/Python%20Scripts/comment.py](https://github.com/shantanu561993/DLL-Sideload/blob/main/Python%20Scripts/comment.py)