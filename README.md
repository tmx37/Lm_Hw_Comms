# embedded_module_template
My own embedded module template for driver and relative logics development.

## Requirements
- GCC Compiler + arm-none-eabi packages (https://developer.arm.com/downloads/-/arm-gnu-toolchain-downloads)
- CMake (>= 3.21) + Ninja
- A C compiler: host GCC, or the ARM toolchain shipped with the STM32Cube extension
- West: `pip3 install --user -U west`

## Module libs
Libraries only for module compilation on separate development.
They are not part of this repo: they are fetched by west (see `west.yml`) into `module_libs/`.

| Project | Repo | Revision | Path |
|---|---|---|---|
|  |  |  |  |

## West initialization
Run `west init -l` from **inside** the cloned repo. West uses the repo's own `west.yml`
and makes the **parent folder** the workspace (it creates `.west/` there, next to the repo).

```sh
cd <workspace>                                   # e.g. ~/Documents/Code
git clone git@github.com:tmx37/embedded_module_template.git
cd embedded_module_template
west init -l                                     # creates <workspace>/.west/config -> Drv_BMP180/west.yml
west update                                      # fetches GenUtl into Drv_BMP180/module_libs/
```

Check with `west list`:
```
manifest     embedded_module_template                   HEAD
GenUtl       embedded_module_template/module_libs       develop
```

Run `west update` again whenever `west.yml` changes or to pull the latest module libs.

## Compile
When this repo is built on its own (it is the top-level CMake project), CMake also builds
`cfg_ex/` as the mocked configuration + related west-pulled modules.

### VS Code (CMake Tools)
Just select the right kit, then build:
1. `CMake: Select a Kit` and pick one of:
   - **GCC x86_64** (host, e.g. `GCC 16.2.1 x86_64-redhat-linux`): quick compile check on the PC
   - **STM32Cube ARM GCC** (`~/.local/share/stm32cube/bundles/gnu-tools-for-stm32/<version>/bin/arm-none-eabi-gcc`): real target build
2. If you switched kit, run `CMake: Delete Cache and Reconfigure`
3. `CMake: Build`

> Do **not** use the distro `arm-none-eabi-gcc` kit (`/usr/bin/arm-none-eabi-gcc`) unless its C library is
> installed (`sudo dnf install arm-none-eabi-newlib` on Fedora): without it `stdint.h` / `math.h` are missing.
> To add the STM32Cube toolchain as a kit, add an entry to `~/.local/share/CMakeTools/cmake-tools-kits.json`
> with `"compilers": { "C": "<path to its arm-none-eabi-gcc>" }`.

### Command line
```sh
cmake -S . -B build -G Ninja                                      # host GCC
# cmake -S . -B build -G Ninja -DCMAKE_C_COMPILER=<arm-none-eabi-gcc>   # ARM
cmake --build build
```

