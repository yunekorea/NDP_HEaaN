#  SPDX-License-Identifier: BSD-3-Clause
#  Copyright (C) 2016 Intel Corporation.
#  All rights reserved.
#

SPDK_ROOT_DIR := $(abspath $(CURDIR)/../..)
include $(SPDK_ROOT_DIR)/mk/spdk.common.mk
LD = g++
include $(SPDK_ROOT_DIR)/mk/spdk.modules.mk

APP = nvmf_heaan_tgt

C_SRCS := nvmf_heaan_main.c

SPDK_LIB_LIST = $(ALL_MODULES_LIST) event event_nvmf

ifeq ($(SPDK_ROOT_DIR)/lib/env_dpdk,$(CONFIG_ENV))
SPDK_LIB_LIST += env_dpdk_rpc
endif

ifeq ($(OS),Linux)
SPDK_LIB_LIST += event_nbd
endif

# !@Gemini@! --- START HEaaN INTEGRATION MODIFICATIONS ---

# 1. Add the include path for HEaaN headers.
# This ensures files like nvmf_heaan_main.c and ctrlr_bdev.c can find the *.h files.
CFLAGS += -I$(SPDK_ROOT_DIR)/include/heaan
CFLAGS += -DHEAAN_LIB

# 2. Add the path to the HEaaN library directory to the linker.
# The '-L' flag tells the linker where to search for the library file.
# LDLIBS += -L$(SPDK_ROOT_DIR)/lib/heaan
# LDLIBS += $(SPDK_ROOT_DIR)/lib/heaan/libHEAAN.a
LDLIBS += -lc $(SPDK_ROOT_DIR)/lib/heaan/libHEAAN.a -lntl -lgmp -lm  -lstdc++ -lrt -lpthread


# 3. Add the HEaaN library name to the linker flags.
# The '-l' flag tells the linker to link against 'libHEaaN.a'.
#LDLIBS += -lHEAAN

# Includes C++ runtime library
# LDLIBS += -lstdc++ -lpthread -lc -lrt

# !@Gemini@! --- END HEaaN INTEGRATION MODIFICATIONS ---

include $(SPDK_ROOT_DIR)/mk/spdk.app.mk
LIBS += -lc $(SPDK_ROOT_DIR)/lib/heaan/libHEAAN.a -lntl -lgmp -lm  -lstdc++ -lrt -lpthread

install: $(APP)
	$(INSTALL_APP)

uninstall:
	$(UNINSTALL_APP)
