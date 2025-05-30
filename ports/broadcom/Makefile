# This file is part of the CircuitPython project: https://circuitpython.org
#
# SPDX-FileCopyrightText: Copyright (c) 2020 Scott Shawcroft for Adafruit Industries
#
# SPDX-License-Identifier: MIT

include ../../py/circuitpy_mkenv.mk

ifeq ($(CHIP_VARIANT), "bcm2711")
CFLAGS += -mcpu=cortex-a72 -DBCM_VERSION=2711
CROSS_COMPILE = aarch64-none-elf-
SUFFIX = 8
else ifeq ($(CHIP_VARIANT), "bcm2837")
CFLAGS += -mcpu=cortex-a53 -DBCM_VERSION=2837
CROSS_COMPILE = aarch64-none-elf-
SUFFIX = 8
else ifeq ($(CHIP_VARIANT), "bcm2835")
CFLAGS += -mcpu=arm1176jzf-s -DBCM_VERSION=2835
CROSS_COMPILE = arm-none-eabi-
SUFFIX =
# TODO add 32-bit support for Cortex-A7 in 2836
endif

INC += -I. \
	   -I../.. \
	   -I../../lib/mp-readline \
	   -I../../lib/timeutils \
	   -I../../lib/sdmmc/include \
	   -Iboards/$(BOARD) \
	   -Iboards/ \
	   -Iperipherals/ \
       -I../../lib/tinyusb/src \
       -I../../supervisor/shared/usb \
	   -I$(BUILD)


SRC_C += bindings/videocore/__init__.c \
         bindings/videocore/Framebuffer.c \
		 boards/$(BOARD)/board.c \
	     boards/$(BOARD)/pins.c \
	     background.c \
	     common-hal/videocore/Framebuffer.c \
		 mphalport.c \
		 lib/sdmmc/sdmmc_cmd.c \
		 lib/sdmmc/sdmmc_common.c \
		 lib/sdmmc/sdmmc_init.c \
		 lib/sdmmc/sdmmc_io.c \
		 lib/sdmmc/sdmmc_mmc.c \
		 lib/sdmmc/sdmmc_sd.c \
		 lib/tinyusb/src/portable/synopsys/dwc2/dcd_dwc2.c \
		 lib/tinyusb/src/portable/synopsys/dwc2/dwc2_common.c \
		 peripherals/broadcom/caches.c \
		 peripherals/broadcom/gen/interrupt_handlers.c \
		 peripherals/broadcom/gen/pins.c \
		 peripherals/broadcom/gpio.c \
		 peripherals/broadcom/interrupts.c \
		 peripherals/broadcom/mmu.c \
		 peripherals/broadcom/vcmailbox.c

SRC_S = peripherals/broadcom/boot$(SUFFIX).s

OBJ = $(PY_O) $(SUPERVISOR_O) $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_COMMON_HAL_SHARED_MODULE_EXPANDED:.c=.o))
	ifeq ($(INTERNAL_LIBM),1)
OBJ += $(addprefix $(BUILD)/, $(SRC_LIBM:.c=.o))
endif
OBJ += $(addprefix $(BUILD)/, $(SRC_CIRCUITPY_COMMON:.c=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_S:.s=.o))
OBJ += $(addprefix $(BUILD)/, $(SRC_MOD:.c=.o))

# BCM CLFAGS
CFLAGS += -nostartfiles -DMICROPY_HW_MCU_NAME="\"$(CHIP_VARIANT)\""


OPTIMIZATION_FLAGS ?= -O3
CFLAGS += $(OPTIMIZATION_FLAGS)

# TinyUSB defines
CFLAGS += -DCFG_TUSB_MCU=OPT_MCU_BCM2711 -DCFG_TUD_MIDI_RX_BUFSIZE=512 \
          -DCFG_TUD_CDC_RX_BUFSIZE=640 -DCFG_TUD_MIDI_TX_BUFSIZE=512 \
          -DCFG_TUD_CDC_TX_BUFSIZE=512 -DCFG_TUD_MSC_BUFSIZE=1024

#Debugging/Optimization
ifeq ($(DEBUG), 1)
	CFLAGS += -ggdb3 -Og
	# No LTO because we may place some functions in RAM instead of flash.
else
	CFLAGS += -DNDEBUG -ggdb3

	# No LTO because we may place some functions in RAM instead of flash.

	ifdef CFLAGS_BOARD
		CFLAGS += $(CFLAGS_BOARD)
	endif
endif


CFLAGS += $(INC) -Wall -Werror -std=gnu11 $(BASE_CFLAGS) $(CFLAGS_MOD) $(COPT) $(DISABLE_WARNINGS) -Werror=old-style-definition

$(BUILD)/lib/tlsf/tlsf.o: CFLAGS += -Wno-cast-align

SRC_QSTR += $(SRC_C) $(SRC_SUPERVISOR) $(SRC_COMMON_HAL_SHARED_MODULE_EXPANDED) $(SRC_CIRCUITPY_COMMON)

LDFLAGS += $(CFLAGS) -T peripherals/broadcom/link$(SUFFIX).ld -Wl,--gc-sections -Wl,-Map=$@.map # -Wl,--cref

# Use toolchain libm if we're not using our own.
ifndef INTERNAL_LIBM
LIBS += -lm
endif

all: $(BUILD)/firmware.kernel$(SUFFIX).img $(BUILD)/firmware.disk.img.zip

$(BUILD)/lib/tlsf/tlsf.o: CFLAGS += -Wno-pointer-to-int-cast -Wno-int-to-pointer-cast

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

ifeq ($(VALID_BOARD),)
$(BUILD)/kernel$(SUFFIX).elf: invalid-board
else
$(BUILD)/kernel$(SUFFIX).elf: $(OBJ)
	$(STEPECHO) "LINK $@"
	$(Q)echo $(OBJ) > $(BUILD)/firmware.objs
	$(Q)$(CC) -o $@ $(LDFLAGS) @$(BUILD)/firmware.objs -Wl,--print-memory-usage -Wl,--start-group $(LIBS) -Wl,--end-group
endif

$(BUILD)/kernel$(SUFFIX).img: $(BUILD)/kernel$(SUFFIX).elf
	$(STEPECHO) "Create $@"
	$(OBJCOPY) -O binary $(BUILD)/kernel$(SUFFIX).elf $@

$(BUILD)/firmware.kernel$(SUFFIX).img: $(BUILD)/kernel$(SUFFIX).img
	$(STEPECHO) "Create $@"
	$(CP) $^ $@

$(BUILD)/firmware.disk.img.zip: $(BUILD)/kernel$(SUFFIX).img
	$(STEPECHO) "Create $@"
	$(Q)dd if=/dev/zero of=$(BUILD)/circuitpython-disk.img bs=1 count=0 seek=256M
	$(Q)parted -s $(BUILD)/circuitpython-disk.img mktable msdos
	$(Q)parted -s $(BUILD)/circuitpython-disk.img mkpart primary fat32 0% 100%
	$(Q)mkfs.fat -F 32 -n BOOT --offset=2048 $(BUILD)/circuitpython-disk.img

	$(Q)mcopy -i $(BUILD)/circuitpython-disk.img@@1M config.txt firmware/bootcode.bin firmware/fixup* firmware/start* firmware/*.dtb ::
	$(Q)mcopy -i $(BUILD)/circuitpython-disk.img@@1M $(BUILD)/kernel$(SUFFIX).img ::
	$(Q)zip $@ $(BUILD)/circuitpython-disk.img
	$(Q)rm $(BUILD)/circuitpython-disk.img

.PHONY: $(BUILD)/rpiboot rpiboot
rpiboot: $(BUILD)/rpiboot
$(BUILD)/rpiboot: $(BUILD)/kernel$(SUFFIX).img
	mkdir -vp $@
	cp -vf $(BUILD)/kernel$(SUFFIX).img $@/
	cp -vfr config.txt firmware/bootcode.bin firmware/fixup* firmware/start* firmware/*.dtb $@/
	#sed -i -e "s/BOOT_UART=0/BOOT_UART=1/" $@/bootcode.bin $@/bootcode.bin
	echo uart_2ndstage=1 >> $@/config.txt

include $(TOP)/py/mkrules.mk
