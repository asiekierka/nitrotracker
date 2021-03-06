#---------------------------------------------------------------------------------
.SUFFIXES:
#---------------------------------------------------------------------------------
ifeq ($(strip $(DEVKITARM)),)
$(error "Please set DEVKITARM in your environment. export DEVKITARM=<path to>devkitARM")
endif

ifeq ($(strip $(LIBDSMI)),)
$(error "Please set LIBDSMI in your environment. export LIBDSMI=<path to>libdsmi")
endif

ifeq ($(strip $(LIBNTXM)),)
$(error "Please set LIBNTXM in your environment. export LIBNTXM=<path to>libntxm")
endif

include $(DEVKITARM)/ds_rules

#---------------------------------------------------------------------------------
# BUILD is the directory where object files & intermediate files will be placed
# SOURCES is a list of directories containing source code
# INCLUDES is a list of directories containing extra header files
# DATA is a list of directories containing binary files
# all directories are relative to this makefile
#---------------------------------------------------------------------------------
BUILD		:=	build
ifeq ($(DEBUG),true)
BUILD		:=	build-debug
endif
SOURCES		:=	source source/tobkit
INCLUDES	:=	include
DATA		:=	data
FONTS		:=	fonts
BITMAPS		:=	bitmaps

#---------------------------------------------------------------------------------
# options for code generation
#---------------------------------------------------------------------------------
#
ARCH	:=	-mthumb -mthumb-interwork
#ARCH	:=	-mthumb-interwork

CFLAGS	:=	-g -Wall -O2\
 			-march=armv5te -mtune=arm946e-s -fomit-frame-pointer\
			-ffast-math \
			$(ARCH)

CFLAGS	+=	$(INCLUDE) -DARM9
ifeq ($(DEBUG),true)
CFLAGS	+=	-DDEBUG
LIBS	:= -ltobkitd -lntxm9d -ldsmi -ldswifi9 -lfat -lnds9d
else
CFLAGS	+=	-DNDEBUG
LIBS	:= -ltobkit -lntxm9 -ldsmi -ldswifi9 -lfat -lnds9
endif

#CXXFLAGS	:=	$(CFLAGS) -fno-rtti -fno-exceptions
CXXFLAGS	:=	$(CFLAGS) -fno-rtti

ASFLAGS	:=	-g $(ARCH) -march=armv5te -mtune=arm946e-s

LDFLAGS	=	-specs=ds_arm9.specs -g $(ARCH) -Wl,-Map,$(notdir $*.map)

#---------------------------------------------------------------------------------
# list of directories containing libraries, this must be the top level containing
# include and lib
#---------------------------------------------------------------------------------
LIBDIRS	:= $(LIBDSMI) $(LIBNTXM) $(TOPDIR)/tobkit $(LIBNDS)

#---------------------------------------------------------------------------------
# no real need to edit anything past this point unless you need to add additional
# rules for different file extensions
#---------------------------------------------------------------------------------
ifneq ($(BUILD),$(notdir $(CURDIR)))
#---------------------------------------------------------------------------------
 
ifeq ($(DEBUG),true)
export ARM9ELF	:=	$(CURDIR)/$(TARGET).arm9.debug.elf
else
export ARM9ELF	:=	$(CURDIR)/$(TARGET).arm9.elf
endif
export DEPSDIR := $(CURDIR)/$(BUILD)

export TOOLSDIR	:=	$(CURDIR)/../tools

export VPATH	:=	$(foreach dir,$(SOURCES),$(CURDIR)/$(dir)) \
			$(foreach dir,$(DATA),$(CURDIR)/$(dir)) \
			$(foreach dir,$(FONTS),$(CURDIR)/$(dir)) \
			$(foreach dir,$(BITMAPS),$(CURDIR)/$(dir))
 
CFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.c)))
CPPFILES	:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.cpp)))
SFILES		:=	$(foreach dir,$(SOURCES),$(notdir $(wildcard $(dir)/*.s)))
BINFILES	:=	$(foreach dir,$(DATA),$(notdir $(wildcard $(dir)/*.*)))
IMGFILES	:=	$(foreach dir,$(FONTS),$(notdir $(wildcard $(dir)/*.*))) \
			$(foreach dir,$(BITMAPS),$(notdir $(wildcard $(dir)/*.*)))

#---------------------------------------------------------------------------------
# use CXX for linking C++ projects, CC for standard C
#---------------------------------------------------------------------------------
ifeq ($(strip $(CPPFILES)),)
#---------------------------------------------------------------------------------
	export LD	:=	$(CC)
#---------------------------------------------------------------------------------
else
#---------------------------------------------------------------------------------
	export LD	:=	$(CXX)
#---------------------------------------------------------------------------------
endif
#---------------------------------------------------------------------------------

export OFILES	:=	$(addsuffix .o,$(BINFILES)) $(IMGFILES:.png=.raw.o) \
					$(CPPFILES:.cpp=.o) $(CFILES:.c=.o) $(SFILES:.s=.o)
 
export INCLUDE	:=	$(foreach dir,$(INCLUDES),-I$(CURDIR)/$(dir)) \
			$(foreach dir,$(LIBDIRS),-I$(dir)/include) \
			-I$(CURDIR)/$(BUILD)
 
export LIBPATHS	:=	$(foreach dir,$(LIBDIRS),-L$(dir)/lib)
 
.PHONY: $(BUILD) clean
 
#---------------------------------------------------------------------------------
$(BUILD):
	@[ -d $@ ] || mkdir -p $@
	@$(MAKE) --no-print-directory -C $(BUILD) -f $(CURDIR)/Makefile
 
#---------------------------------------------------------------------------------
clean:
	@echo clean ...
	@rm -fr build build-debug *.elf *.nds* *.bin 
 
 
#---------------------------------------------------------------------------------
else
 
DEPENDS	:=	$(OFILES:.o=.d)
 
#---------------------------------------------------------------------------------
# main targets
#---------------------------------------------------------------------------------
$(ARM9ELF)	:	$(OFILES) $(TOPDIR)/tobkit/lib/libtobkit.a
	@echo linking $(notdir $@)
	$(LD)  $(LDFLAGS) $(OFILES) $(LIBPATHS) $(LIBS) -o $@

#---------------------------------------------------------------------------------
# you need a rule like this for each extension you use as binary data 
#---------------------------------------------------------------------------------
%.bin.o	:	%.bin
#---------------------------------------------------------------------------------
	@echo $(notdir $<)
	@$(bin2o)

%.raw.o :       %.raw
	@echo $(notdir $<)
	@$(bin2o)

font_3x5.raw :		font_3x5.png
	@echo $(notdir $<)
	@python3 $(TOOLSDIR)/font_3x5_pack.py $< $@

%.raw :		%.png
	@echo $(notdir $<)
	@python3 $(TOOLSDIR)/icon_pack.py $< $@

-include $(DEPENDS)
 
#---------------------------------------------------------------------------------------
endif
#---------------------------------------------------------------------------------------


