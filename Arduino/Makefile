# Simple Makefile for AVR project

PROGRAM_NAME := blink
PROJ_DIR := ..
LIB_DIR := $(PROJ_DIR)/lib
BUILD_DIR := .

MCU := atmega328p
CC := avr-gcc
CXX := avr-g++
LD := avr-g++
OBJCOPY := avr-objcopy
AVRDUDE := avrdude
F_CPU := 8000000UL

# Compiler and options configuration
CFLAGS := -mmcu=$(MCU) -Wall -Os -DF_CPU=$(F_CPU)
CXXFLAGS := $(CFLAGS)
ASFLAGS := -mmcu=$(MCU) -DF_CPU=$(F_CPU)

# Include Arduino headers from lib
INCFLAGS := -I $(LIB_DIR)/arduino \
			-I $(PROJ_DIR)/inc

# Find sources (search only src and lib to avoid duplicates).
# Prefer .S over .c when both exist in the same basename.
S_SRCS := $(shell find $(LIB_DIR) -type f -name '*.S' 2>/dev/null)
C_SRCS := $(shell find $(LIB_DIR) -type f -name '*.c' 2>/dev/null)
CPP_SRCS := $(shell find $(LIB_DIR) -type f -name '*.cpp' 2>/dev/null)

# remove C sources that have a corresponding .S (prefer assembly)
C_SRCS := $(filter-out $(patsubst %.S,%.c,$(S_SRCS)),$(C_SRCS))

SRCS := $(C_SRCS) $(CPP_SRCS) $(S_SRCS) $(PROJ_DIR)/src/main.cpp

# Object files placed in build dir, include source extension to avoid collisions
OBJS := $(notdir $(SRCS))
OBJS := $(patsubst %.c,$(BUILD_DIR)/%.c.o,$(OBJS))
OBJS := $(patsubst %.cpp,$(BUILD_DIR)/%.cpp.o,$(OBJS))
OBJS := $(patsubst %.s,$(BUILD_DIR)/%.s.o,$(OBJS))
OBJS := $(patsubst %.S,$(BUILD_DIR)/%.S.o,$(OBJS))

ELF := $(BUILD_DIR)/$(PROGRAM_NAME).elf
HEX := $(BUILD_DIR)/$(PROGRAM_NAME).hex

# Search paths for sources so pattern rules find files by basename
VPATH := $(PROJ_DIR)/src:$(PROJ_DIR):$(LIB_DIR):$(LIB_DIR)/arduino

.PHONY: all upload clean

# Default target: build the hex in the build directory
all: $(HEX)

# Ensure build directory exists before compiling objects
$(OBJS): | $(BUILD_DIR)

$(BUILD_DIR):
	mkdir -p $(BUILD_DIR)

# Compile C sources
$(BUILD_DIR)/%.c.o: %.c
	$(CC) $(CFLAGS) $(INCFLAGS) -c -o $@ $<

# Compile C++ sources
$(BUILD_DIR)/%.cpp.o: %.cpp
	$(CXX) $(CXXFLAGS) $(INCFLAGS) -c -o $@ $<

# Assemble assembly sources (both .s and .S)
$(BUILD_DIR)/%.s.o: %.s
	$(CC) $(ASFLAGS) -x assembler-with-cpp -c -o $@ $<

$(BUILD_DIR)/%.S.o: %.S
	$(CC) $(ASFLAGS) -x assembler-with-cpp -c -o $@ $<

# Link objects into ELF, then produce HEX
$(ELF): $(OBJS)
	$(LD) $(CFLAGS) $(INCFLAGS) -o $@ $(OBJS)
	$(OBJCOPY) -j .text -j .data -O ihex $@ $(HEX)

$(HEX): $(ELF)
	@echo "Built $(HEX)"

# Upload the hex to the MCU (uses build dir hex)
upload: $(HEX)
	$(AVRDUDE) -c usbasp-clone -p m328p -U flash:w:$(HEX)

# Clean build artifacts in the build directory
clean:
	rm -f $(BUILD_DIR)/*.elf $(BUILD_DIR)/*.hex $(BUILD_DIR)/*.o
