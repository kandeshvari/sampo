cmake_minimum_required(VERSION 3.16)
project(sampo)

set(CMAKE_C_STANDARD 11)
set(CMAKE_VERBOSE_MAKEFILE 1)

enable_language(ASM)

set(COMPILER_PREFIX /opt/toolchain/gcc-arm-none-eabi-9-2020-q2-update/bin/arm-none-eabi)

SET(CMAKE_ASM_COMPILER ${COMPILER_PREFIX}-gcc)
SET(CMAKE_C_COMPILER "${COMPILER_PREFIX}-gcc")
SET(CMAKE_OBJCOPY "${COMPILER_PREFIX}-objcopy")
SET(CMAKE_SIZE "${COMPILER_PREFIX}-size")
SET(CMAKE_AR "${COMPILER_PREFIX}-gcc-ar")
SET(CMAKE_LINKER "${COMPILER_PREFIX}-gcc-ld")
SET(CMAKE_NM "${COMPILER_PREFIX}-gcc-nm")
SET(CMAKE_OBJDUMP "${COMPILER_PREFIX}-gcc-objdump")
SET(CMAKE_RANLIB "${COMPILER_PREFIX}-gcc-ranlib")


#add_subdirectory(freedom-metal)

SET(LINKER_SCRIPT ${CMAKE_CURRENT_SOURCE_DIR}/platform/nrf52840/config/FreeRTOS_gcc_nrf52.ld)

add_executable(sampo
        modules/nrfx/mdk/gcc_startup_nrf52840.S
        components/libraries/log/src/nrf_log_frontend.c
        components/libraries/log/src/nrf_log_str_formatter.c
        components/boards/boards.c
        freertos/source/croutine.c
        freertos/source/event_groups.c
        freertos/source/portable/MemMang/heap_1.c
        freertos/source/list.c
        freertos/portable/GCC/nrf52/port.c
        freertos/portable/CMSIS/nrf52/port_cmsis.c
        freertos/portable/CMSIS/nrf52/port_cmsis_systick.c
        freertos/source/queue.c
        freertos/source/stream_buffer.c
        freertos/source/tasks.c
        freertos/source/timers.c
        components/libraries/button/app_button.c
        components/libraries/util/app_error.c
        components/libraries/util/app_error_handler_gcc.c
        components/libraries/util/app_error_weak.c
        components/libraries/timer/app_timer_freertos.c
        components/libraries/util/app_util_platform.c
        components/libraries/util/nrf_assert.c
        components/libraries/atomic/nrf_atomic.c
        components/libraries/balloc/nrf_balloc.c
        external/fprintf/nrf_fprintf.c
        external/fprintf/nrf_fprintf_format.c
        components/libraries/memobj/nrf_memobj.c
        components/libraries/ringbuf/nrf_ringbuf.c
        components/libraries/strerror/nrf_strerror.c
        integration/nrfx/legacy/nrf_drv_clock.c
        components/drivers_nrf/nrf_soc_nosd/nrf_nvic.c
        components/drivers_nrf/nrf_soc_nosd/nrf_soc.c
        modules/nrfx/soc/nrfx_atomic.c
        modules/nrfx/drivers/src/nrfx_clock.c
        modules/nrfx/drivers/src/nrfx_gpiote.c
        components/libraries/bsp/bsp.c
        modules/nrfx/mdk/system_nrf52840.c
        src/main.c
        )

target_include_directories(sampo
        PUBLIC
        platform/nrf52840/config
        components
        modules/nrfx/mdk
        components/libraries/timer
        components/libraries/strerror
        components/toolchain/cmsis/include
        freertos/source/include
        freertos/config
        components/libraries/util
        components/libraries/balloc
        components/libraries/ringbuf
        modules/nrfx/hal
        components/libraries/bsp
        components/libraries/log
        components/libraries/button
        modules/nrfx
        components/libraries/experimental_section_vars
        integration/nrfx/legacy
        freertos/portable/CMSIS/nrf52
        integration/nrfx
        components/drivers_nrf/nrf_soc_nosd
        components/libraries/atomic
        components/boards
        components/libraries/memobj
        freertos/portable/GCC/nrf52
        modules/nrfx/drivers/include
        external/fprintf
        components/libraries/log/src

        )

target_compile_options(sampo
        PUBLIC
        -O3 -g3
        -DBOARD_PCA10056
        -DCONFIG_GPIO_AS_PINRESET
        -DFLOAT_ABI_HARD
        -DFREERTOS
        -DNRF52840_XXAA
        -mcpu=cortex-m4
        -mthumb -mabi=aapcs
        -Wall -Werror
        -mfloat-abi=hard -mfpu=fpv4-sp-d16
        -ffunction-sections -fdata-sections -fno-strict-aliasing
        -fno-builtin -fshort-enums
        -D__HEAP_SIZE=8192
        -D__STACK_SIZE=8192

        )

target_link_options(sampo
        PUBLIC
        -mthumb -mabi=aapcs -L${CMAKE_CURRENT_SOURCE_DIR}/modules/nrfx/mdk -T${LINKER_SCRIPT}
        -mcpu=cortex-m4
        -mfloat-abi=hard -mfpu=fpv4-sp-d16
        -Wl,--gc-sections
        --specs=nano.specs
        -lc -lnosys -lm

        )

target_link_libraries(sampo
        PRIVATE

        )

add_custom_command(TARGET sampo POST_BUILD
        COMMAND ${COMPILER_PREFIX}-size
        ARGS ${CMAKE_BINARY_DIR}/sampo
        COMMENT "Show ELF size"
        VERBATIM
        )

add_custom_command(TARGET sampo POST_BUILD
        COMMAND ${CMAKE_OBJCOPY}
        ARGS -O ihex ${CMAKE_BINARY_DIR}/sampo ${CMAKE_BINARY_DIR}/sampo.hex
        COMMENT "Generating HEX image: ${CMAKE_BINARY_DIR}/sampo.hex"
        VERBATIM
        )


add_custom_target(flash)

add_custom_command(TARGET flash POST_BUILD
        COMMAND nrfjprog
        ARGS -f nrf52 --program ${CMAKE_BINARY_DIR}/sampo.hex --sectorerase
        COMMENT "Flashing HEX image: ${CMAKE_BINARY_DIR}/sampo.hex"
        VERBATIM
        )

add_custom_command(TARGET flash POST_BUILD
        COMMAND nrfjprog
        ARGS -f nrf52 --reset
        COMMENT "Resetting device..."
        VERBATIM
        )



