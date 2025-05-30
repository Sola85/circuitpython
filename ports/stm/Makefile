# This file is part of the CircuitPython project: https://circuitpython.org
#
# SPDX-FileCopyrightText: Copyright (c) 2019 Dan Halbert for Adafruit Industries
# SPDX-FileCopyrightText: Copyright (c) 2019 Lucian Copeland for Adafruit Industries
#
# SPDX-License-Identifier: MIT

include ../../py/circuitpy_mkenv.mk

CROSS_COMPILE = arm-none-eabi-

MCU_SERIES_LOWER = $(shell echo $(MCU_SERIES) | tr '[:upper:]' '[:lower:]')
MCU_VARIANT_LOWER = $(shell echo $(MCU_VARIANT) | tr '[:upper:]' '[:lower:]')

HAL_DIR=st_driver/stm32$(MCU_SERIES_LOWER)xx_hal_driver

INC += -I.
INC += -I../..
INC += -I$(BUILD)
INC += -I$(BUILD)/genhdr
INC += -I./$(HAL_DIR)/Inc
INC += -I./$(HAL_DIR)/Inc/Legacy
INC += -I./st_driver/cmsis_device_$(MCU_SERIES_LOWER)/Include
INC += -I./st_driver/CMSIS_5/CMSIS/Core/Include
INC += -I./boards
INC += -I./boards/$(BOARD)
INC += -I./hal_conf
INC += -I./peripherals
INC += -I./packages
INC += -I../../lib/mp-readline
INC += -I../../lib/tinyusb/src
INC += -I../../supervisor/shared/usb

#Debugging/Optimization
ifeq ($(DEBUG), 1)
	CFLAGS += -ggdb3
# You may want to enable these flags to make setting breakpoints easier.
	CFLAGS += -fno-inline -fno-ipa-sra
else
	CFLAGS += -DNDEBUG
	OPTIMIZATION_FLAGS ?= -O2 -fno-inline-functions
	CFLAGS += -ggdb3
endif

# to override compiler optimization level, set in boards/$(BOARD)/mpconfigboard.mk
CFLAGS += $(OPTIMIZATION_FLAGS)

# Add -ftree-vrp optimization and checking to all builds. It's not enabled for -Os by default.
CFLAGS += -ftree-vrp

# STM32 MCU series must be defined. See supervisor/linker.h
C_DEFS = -D$(MCU_PACKAGE) -DUSE_HAL_DRIVER -DUSE_FULL_LL_DRIVER -D$(MCU_VARIANT) -DSTM32$(MCU_SERIES)

CFLAGS += $(INC) -Werror -Wall -std=gnu11 -fshort-enums $(BASE_CFLAGS) $(C_DEFS) $(CFLAGS_MOD) $(COPT) -nostdlib -nostartfiles -Werror=old-style-definition

# Undo some warnings.
# STM32 HAL uses undefined preprocessor variables, shadowed variables, casts that change alignment reqs
# You can add your own temporary suppression by setting ADD_CFLAGS in the make command
CFLAGS += -Wno-undef -Wno-shadow -Wno-cast-align $(ADD_CFLAGS)

CFLAGS += -mthumb -mabi=aapcs-linux

# Arm core selection
MCU_FLAGS_L4 = -mcpu=cortex-m4
MCU_FLAGS_F4 = -mcpu=cortex-m4
MCU_FLAGS_F7 = -mcpu=cortex-m7
MCU_FLAGS_H7 = -mcpu=cortex-m7
CFLAGS += $(MCU_FLAGS_$(MCU_SERIES))

# Select HAL file for distribution via mpconfigport
CFLAGS += -DSTM32_HAL_H="<stm32$(MCU_SERIES_LOWER)xx_hal.h>"

CFLAGS += -DSTM32_SERIES_LOWER='"stm32$(MCU_SERIES_LOWER)"'

# Floating point settings
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32F765xx STM32F767xx STM32F769xx STM32H743xx STM32H750xx))
CFLAGS += -mfpu=fpv5-d16 -mfloat-abi=hard
else
CFLAGS += -mfpu=fpv4-sp-d16 -mfloat-abi=hard
endif

# Need both command and valid file to use uf2 bootloader
ifndef LD_FILE
    ifneq ($(and $(UF2_BOOTLOADER),$(LD_BOOT)),)
        LD_FILE = $(LD_BOOT)
        BOOTLOADER_OFFSET = $(UF2_OFFSET)
        CFLAGS += -DUF2_BOOTLOADER_ENABLED
    else
        LD_FILE = $(LD_DEFAULT)
    endif
endif

# Add bootloader specific items
ifndef BOOTLOADER_OFFSET
	BOOTLOADER_OFFSET := 0x8000000
endif

LDFLAGS = $(CFLAGS) -Wl,-nostdlib -Wl,-T,$(LD_FILE) -Wl,-T,$(LD_COMMON) -Wl,-Map=$@.map -Wl,-cref -Wl,-gc-sections -specs=nano.specs
LIBS := -lgcc -lc

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

# TinyUSB defines
CFLAGS += \
	-DCFG_TUSB_MCU=OPT_MCU_STM32$(MCU_SERIES) \
	-DCFG_TUD_CDC_RX_BUFSIZE=1024 \
	-DCFG_TUD_CDC_TX_BUFSIZE=1024 \
	-DCFG_TUD_MSC_BUFSIZE=4096 \
	-DCFG_TUD_MIDI_RX_BUFSIZE=128 \
	-DCFG_TUD_MIDI_TX_BUFSIZE=128

ifdef CIRCUITPY_USB_VENDOR
CFLAGS += \
	-DCFG_TUD_VENDOR_RX_BUFSIZE=1024 \
	-DCFG_TUD_VENDOR_TX_BUFSIZE=1024
endif


SRC_STM32 = $(addprefix $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_,\
	hal.c \
	hal_adc.c \
	hal_adc_ex.c \
	hal_cortex.c \
	hal_dac.c \
	hal_dac_ex.c \
	hal_dma.c \
	hal_dma_ex.c \
	hal_exti.c \
	hal_flash.c \
	hal_flash_ex.c \
	hal_gpio.c \
	hal_i2c.c \
	hal_i2c_ex.c \
	hal_pwr.c \
	hal_pwr_ex.c \
	hal_qspi.c \
	hal_rcc.c \
	hal_rcc_ex.c \
	hal_rng.c \
	hal_rtc.c \
	hal_rtc_ex.c \
	hal_sd.c \
	hal_spi.c \
	hal_tim.c \
	hal_tim_ex.c \
	hal_uart.c \
	hal_usart.c \
	ll_adc.c \
	ll_dma.c \
	ll_exti.c \
	ll_gpio.c \
	ll_i2c.c \
	ll_rcc.c \
	ll_sdmmc.c \
	ll_usart.c \
	ll_utils.c \
)

ifeq ($(CIRCUITPY_CANIO),1)
SRC_STM32 += $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_can.c
endif

# Need this to avoid UART linker problems. TODO: rewrite to use registered callbacks.
# Does not exist for F4 and lower
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32F765xx STM32F767xx STM32F769xx STM32H743xx STM32H750xx STM32L4R5xx STM32L433xx))
	SRC_STM32 += $(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_uart_ex.c
endif

ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32H750xx))
	C_DEFS += -DHAL_SDRAM_MODULE_ENABLED
	SRC_STM32 += st_driver/stm32$(MCU_SERIES_LOWER)xx_hal_driver/Src/stm32h7xx_hal_sdram.c
	SRC_STM32 += st_driver/stm32$(MCU_SERIES_LOWER)xx_hal_driver/Src/stm32h7xx_ll_fmc.c
	SRC_C += peripherals/sdram.c
endif

SRC_STM32 += boards/system_stm32$(MCU_SERIES_LOWER)xx.c

SRC_C += \
	background.c \
	mphalport.c \
	boards/$(BOARD)/board.c \
	boards/$(BOARD)/pins.c \
	peripherals/timers.c \
	peripherals/exti.c \
	peripherals/rtc.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/clocks.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/pins.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/gpio.c \
	peripherals/stm32$(MCU_SERIES_LOWER)/$(MCU_VARIANT_LOWER)/periph.c \
	packages/$(MCU_PACKAGE).c

ifneq ($(CIRCUITPY_AUDIOBUSIO_PDMIN),0)
	SRC_C += \
		common-hal/audiobusio/MEMS_Audio.c \
		common-hal/audiobusio/MEMS_Audio_ll_stm32l4.c \
		common-hal/audiobusio/OpenPDMFilter.c

	SRC_STM32 += \
		$(HAL_DIR)/Src/stm32$(MCU_SERIES_LOWER)xx_hal_sai.c

endif

ifneq ($(CIRCUITPY_USB),0)
	ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32L433xx))
		SRC_C += lib/tinyusb/src/portable/st/stm32_fsdev/dcd_stm32_fsdev.c
	else
		SRC_C += lib/tinyusb/src/portable/synopsys/dwc2/dcd_dwc2.c
		SRC_C += lib/tinyusb/src/portable/synopsys/dwc2/dwc2_common.c
	endif
endif

SRC_S_UPPER = supervisor/shared/cpu_regs.S
SRC_S = \
	st_driver/cmsis_device_$(MCU_SERIES_LOWER)/Source/Templates/gcc/startup_$(MCU_VARIANT_LOWER).s

ifneq ($(FROZEN_MPY_DIR),)
FROZEN_MPY_PY_FILES := $(shell find -L $(FROZEN_MPY_DIR) -type f -name '*.py')
FROZEN_MPY_MPY_FILES := $(addprefix $(BUILD)/,$(FROZEN_MPY_PY_FILES:.py=.mpy))
endif

OBJ += $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_STM32:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_SHARED_MODULE_EXPANDED:.c=.o))
ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_CIRCUITPY_COMMON:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_S_UPPER:.S=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))

$(BUILD)/$(FATFS_DIR)/ff.o: COPT += -Os
$(filter $(PY_BUILD)/../extmod/vfs_fat_%.o, $(PY_O)): COPT += -Os

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_MOD) $(SRC_COMMON_HAL_SHARED_MODULE_EXPANDED) $(SRC_CIRCUITPY_COMMON)
# Sources that only hold QSTRs after pre-processing.
SRC_QSTR_PREPROCESSOR +=

# Bin section settings specific to the STM32H7
ifeq ($(MCU_VARIANT),$(filter $(MCU_VARIANT),STM32H743xx STM32H750xx))
MCU_SECTIONS = -j .isr_vector -j .text -j .data -j .itcm -j .dtcm_data $^ $@
else
MCU_SECTIONS = $^ $@
endif

all: $(BUILD)/firmware.bin $(BUILD)/firmware.uf2

ifeq ($(VALID_BOARD),)
$(BUILD)/firmware.elf: invalid-board
else
$(BUILD)/firmware.elf: $(OBJ)
	$(STEPECHO) "LINK $@"
	$(Q)echo $^ > $(BUILD)/firmware.objs
	$(Q)$(CC) -o $@ $(LDFLAGS) @$(BUILD)/firmware.objs -Wl,--print-memory-usage -Wl,--start-group $(LIBS) -Wl,--end-group
	$(Q)$(SIZE) $@ | $(PYTHON) $(TOP)/tools/build_memory_info.py $(LD_FILE) $(BUILD)
endif

$(BUILD)/firmware.bin: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O binary $(MCU_SECTIONS)

$(BUILD)/firmware.hex: $(BUILD)/firmware.elf
	$(STEPECHO) "Create $@"
	$(Q)$(OBJCOPY) -O ihex $(MCU_SECTIONS)

$(BUILD)/firmware.uf2: $(BUILD)/firmware.hex
	$(ECHO) "Create $@"
	$(PYTHON) $(TOP)/tools/uf2/utils/uf2conv.py -f $(UF2_FAMILY_ID) -b $(BOOTLOADER_OFFSET) -c -o "$(BUILD)/firmware.uf2" $^

flash: $(BUILD)/firmware.bin
	$(ECHO) "Writing $< to the board"
	dfu-util -a 0 --dfuse-address $(BOOTLOADER_OFFSET) -D $(BUILD)/firmware.bin

include $(TOP)/py/mkrules.mk
