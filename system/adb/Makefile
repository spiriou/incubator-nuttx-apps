############################################################################
# system/adb/Makefile
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.  The
# ASF licenses this file to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance with the
# License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.  See the
# License for the specific language governing permissions and limitations
# under the License.
#
############################################################################

include $(APPDIR)/Make.defs

WGET ?= wget
UNPACK ?= unzip -q

ADB_DIR := $(APPDIR)/system/adb
CONFIG_ADB_URL ?= "https://github.com/spiriou/microADB/archive"
CONFIG_ADB_VERSION ?= draft
CONFIG_ADB_TARBALL ?= $(CONFIG_ADB_VERSION).zip

ADB_UNPACKNAME := microADB-$(CONFIG_ADB_VERSION)
ADB_UNPACKDIR := $(ADB_DIR)/$(ADB_UNPACKNAME)

$(ADB_DIR)/$(CONFIG_ADB_TARBALL):
	@echo "Downloading: $(CONFIG_ADB_TARBALL)"
	$(Q) $(WGET) $(CONFIG_ADB_URL)/$(CONFIG_ADB_TARBALL) \
		-O $(ADB_DIR)/$(CONFIG_ADB_TARBALL)

$(ADB_UNPACKDIR): $(ADB_DIR)/$(CONFIG_ADB_TARBALL)
	@echo "Unpacking: $(CONFIG_ADB_TARBALL) -> $(ADB_UNPACKDIR)"
	$(call DELDIR, $(ADB_UNPACKDIR))
	$(Q) unzip -q -d "$(ADB_DIR)" "$(ADB_DIR)/$(CONFIG_ADB_TARBALL)"
	$(Q) touch $(ADB_UNPACKDIR)

# adb server app

CONFIG_SYSTEM_ADB_PRIORITY ?= SCHED_PRIORITY_DEFAULT
CONFIG_SYSTEM_ADB_STACKSIZE ?= 2048
CONFIG_SYSTEM_ADB_PROGNAME ?= adbd$(EXEEXT)
PROGNAME = $(CONFIG_SYSTEM_ADB_PROGNAME)

PROGNAME  := $(CONFIG_SYSTEM_ADB_PROGNAME)
PRIORITY  := $(CONFIG_SYSTEM_ADB_PRIORITY)
STACKSIZE := $(CONFIG_SYSTEM_ADB_STACKSIZE)
MODULE    := $(CONFIG_SYSTEM_ADB)

# Files

MAINSRC := $(ADB_UNPACKNAME)/adb_main.c
CSRCS += $(ADB_UNPACKNAME)/adb_client.c
CSRCS += $(ADB_UNPACKNAME)/adb_frame.c

CSRCS += $(ADB_UNPACKNAME)/hal/hal_uv.c
CSRCS += $(ADB_UNPACKNAME)/hal/hal_uv_packet.c
CSRCS += $(ADB_UNPACKNAME)/dumphex.c

CFLAGS += -D__NUTTX__=1
CFLAGS += -I$(ADB_UNPACKNAME)

ifeq ($(CONFIG_SYSTEM_ADB_TCP_SERVER),y)
CSRCS += $(ADB_UNPACKNAME)/hal/hal_uv_client_tcp.c
endif

ifeq ($(CONFIG_SYSTEM_ADB_USB_SERVER),y)
CSRCS += $(ADB_UNPACKNAME)/hal/hal_uv_client_usb.c
endif

ifeq ($(CONFIG_SYSTEM_ADB_FILE_SERVICE),y)
CSRCS += $(ADB_UNPACKNAME)/file_sync_service.c
endif

ifeq ($(CONFIG_SYSTEM_ADB_SOCKET_SERVICE),y)
CSRCS += $(ADB_UNPACKNAME)/tcp_service.c
CSRCS += $(ADB_UNPACKNAME)/hal/hal_uv_socket.c
endif

ifeq ($(CONFIG_SYSTEM_ADB_SHELL_SERVICE),y)
CSRCS += $(ADB_UNPACKNAME)/shell_service.c
CSRCS += $(ADB_UNPACKNAME)/hal/hal_uv_pipe.c
endif

ifeq ($(CONFIG_SYSTEM_ADB_LOGCAT_SERVICE),y)
CSRCS += $(ADB_UNPACKNAME)/logcat_service.c
endif

ifeq ($(CONFIG_SYSTEM_ADB_AUTHENTICATION),y)
CSRCS += $(ADB_UNPACKNAME)/adb_auth_key.c
CSRCS += $(ADB_UNPACKNAME)/mincrypt/rsa.c
CSRCS += $(ADB_UNPACKNAME)/mincrypt/sha.c
CSRCS += $(ADB_UNPACKNAME)/mincrypt/sha256.c
endif

context:: $(ADB_UNPACKDIR)

clean::
	$(call DELFILE, $(OBJS))

distclean::
	$(call DELDIR, $(ADB_UNPACKDIR))
	$(call DELFILE, $(ADB_DIR)/$(CONFIG_ADB_TARBALL))

include $(APPDIR)/Application.mk
