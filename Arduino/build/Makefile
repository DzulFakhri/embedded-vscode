######################################
# target
######################################
TARGET = arduino_blink


######################################
# building variables
######################################
# debug build?
DEBUG = 1
# optimization
OPT = -Os
# CPU Frequency
F_CPU := 8000000UL

#######################################
# paths
#######################################
# Build path
BUILD_DIR = ./
# Project root path
PROJECT_DIR = ../
# Driver path
DRIVER_DIR = $(PROJECT_DIR)/driver

######################################
# source
######################################
# C sources
C_SOURCES =  \
$(PROJECT_DIR)/src/main.cpp \
$(DRIVER_DIR)/arduino/abi.cpp \
$(DRIVER_DIR)/arduino/CDC.cpp \
$(DRIVER_DIR)/arduino/HardwareSerial.cpp \
$(DRIVER_DIR)/arduino/hooks.c \
$(DRIVER_DIR)/arduino/new.c \
$(DRIVER_DIR)/arduino/PluggableUSB.cpp \
$(DRIVER_DIR)/arduino/Print.cpp \
$(DRIVER_DIR)/arduino/Stream.cpp \
$(DRIVER_DIR)/arduino/USBCore.cpp \
$(DRIVER_DIR)/arduino/WInterrupts.c \
$(DRIVER_DIR)/arduino/wiring_analog.c \
$(DRIVER_DIR)/arduino/wiring_digital.c \
$(DRIVER_DIR)/arduino/wiring_shift.c \
$(DRIVER_DIR)/arduino/wiring.c \
$(DRIVER_DIR)/arduino/WMath.cpp \
$(DRIVER_DIR)/arduino/WString.cpp 

# ASM sources
ASM_SOURCES = 
# ASMM sources
ASMM_SOURCES = \
$(DRIVER_DIR)/arduino/wiring_pulse.S



#######################################
# binaries
#######################################
PREFIX = avr-
# The gcc compiler bin path can be either defined in make command via GCC_PATH variable (> make GCC_PATH=xxx)
# either it can be added to the PATH environment variable.
ifdef GCC_PATH
CC = $(GCC_PATH)/$(PREFIX)gcc
CXX = $(GCC_PATH)/$(PREFIX)g++
AS = $(GCC_PATH)/$(PREFIX)gcc -x assembler-with-cpp
CP = $(GCC_PATH)/$(PREFIX)objcopy
else
CC = $(PREFIX)gcc
CXX = $(PREFIX)g++
AS = $(PREFIX)gcc -x assembler-with-cpp
CP = $(PREFIX)objcopy
endif
LD = $(CXX)
HEX = $(CP) -O ihex
BIN = $(CP) -O binary -S
 
#######################################
# CFLAGS
#######################################
# mcu
MCU = -mmcu=atmega328p

# macros for gcc
# AS defines
AS_DEFS = 

# C defines
C_DEFS =  \
-DF_CPU=$(F_CPU)


# AS includes
AS_INCLUDES = 

# C includes
C_INCLUDES =  \
-I $(PROJECT_DIR)/inc \
-I $(DRIVER_DIR)/arduino


# compile gcc flags
ASFLAGS = $(MCU) $(AS_DEFS) $(AS_INCLUDES) $(OPT) -Wall

CFLAGS += $(MCU) $(C_DEFS) $(C_INCLUDES) $(OPT) -Wall

ifeq ($(DEBUG), 1)
CFLAGS += -g -gdwarf-2
endif


# Generate dependency information
CFLAGS += -MMD -MP -MF"$(@:%.o=%.d)"
CXXFLAGS = $(CFLAGS) -fno-exceptions -fno-rtti -fno-threadsafe-statics
LDFLAGS = $(MCU) -Wl,--gc-sections -lm

# default action: build all
all: $(BUILD_DIR)/$(TARGET).elf $(BUILD_DIR)/$(TARGET).hex $(BUILD_DIR)/$(TARGET).bin
	@rm -f $(BUILD_DIR)/*.o $(BUILD_DIR)/*.d $(BUILD_DIR)/*.lst

#######################################
# build the application
#######################################
# list of objects
OBJECTS = $(addprefix $(BUILD_DIR)/,$(patsubst %.c,%.o,$(patsubst %.cpp,%.o,$(notdir $(C_SOURCES)))))
vpath %.c $(sort $(dir $(C_SOURCES)))
vpath %.cpp $(sort $(dir $(C_SOURCES)))
# list of ASM program objects
OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASM_SOURCES:.s=.o)))
vpath %.s $(sort $(dir $(ASM_SOURCES)))
OBJECTS += $(addprefix $(BUILD_DIR)/,$(notdir $(ASMM_SOURCES:.S=.o)))
vpath %.S $(sort $(dir $(ASMM_SOURCES)))

$(BUILD_DIR)/%.o: %.c Makefile | $(BUILD_DIR) 
	$(CC) -c $(CFLAGS) -Wa,-a,-ad,-alms=$(BUILD_DIR)/$(notdir $(<:.c=.lst)) $< -o $@

$(BUILD_DIR)/%.o: %.cpp Makefile | $(BUILD_DIR)
	$(CXX) -c $(CXXFLAGS) -Wa,-a,-ad,-alms=$(BUILD_DIR)/$(notdir $(<:.cpp=.lst)) $< -o $@

$(BUILD_DIR)/%.o: %.s Makefile | $(BUILD_DIR)
	$(AS) -c $(ASFLAGS) $< -o $@
$(BUILD_DIR)/%.o: %.S Makefile | $(BUILD_DIR)
	$(AS) -c $(ASFLAGS) $< -o $@

$(BUILD_DIR)/$(TARGET).elf: $(OBJECTS) Makefile
	$(LD) $(LDFLAGS) $(OBJECTS) -o $@

$(BUILD_DIR)/%.hex: $(BUILD_DIR)/%.elf | $(BUILD_DIR)
	$(HEX) $< $@
	
$(BUILD_DIR)/%.bin: $(BUILD_DIR)/%.elf | $(BUILD_DIR)
	$(BIN) $< $@	

$(BUILD_DIR):
	mkdir $@	

#######################################
# clean up
#######################################
clean:
	rm -f $(BUILD_DIR)/*.elf $(BUILD_DIR)/*.hex $(BUILD_DIR)/*.bin

#######################################
# flash USBASP
#######################################
upload: $(HEX)
	$(AVRDUDE) -c usbasp-clone -p m328p -U flash:w:$(HEX)

#######################################
# dependencies
#######################################
-include $(wildcard $(BUILD_DIR)/*.d)

# *** EOF ***
