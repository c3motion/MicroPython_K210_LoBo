#
# Makefile for maixpy
# ports/k210-freetos/mpy_support/Makefile
#
.PHONY:all update_mk compile rm_genhdr_qstrdefs_generated_h
CUR_DIR_ADDR := $(shell pwd)/
###############################################################################
# USER OPTIONS
LIB_NAME ?= mpy_support
CROSS_COMPILE ?= 
OUTPUT_DIR := build/
FROZEN_MPY_DIR ?= modules
MK_VALUE :="INC += "$(CUR_DIR_ADDR)"\n"
#MK_VALUE +="INC += "$(CUR_DIR_ADDR)"Maix/include/""\n"
MK_VALUE +="INC += "$(CUR_DIR_ADDR)"standard_lib/include/""\n"
MK_VALUE +="INC += "$(CUR_DIR_ADDR)"../../micropython/""\n"
MK_VALUE +="INC += "$(CUR_DIR_ADDR)$(OUTPUT_DIR)"\n"
MK_VALUE +="liba-mpy += "$(CUR_DIR_ADDR)$(OUTPUT_DIR)"mpy_support.a""\n"
MK_VALUE +="output-file += "'$$(liba-mpy)'"\r\n"

OUTPUT_STATIC_LIB := $(LIB_NAME).a
CXX ?= $(CROSS_COMPILE)c++
AR ?= $(CROSS_COMPILE)AR

###############################################################################
# MPY OPTIONS
# include ../../../py/mkenv.mk需要比较靠前
include ../../micropython/py/mkenv.mk
# qstr definitions (must come before including py.mk)
QSTR_DEFS = qstrdefsport.h
# include py core make definitionsvers/uarths.c
include ../../micropython/py/py.mk
MPY_EXT_SRC_C := \
		lib/utils/stdout_helpers.c \
		lib/utils/interrupt_char.c \
		lib/utils/pyexec.c \
		lib/mp-readline/readline.c \
		lib/netutils/netutils.c \
		lib/timeutils/timeutils.c \
		lib/utils/sys_stdio_mphal.c

###############################################################################
# Optional options
PLATFORM ?=
###############################################################################
# COMPILE OPTIONS
# CFLAGS :=
# LDFLAGS :=
CXXFLAGS :=
INC :=
output-file :=
###############################################################################
# IMPORT MK FILE       s32_t f_res = SPIFFS_fflush(&fs, fd);
# sinclude mk MICROPYTHON NEED OTHER MK,SO I INCLUDE THE MK WHERE IS IN ROOT DIR
sinclude ../mk
###############################################################################
# IMPORT PLATFORM OPTIONS
sinclude $(PLATFORM).mk
###############################################################################
# SOME FUNCTIONS
define sub_clean
	$(MAKE) -C $(1) clean CROSS_COMPILE=$(CROSS_COMPILE) PLATFORM=$(PLATFORM);
endef

define sub_make
	$(info =====[ $(dir $(1)) ]=====)
	$(MAKE) -C $(basename $(patsubst %/,%,$(patsubst ./%,%,$(dir $(1))))) all CROSS_COMPILE=$(CROSS_COMPILE) PLATFORM=$(PLATFORM)
endef

define update_sub_mk
	if [ "$(dir $(1))" != "./" ] ; \
	then \
		$(MAKE) -C $(1) update_mk CROSS_COMPILE=$(CROSS_COMPILE) PLATFORM=$(PLATFORM); \
	fi;
endef
###############################################################################
GET_SUBDIRS1 := $(shell find . -maxdepth 1 -type d)
GET_SUBDIRS2 := $(basename $(patsubst ./%,%,$(GET_SUBDIRS1)))
SUBDIRS := $(GET_SUBDIRS2)

AllDirs := $(shell ls -R | grep '^\./.*:$$' | awk '{gsub(":","");print}') .
FILE_MAKEFILE := $(foreach n,$(SUBDIRS) , $(wildcard $(n)/Makefile))

# $(BUILD)/_frozen_mpy.c: frozentest.mpy $(BUILD)/genhdr/qstrdefs.generated.h
# $(ECHO) "MISC freezing bytecode"
# $(Q)$(TOP)/tools/mpy-tool.py -f -q $(BUILD)/genhdr/qstrdefs.preprocessed.h -mlongint-impl=none $< > $@

all: compile
.SILENT: clean clean-all update_mk compile sub-compile

###############################################################################
# COMPILE CUR SRC FILE
TEMP_DIRS := $(subst build,,$(AllDirs))
SRC_C := $(foreach n,$(TEMP_DIRS) , $(wildcard $(n)/*.c))
SRC_CPP := $(foreach n,$(TEMP_DIRS) , $(wildcard $(n)/*.cpp))
SRC_C += $(MPY_EXT_SRC_C)
########################

C_OBJ := $(addprefix $(BUILD)/, $(SRC_C:.c=.o))
OBJ += $(PY_O) $(C_OBJ)
# USE MPY COMPILE SRICPT COMPILE SRC FILE
CXX_OBJ := $(addprefix $(BUILD)/, $(SRC_CPP:.cpp=.o))
INCLUDE_FLAGS := $(addprefix -I, $(INC))
CFLAGS += $(INCLUDE_FLAGS)

# List of sources for qstr extraction
SRC_QSTR += $(SRC_C)
# Append any auto-generated sources that are needed by sources listed in SRC_QSTR
SRC_QSTR_AUTO_DEPS +=

$(CXX_OBJ):%.o:%.cpp
	@mkdir -p $(dir $(OUTPUT_DIR))
	$(Q)$(CXX)  -o $(OUTPUT_DIR)/$@ -c $< -I$(INC) $(CXXFLAGS) -lstdc++

# $(HEADER_BUILD)/qstr.i.last $(HEADER_BUILD)/qstrdefs.generated.h $(BUILD)/_frozen_mpy.c
compile: sub_compile $(OUTPUT_STATIC_LIB)

$(OUTPUT_STATIC_LIB): $(OBJ) $(CXX_OBJ) 
	@$(AR) rcs -o $(OUTPUT_DIR)$@ $(CXX_OBJ) $(OBJ) $(LIBS) $(LDFLAGS)
###############################################################################

sub_compile:
	$(foreach n,$(FILE_MAKEFILE),$(call sub_make,$(n)))

update_mk:
	$(foreach n,$(FILE_MAKEFILE),$(call update_sub_mk, $(dir $(n))))
	$(info =====[ mpy_support update mk file$(foreach n,$(SUBDIRS) , $(wildcard $(n)/mk)) ... ]=====)
	$(shell echo $(MK_VALUE)$(foreach nn,$(foreach n,$(SUBDIRS) , $(wildcard $(n)/mk)),"-include "$(CUR_DIR_ADDR)$(nn)"\n") > mk)

clean_all: clean
	@rm -f mk
	$(info =====[ Cleaning $(SUBDIRS) ... ]=====)
	$(foreach n,$(FILE_MAKEFILE),$(call sub_clean, $(n)))

include $(TOP)/py/mkrules.mk
