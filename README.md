# SecureLuaVirtualMachine

SLVM (SecureLuaVirtualMachine) is a secure Lua virtual machine for Roblox based on the FiOne Project by Rerumu to provide a controlled execution environment for Lua code, mainly for games allowing the execution of untrusted usercode.

> [!NOTE]  
> This project was originally created by [AstonishedLiker](https://github.com/AstonishedLiker).  
> It has been reuploaded and maintained here following the project's abandonment.
>
> This repository's purpose is to preserve and distribute the module via [Wally](https://wally.run/) for continued community use.

## Features

- Closure Hooking — modify how functions behave within the sandbox
- Read/Write Hooks — restrict access to important properties (e.g., `game.Parent`) using safe string-checking
- Env Table for Custom Globals — supply your own environment variables to the VM
- Call Sandboxing — isolate each function call to prevent attack via `xpcall` or call-stack tricks
- Loop Throttling — safeguard against infinite loops and high-iteration crashes

## Usage

### Basic Usage

```lua
local SLVM = require(Packages.SLVM)

--> First argument is `ChunkName` which is a string, second argument is `Env` which provides additional globals to the Virtual Machine. Both of them are optional.
local LuaVM = SLVM.LuaVM.new("UntrustedCode@" .. Player.Name, {
    CodeInvoker = Player, --> Adds new variable to environment
    _VERSION = "Lua 1.2.3" --> Overwrites built-in global "_VERSION"
})

--> Run is just a shortcut to LuaVM:Compile(Code)(...) which includes error handling 
local Number = LuaVM:Run("print('Hello, Secure Lua Virtual Machine!'); return ...", 123) --> will return 123

--> Direct compiling of the code
local CompiledFunc, FailReason = LuaVM:Compile("return function(MyNumber) return MyNumber * 10; end")
if CompiledFunc then
    print("Result with calling 10: " .. CompiledFunc(10)) --> Result with calling 10: 100
else
    warn("Error compiling code: " .. FailReason)
end
```

### Hooking Closures

```lua
--> Hook a closure (in this example, print) inside the virtual machine
local OriginalPrint; OriginalPrint = LuaVM:ReplaceClosure(print, function(...)
    return OriginalPrint("Hooked via SLVM!", ...)
end)

LuaVM:Run("print(123)") --> "Hooked via SLVM! 123"
```

### Hooking Read & Write on objects

> [!WARNING]  
> Only use `LuaVM:CheckCString` with the **first argument** being the string you want to check  
> (for example `"Parent"`) and the **second argument** being the `"Index"` or another unknown string.  
>
> **Do not switch these arguments!**  
> Reversing them will change the behavior of the function.

```lua
-- /!\ If an instance is passed to AddXHook, it will automatically apply that to all instances.
-- The following code snippet blocks the manipulation and reading of "Parent".
LuaVM:AddReadHook(game, function(self, Index, NormalValue)
    if type(Index) == "string" then
        if LuaVM:CheckCString("Parent", Index) or LuaVM:CheckCString("parent", Index) then
            error("Not allowed to read Parent", 2)
        end
    end

    return NormalValue
end)

LuaVM:AddWriteHook(game, function(self, Index, NewValue)
    if type(Index) == "string" then
        if LuaVM:CheckCString("Parent", Index) or LuaVM:CheckCString("parent", Index) then
            error("Not allowed to write Parent", 2)
        end
    end

    return NewValue
end)
```

### Managing Additional Settings

```lua
--> Check if a specific additional setting is enabled
local IsSettingEnabled = LuaVM:IsAdditionalSettingEnabled(SLVM.Enum.AdditionalSettings.SandboxCalls)

--> Enable or disable additional settings
LuaVM:EnableAdditionalSetting(SLVM.Enum.AdditionalSettings.ThrottleLoopInstructions)
LuaVM:DisableAdditionalSetting(SLVM.Enum.AdditionalSettings.ThrottleLoopInstructions)
```

## License

This project is licensed under the [GNU General Public License v3.0](LICENSE), allowing for free and commercial usage, modification, and distribution.
