
###########################################
#  project: STM32F103RB FreeRTOS Demo 01
#     file: Makefile
###########################################

PROJECT = STM32_FreeRTOS_Template

CROSS_COMPILE ?= arm-none-eabi-

#--- sub-directories ---#
SRC_DIR = ./src
USER_DIR = $(SRC_DIR)/User
STLIB_DIR = $(SRC_DIR)/Libraries
FREERTOS_DIR = $(SRC_DIR)/FreeRTOS
XUART_DIR = $(SRC_DIR)/xuart
OUTPUT_DIR = ./out

#--- user sources ---#
USER_SOURCES = main.c system_stm32f10x.c syscalls.c
USER_SRCS = $(USER_SOURCES:%.c=$(USER_DIR)/%.c) #add user *.c files manually
#USER_SRCS = $(wildcard $(USER_DIR)/*.c) #or, compile all *.c in user dir

#--- FreeRTOS sources ---#
FREERTOS_SRCS  = $(FREERTOS_DIR)/list.c
FREERTOS_SRCS += $(FREERTOS_DIR)/queue.c
FREERTOS_SRCS += $(FREERTOS_DIR)/tasks.c
FREERTOS_SRCS += $(FREERTOS_DIR)/portable/GCC/ARM_CM3/port.c
FREERTOS_SRCS += $(FREERTOS_DIR)/portable/MemMang/heap_2.c
#FREERTOS_SRCS += $(FREERTOS_DIR)/croutine.c
#FREERTOS_SRCS += $(FREERTOS_DIR)/timers.c

#--- xUART sources ---#
XUART_SOURCES = uart.c xprintf.c
XUART_SRCS = $(XUART_SOURCES:%.c=$(XUART_DIR)/%.c)

#--- startup code & linker script ---#
LKR_SCRIPT = ./src/stm32_flash_md.ld
STARTUP = ./src/startup_stm32f10x_md.s

DEVICE = STM32F10X_MD #e.g. medium density

PERIPHERALS = #uncomment below the library/peripheral(s) to be used
# PERIPHERALS += stm32f10x_adc
# PERIPHERALS += stm32f10x_bkp
# PERIPHERALS += stm32f10x_can
# PERIPHERALS += stm32f10x_cec
# PERIPHERALS += stm32f10x_crc
# PERIPHERALS += stm32f10x_dac
# PERIPHERALS += stm32f10x_dbgmcu
# PERIPHERALS += stm32f10x_dma
# PERIPHERALS += stm32f10x_exti
# PERIPHERALS += stm32f10x_flash
# PERIPHERALS += stm32f10x_fsmc
 PERIPHERALS += stm32f10x_gpio
# PERIPHERALS += stm32f10x_i2c
# PERIPHERALS += stm32f10x_iwdg
# PERIPHERALS += stm32f10x_pwr
 PERIPHERALS += stm32f10x_rcc
# PERIPHERALS += stm32f10x_rtc
# PERIPHERALS += stm32f10x_sdio
# PERIPHERALS += stm32f10x_spi
# PERIPHERALS += stm32f10x_tim
 PERIPHERALS += stm32f10x_usart
# PERIPHERALS += stm32f10x_wwdg
 PERIPHERALS += misc

PERIPH_DIR = $(STLIB_DIR)/STM32F10x_StdPeriph_Driver
CM3_DIR = $(STLIB_DIR)/CMSIS/CM3
CM3_CORE_DIR = $(CM3_DIR)/CoreSupport
CM3_DEVICE_DIR = $(CM3_DIR)/DeviceSupport/ST/STM32F10x

STARTUP_OBJ = $(STARTUP:%.s=%.o)
USER_OBJS = $(USER_SRCS:%.c=%.o)
FREERTOS_OBJS = $(FREERTOS_SRCS:%.c=%.o)
XUART_OBJS = $(XUART_SRCS:%.c=%.o)

CM3_SRC = $(CM3_CORE_DIR)/core_cm3.c
CM3_OBJ = $(CM3_SRC:%.c=%.o)

PERIPH_SRCS = $(PERIPHERALS:%=$(PERIPH_DIR)/src/%.c)
PERIPH_OBJS = $(PERIPH_SRCS:%.c=%.o)

#--- ToolChain section ---#
CC = $(CROSS_COMPILE)gcc
AS = $(CROSS_COMPILE)as
#LD = $(CROSS_COMPILE)ld
LD = $(CROSS_COMPILE)gcc # linker tool workaround
CP = $(CROSS_COMPILE)objcopy
OD = $(CROSS_COMPILE)objdump
SZ = $(CROSS_COMPILE)size
NM = $(CROSS_COMPILE)nm

OPT = s #optimize: 0, 1, 2, 3, or s

INCLUDES = -I. -I$(USER_DIR) -I$(PERIPH_DIR)/inc \
		-I$(CM3_CORE_DIR) -I$(CM3_DEVICE_DIR) \
		-I$(FREERTOS_DIR) -I$(FREERTOS_DIR)/include -I$(FREERTOS_DIR)/portable/GCC/ARM_CM3 \
		-I$(XUART_DIR)

#--- Toolchain flags ---#
COMMON_FLAGS = -mcpu=cortex-m3 -mthumb# -march=armv7-m
CFLAGS  = $(COMMON_FLAGS) -O$(OPT) -c -g -fno-common -DUSE_STDPERIPH_DRIVER -D$(DEVICE) $(INCLUDES)
CFLAGS += -nostdlib -ffunction-sections -fdata-sections -Wl,--gc-sections
AFLAGS  = $(COMMON_FLAGS)
LFLAGS  = $(COMMON_FLAGS) -nostartfiles -T$(LKR_SCRIPT)
CPFLAGS = -Obinary
ODFLAGS = -S
NMFLAGS = -n

#--- toolchain outputs ---#
OBJECTS  = $(STARTUP_OBJ) $(CM3_OBJ) $(PERIPH_OBJS)
OBJECTS += $(USER_OBJS) $(FREERTOS_OBJS) $(XUART_OBJS)
ELF_FILE = $(OUTPUT_DIR)/$(PROJECT).elf
BIN_FILE = $(OUTPUT_DIR)/$(PROJECT).bin
LST_FILE = $(OUTPUT_DIR)/$(PROJECT).lst
MAP_FILE = $(OUTPUT_DIR)/$(PROJECT).map

#--- remove files ---#
RM = rm -rf 
#RM = cs-rm -f #Codesourcery for Windows

#--- serial bootloader ---#
SERIALPORT = /dev/ttyUSB0
#SERIALPORT = COM3 #COMx if Windows
FLASHLOADER_CMD = python stm32loader.py -p $(SERIALPORT) -e -w -v $(BIN_FILE)

#----- rules for make -----#
all: $(BIN_FILE)
	@echo optimize level = $(OPT) with code size:
	$(SZ) $(ELF_FILE)
	@echo build finished.

.c.o:
	@echo compiling: $(^F)
	$(CC) $(CFLAGS) $< -o $@

.s.o:
	@echo assembling: $(^F)
	$(AS) $(AFLAGS) $< -o $@

$(ELF_FILE): $(OBJECTS)
	@echo linking objects...
	$(LD) $(LFLAGS) $^ -o $@

$(BIN_FILE): $(ELF_FILE)
	@echo creating binary image: $@
	$(CP) $(CPFLAGS) $< $@
	@echo creating code listing: $(LST_FILE)
	$(OD) $(ODFLAGS) $< > $(LST_FILE)
	@echo creating system mapping: $(MAP_FILE)
	$(NM) $(NMFLAGS) $< > $(MAP_FILE)

clean:
	@echo removing temporary files...
	$(RM) $(OBJECTS) $(ELF_FILE) $(BIN_FILE) $(LST_FILE) $(MAP_FILE)

program:
	@echo flashing device...
	$(FLASHLOADER_CMD)



