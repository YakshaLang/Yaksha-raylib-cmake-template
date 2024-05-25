cmake_minimum_required(VERSION 3.27)
project(mygame)

set(CMAKE_C_STANDARD 99)
set(CMAKE_CXX_STANDARD 17)

# --------------------------------------------------------------------------------------------------
# Raylib

# Generate compile_commands.json
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Dependencies
set(RAYLIB_VERSION 5.0)
find_package(raylib ${RAYLIB_VERSION} QUIET) # QUIET or REQUIRED
if (NOT raylib_FOUND) # If there's none, fetch and build raylib
    include(FetchContent)
    FetchContent_Declare(
            raylib
            DOWNLOAD_EXTRACT_TIMESTAMP OFF
            URL https://github.com/raysan5/raylib/archive/refs/tags/${RAYLIB_VERSION}.tar.gz
    )
    FetchContent_GetProperties(raylib)
    if (NOT raylib_POPULATED) # Have we downloaded raylib yet?
        set(FETCHCONTENT_QUIET NO)
        FetchContent_Populate(raylib)
        set(BUILD_EXAMPLES OFF CACHE BOOL "" FORCE) # don't build the supplied examples
        add_subdirectory(${raylib_SOURCE_DIR} ${raylib_BINARY_DIR})
    endif ()
endif ()

# --------------------------------------------------------------------------------------------------

get_filename_component(CMAKE_BINARY_DIR "." ABSOLUTE)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

# ------- Set this path to yaksha compiler --------
get_filename_component(YK_PATH "../Yaksha/compiler" ABSOLUTE)
# --------------------------------------------------------------------------------------------------

set(YK_RT_PATH "${YK_PATH}/runtime")
set(YK_COMPILER_PATH "${YK_PATH}/bin/yaksha")
get_filename_component(CODE_PATH "." ABSOLUTE)
set(main_program "${CODE_PATH}/main.yaka")
set(yaksha_dialogs "${CODE_PATH}/engine/dialogs.yaka")
set(mygame "${CODE_PATH}/mygame.c")

set(YAKSHA_RUNTIME_FILES
        ${YK_RT_PATH}/_include_bstr.h ${YK_RT_PATH}/_include_io.h ${YK_RT_PATH}/_include_mman_win.h ${YK_RT_PATH}/argparse.h ${YK_RT_PATH}/cr.h ${YK_RT_PATH}/cr_userdata.h ${YK_RT_PATH}/endianness.h ${YK_RT_PATH}/ini.h ${YK_RT_PATH}/stmmr.h ${YK_RT_PATH}/subprocess.h ${YK_RT_PATH}/tinycthread.h ${YK_RT_PATH}/toml.h ${YK_RT_PATH}/utf8proc.h ${YK_RT_PATH}/wasm4.h ${YK_RT_PATH}/whereami.h ${YK_RT_PATH}/yk__argparse.h ${YK_RT_PATH}/yk__arrayutils.h ${YK_RT_PATH}/yk__console.h ${YK_RT_PATH}/yk__cpu.h ${YK_RT_PATH}/yk__graphic_utils.h ${YK_RT_PATH}/yk__lib.h ${YK_RT_PATH}/yk__process.h ${YK_RT_PATH}/yk__system.h ${YK_RT_PATH}/yk__utf8iter.h ${YK_RT_PATH}/yk__wasm4_libc.h ${YK_RT_PATH}/argparse.c ${YK_RT_PATH}/toml.c ${YK_RT_PATH}/utf8proc.c ${YK_RT_PATH}/whereami.c ${YK_RT_PATH}/yk__argparse.c ${YK_RT_PATH}/yk__arrayutils.c ${YK_RT_PATH}/yk__console.c ${YK_RT_PATH}/yk__cpu.c ${YK_RT_PATH}/yk__graphic_utils.c ${YK_RT_PATH}/yk__main.c ${YK_RT_PATH}/yk__process.c ${YK_RT_PATH}/yk__system.c)


include_directories(engine/3rd/tinyfiledialogs ${YK_RT_PATH})


# --------------------------------------------------------------------------------------------------
# Compile Yaksha source
add_custom_command(OUTPUT ${mygame}
        COMMAND ${YK_COMPILER_PATH} compile ${main_program} > ${mygame}
        COMMAND ${CMAKE_COMMAND} -E touch ${mygame}
        DEPENDS ${main_program} ${yaksha_dialogs}
        COMMENT "Compiling Yaksha source code for mygame")

add_custom_target(compile_yaksha_code DEPENDS ${mygame}
        COMMENT "Checking if re-compilation is required")
# --------------------------------------------------------------------------------------------------
# Compile Project
add_executable(${PROJECT_NAME} ${YAKSHA_RUNTIME_FILES} engine/3rd/tinyfiledialogs/tinyfiledialogs.c engine/engine_build.c)


IF (WIN32)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -static")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -static")
    target_compile_definitions(${PROJECT_NAME} PUBLIC YK_IGNORE_WINDOWS_CONSOLE_SET_MODE_ERROR)
ELSE ()
    target_link_libraries(${PROJECT_NAME} m)
ENDIF ()
add_dependencies(${PROJECT_NAME} compile_yaksha_code)
target_compile_features(${PROJECT_NAME} PRIVATE c_std_99)
target_compile_definitions(${PROJECT_NAME} PUBLIC UTF8PROC_STATIC)
target_link_libraries(${PROJECT_NAME} raylib)

# --------------------------------------------------------------------------------------------------
# Raylib
# Web Configurations
if (${PLATFORM} STREQUAL "Web")
    # Tell Emscripten to build an example.html file.
    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".html")
endif ()

# Checks if OSX and links appropriate frameworks (Only required on MacOS)
if (APPLE)
    target_link_libraries(${PROJECT_NAME} "-framework IOKit")
    target_link_libraries(${PROJECT_NAME} "-framework Cocoa")
    target_link_libraries(${PROJECT_NAME} "-framework OpenGL")
endif ()

# --------------------------------------------------------------------------------------------------