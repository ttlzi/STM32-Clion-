cmake_minimum_required(VERSION 3.26)
set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)

set(scatter ${CMAKE_SOURCE_DIR}/MDK-ARM/Temp/Temp.sct)

add_link_options(
        --library_type=microlib
        --strict
        --summary_stderr
        --info summarysizes,sizes,totals,unused,veneers
        --map
        --load_addr_map_info
        --xref
        --callgraph
        --symbols
)
add_link_options(
        --scatter=${scatter}
)

set(CPU Cortex-M3)
set(CMAKE_C_FLAGS "-mcpu=${CPU} -std=c11 -xc -c")
set(CMAKE_CXX_FLAGS "-mcpu=${CPU} -std=c++11 -xc++ -c")
set(CMAKE_ASM_FLAGS "-mcpu=${CPU} -masm=auto -gdwarf-4 -c")

add_compile_options(
        --target=arm-arm-none-eabi
        -fno-rtti
        -fsigned-char
        -fshort-enums
        -fshort-wchar
        -ffunction-sections
        -Wno-packed
        -Wno-missing-variable-declarations
        -Wno-missing-prototypes
        -Wno-missing-noreturn
        -Wno-sign-conversion
        -Wno-nonportable-include-path
        -Wno-reserved-id-macro
        -Wno-unused-macros
        -Wno-documentation-unknown-command
        -Wno-documentation
        -Wno-license-management
        -Wno-parentheses-equality
        -Wno-reserved-identifier
)

add_compile_definitions(
        -D__MICROLIB
        -D__UVISION_VERSION="538"
        -D_RTE_
        -DSTM32F10X_MD
        -D_RTE_
        -DUSE_HAL_DRIVER
        -DSTM32F103xB
        -DMD
)

#设置工程名
project(Temp C CXX ASM)

#设置C标准版本
set(CMAKE_C_STANDARD 11)

#设置C++标准版本
set(CMAKE_CXX_STANDARD 11)

#这里这个是设置debug版本的，但是在这没用，后续我会说明
if ("${CMAKE_BUILD_TYPE}" STREQUAL "Release")
    message(STATUS "Maximum optimization for speed")
    add_compile_options(-Ofast)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "RelWithDebInfo")
    message(STATUS "Maximum optimization for speed, debug info included")
    add_compile_options(-Ofast -g)
elseif ("${CMAKE_BUILD_TYPE}" STREQUAL "MinSizeRel")
    message(STATUS "Maximum optimization for size")
    add_compile_options(-Os)
else ()
    message(STATUS "Minimal optimization, debug info included")
    add_compile_options(-O1 -DDEBUG)
endif ()

#头文件路径
include_directories(
        Core/Inc
        Drivers/CMSIS/Device/ST/STM32F1xx/Include
        Drivers/CMSIS/Include
        Drivers/STM32F1xx_HAL_Driver/Inc/Legacy
        Drivers/STM32F1xx_HAL_Driver/Inc
        MDK-ARM/RTE/_Temp

        App/Soft/SRAM
        App/Hard/DISPLAY
)

# 源文件
file(GLOB_RECURSE SOURCES
        Core/Src/*.c
        Drivers/STM32F1xx_HAL_Driver/Src/*.c
        MDK-ARM/*.s

        App/Soft/SRAM/*.c
        App/Hard/DISPLAY/*.c
)

# lib库
#link_directories(Drivers/xxx/Source)
#link_libraries("${CMAKE_CURRENT_SOURCE_DIR}/Drivers/xxx/Source/xxx.lib")

add_executable(${PROJECT_NAME} ${SOURCES})

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)
set(MAP_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.elf.map)

add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${ARMCLANG_FROMELF} --output ${HEX_FILE} --i32 $<TARGET_FILE:${PROJECT_NAME}>
        COMMAND ${ARMCLANG_FROMELF} --output ${BIN_FILE} --bin $<TARGET_FILE:${PROJECT_NAME}>
        COMMENT "Building ${HEX_FILE}\r\nBuilding ${BIN_FILE}")

#add_compile_definitions(ARM_MATH_CM4;ARM_MATH_MATRIX_CHECK;ARM_MATH_MATH_ROUNDING)
#add_compile_options(-mfloat-abi=hard -mfpu=fpv4-sp-d16)
#
#add_compile_options(-float-abi=soft)
