set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_VERSION 1)
cmake_minimum_required(VERSION 3.20)

SET(CMAKE_C_COMPILER_WORKS TRUE)
SET(CMAKE_CXX_COMPILER_WORKS TRUE)

set(CMAKE_C_COMPILER arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER arm-none-eabi-g++)
set(CMAKE_ASM_COMPILER arm-none-eabi-gcc)
set(CMAKE_AR arm-none-eabi-ar)
set(CMAKE_OBJCOPY arm-none-eabi-objcopy)
set(CMAKE_OBJDUMP arm-none-eabi-objdump)
set(SIZE arm-none-eabi-size)
set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)

project(MSP432E4 C CXX ASM)

message(NOTICE "-- PATH {SIMPLELINK_MSP432E4_SDK}: $ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}")
message(NOTICE "-- PATH {GCC_ARMCOMPILER}: $ENV{GCC_ARMCOMPILER}")

set(LINKER_SCRIPT $ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source/ti/devices/msp432e4/linker_files/gcc/msp432e401y.lds)
set(STARTUP_FILE $ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source/ti/devices/msp432e4/startup_system_files/gcc/startup_msp432e401y_gcc.c)
set(SYSTEM_FILE $ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source/ti/devices/msp432e4/startup_system_files/system_msp432e401y.c)
set(DRIVER_LIB $ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source/ti/devices/msp432e4/driverlib/lib/gcc/m4f/msp432e4_driverlib.a)

add_compile_options(
	-DDeviceFamily_MSP432E4
	-D__MSP432E401Y__
	-mcpu=cortex-m4
	-march=armv7e-m
	-mthumb
	-std=c99
	-mfloat-abi=hard
	-mfpu=fpv4-sp-d16
	-ffunction-sections
	-fdata-sections
	-g
	-gstrict-dwarf
	-Wall
)

add_link_options(
	-Wl,-T,${LINKER_SCRIPT}
	-march=armv7e-m
	-mthumb
	-mfloat-abi=hard
	-mfpu=fpv4-sp-d16
	-static
	-Wl,--gc-sections
	-lgcc
	-lc
	-lm
	-lnosys
	--specs=nano.specs
)

include_directories(
	$ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source
	$ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source/ti/
	$ENV{SIMPLELINK_MSP432E4_SDK_INSTALL_DIR}/source/third_party/CMSIS/Include
	$ENV{GCC_ARMCOMPILER}/arm-none-eabi/include/newlib-nano
	$ENV{GCC_ARMCOMPILER}/arm-none-eabi/include
)

file(GLOB_RECURSE SOURCES App/*.*)

add_executable(
	${PROJECT_NAME}.elf
	${SOURCES}
	${STARTUP_FILE}
	${SYSTEM_FILE}
)

target_link_libraries(
	${PROJECT_NAME}.elf
	${DRIVER_LIB}
)

set(HEX_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.hex)
set(BIN_FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}.bin)

add_custom_command(TARGET ${PROJECT_NAME}.elf POST_BUILD
	COMMAND ${CMAKE_OBJCOPY} -Oihex $<TARGET_FILE:${PROJECT_NAME}.elf> ${HEX_FILE}
	COMMAND ${CMAKE_OBJCOPY} -Obinary $<TARGET_FILE:${PROJECT_NAME}.elf> ${BIN_FILE}
	COMMENT "Building ${HEX_FILE}\nBuilding ${BIN_FILE}")
