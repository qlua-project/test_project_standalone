QUIK-LUA-INTEGRATION(7)        Project Management       QUIK-LUA-INTEGRATION(7)

NAME
       Quik-Lua-Integration - Standardized 3-tier Scope Architecture

SYNOPSIS
       Managing cross-environment projects for QUIK and standalone LUA 
       interpreters using industry-standard layouts and tiered scopes.

DESCRIPTION
       This architecture addresses behavioral divergence between embedded 
       QUIK and standalone LUA environments.
       
       Decision Drivers:
       1. CWD Consistency: QUIK sets CWD to the script folder; LUA retains 
          the User's CWD. Bootstrapping ensures path portability.
       2. Path Isolation: Prevents polluting the QUIK directory by utilizing a 
          dedicated "Shared" library structure.
       3. Standardized Layout: Adopting the LuaRocks "lua_modules" tree 
          ensures interoperability with modern LUA tooling.

DEFAULT PATH ANALYSIS
       Approach use a multi-tiered search strategy. Paths are relative 
       to the QUIK installation directory (QUIK) and the script folder (.):

       Paths (package.path):
           QUIK\lua\?.lua
           QUIK\lua\?\init.lua
           QUIK\?.lua
           QUIK\?\init.lua
           QUIK\..\share\lua\5.x\?.lua
           .\?.lua
           .\?\init.lua

       C-Paths (package.cpath):
           QUIK\?.dll
           QUIK\..\lib\lua\5.x\?.dll
           QUIK\loadall.dll
           .\?.dll

3-TIER SCOPE ENVIRONMENT
       Paths are resolved in descending order of priority:

       Tier 1: Local Project Scope
           Repository-specific modules. Managed via a local "lua_modules" 
           tree and the project "src" folder.

       Tier 2: Shared Environment Scope (qlua-shared-library)
           A dedicated sibling project for shared 3rd-party dependencies 
           and common modules. Avoids direct QUIK directory modification.

       Tier 3: QUIK Global Scope
           Implicit paths defined by the embedded Lua interpreter 
           (see PATH ANALYSIS).

PROJECT ARCHITECTURE
       Structure your project at the root level to support both QUIK and 
       standard LUA invocation:

       ProjectRoot/
       ├── lua_modules/      <-- LOCAL DEPENDENCIES (Tier 1)
       │   ├── lib/lua/5.x/  <-- Rock DLLs
       │   └── share/lua/5.x/ <-- Rock Lua files
       ├── src/              <-- INTERNAL SOURCE (Tier 1)
       │   └── core/         <-- Submodules and business logic
       ├── externals/        <-- SOURCE REPOS
       │   └── openssl/      <-- C-library headers/libs for compilation
       ├── main.lua          <-- ENTRY POINT (Tier 1)
       ├── libssl.dll        <-- Unmanaged DLLs (OS-level)
       └── zlib.dll          <-- Unmanaged DLLs (OS-level)

SCOPE MANAGEMENT (BOOTSTRAP)
       Place this script in the Tier 3 directory (QUIK/Lua/) to allow 
       global access via require().

       QUIK/Lua/bootstrap.lua (Doc Version)
              function M.init(project_root)
                  -- Add Local Project Paths (src + lua_modules)
                  package.path = project_root .. "/src/?.lua;" .. package.path
                  package.path = project_root .. "/lua_modules/share/lua/5.x/?.lua;" .. package.path
                  package.cpath = project_root .. "/lua_modules/lib/lua/5.x/?.dll;" .. package.cpath

                  -- Add Shared Environment Paths (Tier 2)
                  local shared_root = project_root .. "/../qlua-shared-library"
                  package.path = shared_root .. "/lua_modules/share/lua/5.x/?.lua;" .. package.path
                  package.cpath = shared_root .. "/lua_modules/lib/lua/5.x/?.dll;" .. package.cpath
              end

       Usage in Project main.lua:
              require("bootstrap").init(getScriptPath())
              require("core/logic")

FILES
       QUIK/Lua/
           Primary shared location for global Lua scripts and 
           bootstrap.lua logic (Tier 3).

       ../qlua-shared-library/
           The "Shared Environment" sibling directory (Tier 2).

       ProjectRoot/main.lua
           Entry point script. Must sit at the root to resolve unmanaged 
           DLLs from the same directory via standard Windows search order.

SEE ALSO
       Official [ARQA QLua Guide](https://arqatech.com/en/products/quik/capabilities/integration/programming-languages/)
       [LuaRocks Project Layout](https://martin-fieber.de/blog/lua-project-setup-with-luarocks/)

QUIK-LUA-INTEGRATION           February 2026           QUIK-LUA-INTEGRATION(7)
