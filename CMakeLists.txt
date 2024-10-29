

set(FREERTOS_KERNEL_PATH ${CMAKE_CURRENT_LIST_DIR}/freertos)
include(freertos/portable/ThirdParty/GCC/RP2350_ARM_NTZ/FreeRTOS_Kernel_import.cmake)

project(W55RP20_DebugProbe)


if (${PICO_SDK_VERSION_MAJOR} LESS 2)
	message(SEND_ERROR "Version 2 of the Pico SDK is required to compile this project. Please update your installation at ${PICO_SDK_PATH}")
endif ()

add_executable(W55RP20_DebugProbe
        src/probe_config.c
        src/led.c
        src/main.c
        src/usb_descriptors.c
        src/probe.c
        src/cdc_uart.c
        src/get_serial.c
        src/sw_dp_pio.c
        src/tusb_edpt_handler.c
)

target_sources(W55RP20_DebugProbe PRIVATE
        CMSIS_5/CMSIS/DAP/Firmware/Source/DAP.c
        CMSIS_5/CMSIS/DAP/Firmware/Source/JTAG_DP.c
        CMSIS_5/CMSIS/DAP/Firmware/Source/DAP_vendor.c
        CMSIS_5/CMSIS/DAP/Firmware/Source/SWO.c
        #CMSIS_5/CMSIS/DAP/Firmware/Source/SW_DP.c
        )

target_include_directories(W55RP20_DebugProbe PRIVATE
        CMSIS_5/CMSIS/DAP/Firmware/Include/
        CMSIS_5/CMSIS/Core/Include/
        include/
        )

target_compile_options(W55RP20_DebugProbe PRIVATE -Wall)

pico_generate_pio_header(W55RP20_DebugProbe ${CMAKE_CURRENT_LIST_DIR}/src/probe.pio)
pico_generate_pio_header(W55RP20_DebugProbe ${CMAKE_CURRENT_LIST_DIR}/src/probe_oen.pio)

target_include_directories(W55RP20_DebugProbe PRIVATE src)

target_compile_definitions (W55RP20_DebugProbe PRIVATE
	PICO_RP2040_USB_DEVICE_ENUMERATION_FIX=1
)

option (DEBUG_ON_PICO "Compile firmware for the Pico instead of Debug Probe" OFF)
if (DEBUG_ON_PICO)
    target_compile_definitions (W55RP20_DebugProbe PRIVATE 
	DEBUG_ON_PICO=1
    )
    set_target_properties(W55RP20_DebugProbe PROPERTIES 
        OUTPUT_NAME "debugprobe_on_pico"
    )
endif ()


target_link_libraries(W55RP20_DebugProbe PRIVATE
        pico_multicore
        pico_stdlib
        pico_unique_id
        tinyusb_device
        tinyusb_board
        hardware_pio
        FreeRTOS-Kernel
        FreeRTOS-Kernel-Heap1
)

pico_set_binary_type(W55RP20_DebugProbe copy_to_ram)

pico_add_extra_outputs(W55RP20_DebugProbe)
