cmake_minimum_required(VERSION 3.12)

include(pico_sdk_import.cmake)

set(FREERTOS_KERNEL_PATH ${CMAKE_CURRENT_LIST_DIR}/freertos)

set(PICO_BOARD_HEADER_DIRS ${CMAKE_CURRENT_LIST_DIR}/boards)

if (DEFINED ENV{PICO_BOARD})
        set(PICO_BOARD $ENV{PICO_BOARD})
        message("Using PICO_BOARD from environment ('${PICO_BOARD}')")
else ()
        set(PICO_BOARD "voltix_probe")
        message("Using default PICO_BOARD ${PICO_BOARD}")
endif()

include(FreeRTOS_Kernel_import.cmake)

project(voltixprobe)

pico_sdk_init()

add_executable(voltixprobe
        src/main.c
        src/usb_descriptors.c
        src/swd_transport.c
        src/cdc_uart.c
        src/get_serial.c
        src/sbw_transport.c
        src/sbw_jtag.c
        src/sbw_device.c
        src/probe_vendor.c
        )

target_sources(voltixprobe PRIVATE
        CMSIS_5/CMSIS/DAP/Firmware/Source/DAP.c
        CMSIS_5/CMSIS/DAP/Firmware/Source/JTAG_DP.c
        CMSIS_5/CMSIS/DAP/Firmware/Source/SWO.c
        #CMSIS_5/CMSIS/DAP/Firmware/Source/SW_DP.c
        )

target_include_directories(voltixprobe PRIVATE
        CMSIS_5/CMSIS/DAP/Firmware/Include/
        CMSIS_5/CMSIS/Core/Include/
        include/
        )

target_compile_options(voltixprobe PRIVATE -Wall)


target_include_directories(voltixprobe PRIVATE src)

target_compile_definitions (voltixprobe PRIVATE
	PICO_RP2040_USB_DEVICE_ENUMERATION_FIX=1
        PICO_DEFAULT_UART_TX_PIN=28
)

target_link_libraries(voltixprobe PRIVATE
        pico_multicore
        pico_stdlib
        pico_unique_id
        tinyusb_device
        tinyusb_board
        FreeRTOS-Kernel
        FreeRTOS-Kernel-Heap1
)

pico_set_binary_type(voltixprobe copy_to_ram)

pico_add_extra_outputs(voltixprobe)
