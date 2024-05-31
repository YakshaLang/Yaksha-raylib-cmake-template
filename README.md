# Yaksha+raylib with cmake template project

This is a template project for Yaksha and raylib using CMake.

## How to use
* Download the compiler release or compile the yaksha-compiler in a parent directory of this project.
* Change compiler path in CMakeLists.txt to your compiler path. (Skip this step if compiler is located in `../Yaksha/compiler/bin/yaksha[.exe]`)
```cmake
# ------- Set this path to yaksha compiler --------
get_filename_component(YK_PATH "../Yaksha/compiler" ABSOLUTE)
# --------------------------------------------------------------------------------------------------
```
* Optional - Change project name in CMakeLists.txt to your project name. (rename `mygame` to `any_name_you_want`)
* Optional - Change engine/engine_build.c to your include your game name `.c` file. (rename `mygame.c` to `any_name_you_want.c`)

## How to build

```bash
mkdir cmake-build-release
cd cmake-build-release
cmake -DCMAKE_BUILD_TYPE=Release ..
cmake --build .
```

* On Windows you can use 'https://github.com/YakshaLang/ExePatch' to convert console application .exe to gui application .exe.
