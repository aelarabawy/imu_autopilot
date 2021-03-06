TARGET = sca3000_testing
OBJDIR = bin
Q = @
TOOLS = ../../../../../tools
 
SRCARM = \
    $(TARGET).c \
    arm7/armVIC.c \
    arm7/led.c \
    arm7/sys_time.c \
    arm7/uart.c \
    hal/downlink.c \
    testing/comm_temp.c \
	sca3000.c \
 
ASRCARM = \
    arm7/include/crt0.S
 
MCU = arm7tdmi
THUMB = -mthumb
THUMB_IW = -mthumb-interwork
FORMAT = ihex
CSTANDARD = -std=gnu99 # c99 might also be an option
 
# Define programs and commands.
CC     = arm-elf-gcc
LD     = arm-elf-gcc
SHELL = sh
OBJCOPY = arm-elf-objcopy
OBJDUMP = arm-elf-objdump
SIZE = arm-elf-size
NM = arm-elf-nm
REMOVE = rm -f
COPY = cp
 
# Compiler flags.
CFLAGS += -I arm7
CFLAGS += -I arm7/include
CFLAGS += -I conf
CFLAGS += -I hal
CFLAGS += -I testing
	
CFLAGS += -Os
# CFLAGS += -malignment-traps
CFLAGS += -Winline
CFLAGS += -Wall -Wcast-qual -Wimplicit -Wcast-align
CFLAGS += -Wpointer-arith -Wswitch
CFLAGS += -Wredundant-decls -Wreturn-type -Wshadow -Wunused
CFLAGS += -Wa,-adhlns=$(OBJDIR)/$(notdir $(subst $(suffix $<),.lst,$<))
CFLAGS += -Wstrict-prototypes -Wmissing-declarations
CFLAGS += -Wmissing-prototypes -Wnested-externs
CFLAGS += -std=gnu99
 
#CFLAG Hacks needed while using autogenerated messages.h
CFLAGS += -DDOWNLINK
 
# Linker flags.
LDFLAGS = -n -nostartfiles -Wl,-Map=$(OBJDIR)/$(TARGET).map,--cref
LDFLAGS += -lc
LDFLAGS += -lm
LDFLAGS += -lc -lgcc
LDSCRIPT = arm7/include/LPC2148-ROM-bl.ld
LDFLAGS +=-T$(LDSCRIPT)
 
# Define all object files.
COBJARM = $(SRCARM:%.c=$(OBJDIR)/%.o)
AOBJARM = $(ASRCARM:%.S=$(OBJDIR)/%.o)
 
# Define all list files
LST = $(SRCARM:%.c=$(OBJDIR)/%.lst)
LST += $(ASRCARM:%.S=$(OBJDIR)/%.lst)
 
# Combine all necessary flags and optional flags.
# Add target processor to flags.
ALL_CFLAGS = -mcpu=$(MCU) $(THUMB_IW) -I. $(CFLAGS) $(GENDEPFLAGS)
ALL_ASFLAGS = -mcpu=$(MCU) $(THUMB_IW) -I. -x assembler-with-cpp $(ASFLAGS)
 
# Default target.
all: build size
 
#build: generated elf hex lss sym TODO: For what we need lss and sym?
build: elf hex

 
elf: $(OBJDIR)/$(TARGET).elf
hex: $(OBJDIR)/$(TARGET).hex
lss: $(OBJDIR)/$(TARGET).lss
sym: $(OBJDIR)/$(TARGET).sym
 
# Display size of file.
ELFSIZE = $(SIZE) -A $(OBJDIR)/$(TARGET).elf
size:
	@if [ -f $(OBJDIR)/$(TARGET).elf ]; then echo; $(ELFSIZE); echo; fi
 
# Program the device.
LPC21IAP = $(TOOLS)/lpc21iap/lpc21iap
LPC21IAP-WIN32 = $(TOOLS)/lpc21iap-win32/lpc21iap.exe
bootloader:
  @cd lpc21iap && make --quiet --no-print-directory Q=$(Q)
upload: $(OBJDIR)/$(TARGET).elf bootloader
	@echo ***UPLOAD $(OBJDIR)/$(TARGET).elf
	$(Q)$(LPC21IAP) $(OBJDIR)/$(TARGET).elf
	
upload-win: $(OBJDIR)/$(TARGET).elf bootloader
	@echo ***UPLOAD $(OBJDIR)/$(TARGET).elf
	$(LPC21IAP-WIN32) $(OBJDIR)/$(TARGET).elf
 
# All generated headers
#generated: generated/radio.h generated/build.h
 
# Radio header
#generated/radio.h: conf-$(CONFIG)/radio.xml
#	@echo GENERATE $@
#	$(Q)$(TOOLS)/gen-radio.py $^ > $@
 
#generated/build.h: $(SRCARM)
#	@echo GENERATE $@
#	$(Q)$(TOOLS)/gen-build-info.py > $@
 
# Create final output files (.hex, .eep) from ELF output file.
# TODO: handling the .eeprom-section should be redundant
$(OBJDIR)/%.hex: $(OBJDIR)/%.elf
	@echo OBJC $@
	$(Q)$(OBJCOPY) -O $(FORMAT) $< $@
 
# Create extended listing file from ELF output file.
# testing: option -C
$(OBJDIR)/%.lss: $(OBJDIR)/%.elf
	@echo OBJD $@
	$(Q)$(OBJDUMP) -h -S -C $< > $@
 
# Create a symbol table from ELF output file.
$(OBJDIR)/%.sym: $(OBJDIR)/%.elf
	@echo NM $@
	@echo ***no errors
	$(Q)$(NM) -n $< > $@
 
# Link: create ELF output file from object files.
.SECONDARY : $(OBJDIR)/$(TARGET).elf
.PRECIOUS : $(AOBJARM) $(COBJARM)
$(OBJDIR)/$(TARGET).elf: $(AOBJARM) $(COBJARM)
	@echo LD $@
	$(Q)$(CC) $(THUMB) $(ALL_CFLAGS) $(AOBJARM) $(COBJARM) --output $@ $(LDFLAGS)
	@echo ***no errors
# Compile: create object files from C source files. ARM-only
$(OBJDIR)/%.o : %.c
	@echo CC $@
	$(Q)test -d $(dir $@) || mkdir -p $(dir $@)
	$(Q)$(CC) -c $(ALL_CFLAGS) $(CONLYFLAGS) $< -o $@
 
# Assemble: create object files from assembler source files. ARM-only
$(AOBJARM) : $(OBJDIR)/%.o : %.S
	@echo AS $@
	$(Q)test -d $(dir $@) || mkdir -p $(dir $@)
	$(Q)$(CC) -c $(ALL_ASFLAGS) $< -o $@
 
clean:
	$(REMOVE) -r $(OBJDIR)/
	mkdir $(OBJDIR)
 
# Listing of phony targets.
.PHONY : all size build elf hex lss sym clean upload
 
# Dependencies TODO: is this necessary?
#$(OBJDIR)/.depend:
#	@echo DEPEND $@
#	@test -d $(OBJDIR) || mkdir -p $(OBJDIR)
#	$(Q)$(CC) -MM -MG $(CFLAGS) $(SRCARM) | sed 's|\([^\.]*\.o\)|$(OBJDIR)/\1|' > $@
 
ifneq ($(MAKECMDGOALS),clean)
ifneq ($(MAKECMDGOALS),erase)
#TODO: is this necessary?
#-include $(OBJDIR)/.depend
endif
endif
 
