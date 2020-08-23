############################################################################
# system/iotjs/Makefile
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

IOTJS_DIR := $(APPDIR)/system/iotjs

# FIXME we cannot use wget because there is no recent tag/release of iotjs
CONFIG_IOTJS_URL ?= "https://github.com/jerryscript-project/iotjs.git"
CONFIG_IOTJS_VERSION ?= 7fdafb0e9937ae92f4d703387ee75c8264ee67ae

IOTJS_PATCHS ?= $(addprefix ../,$(sort $(wildcard 000*.patch)))

$(IOTJS_DIR)/iotjs:
	@echo "Downloading: iot.js"
	$(call DELDIR, "$@")
	$(Q) mkdir "$@"
	$(Q) cd "$@" && git init && \
	git remote add origin "$(CONFIG_IOTJS_URL)" && \
	git fetch origin $(CONFIG_IOTJS_VERSION) --depth=1 && \
	git reset --hard FETCH_HEAD && \
	git am $(IOTJS_PATCHS) && \
	git submodule update --init

# iotjs application configuration

CONFIG_IOTJS_PRIORITY ?= SCHED_PRIORITY_DEFAULT
CONFIG_IOTJS_STACKSIZE ?= 16384

PROGNAME  := iotjs
PRIORITY  := $(CONFIG_IOTJS_PRIORITY)
STACKSIZE := $(CONFIG_IOTJS_STACKSIZE)
HEAPSIZE  := $(CONFIG_IOTJS_HEAPSIZE)
MODULE    := $(CONFIG_IOTJS)

MAINSRC := $(IOTJS_DIR)/iotjs/config/nuttx/stm32f4dis/app/iotjs_main.c
CSRCS   += $(IOTJS_DIR)/iotjs/config/nuttx/stm32f4dis/app/jerry_port.c

ifeq ($(CONFIG_ARCH_ARM),y)
# FIXME setjmp already in NuttX
# ASRCS = $(IOTJS_DIR)/iotjs/config/nuttx/stm32f4dis/app/setjmp.S
IOTJS_ARCH := arm
IOTJS_TARGET := arm
else ifeq ($(CONFIG_ARCH_SIM),y)
	ifeq ($(CONFIG_SIM_M32),y)
	IOTJS_ARCH := x86
	IOTJS_TARGET := i686
	else
	IOTJS_ARCH := x86_64
	IOTJS_TARGET := x86_64
	endif
endif

IOJS_BUILD_TYPE := debug
# release

IOTJS_BUILD_PATH := $(IOTJS_DIR)/iotjs/build/$(IOTJS_TARGET)-nuttx/$(IOJS_BUILD_TYPE)

CFLAGS += -I$(IOTJS_DIR)/iotjs/deps/jerry/jerry-core/include
CFLAGS += -I$(IOTJS_DIR)/iotjs/deps/jerry/jerry-ext/include

# iotjs library configuration

IOTJS_LIBS := libhttpparser.a libiotjs.a
IOTJS_LIBS += libjerry-core.a libjerry-libm.a libjerry-ext.a

IOTJS_LIBS := $(addprefix $(IOTJS_BUILD_PATH)/lib/,$(IOTJS_LIBS))

OTHER_OBJS := acos.c.obj                     debugger-common.c.obj  handler-print.c.obj           iotjs_module_console.c.obj      isnan.c.obj
OTHER_OBJS += acosh.c.obj                    debugger-rp.c.obj      handler-register.c.obj        iotjs_module_constants.c.obj    jerry-all-in.c.obj
OTHER_OBJS += arg.c.obj                      debugger-serial.c.obj  handler-resource-name.c.obj   iotjs_module_dns.c.obj          log10.c.obj
OTHER_OBJS += arg-js-iterator-helper.c.obj   debugger-sha1.c.obj    handle-scope-allocator.c.obj  iotjs_module_fs.c.obj           log1p.c.obj
OTHER_OBJS += arg-transform-functions.c.obj  debugger-tcp.c.obj     handle-scope.c.obj            iotjs_module_http_parser.c.obj  log2.c.obj
OTHER_OBJS += asin.c.obj                     debugger-ws.c.obj      http_parser.c.obj             iotjs_module_process.c.obj      log.c.obj
OTHER_OBJS += asinh.c.obj                    exp.c.obj              iotjs_binding.c.obj           iotjs_module_tcp.c.obj          module.c.obj
OTHER_OBJS += atan2.c.obj                    expm1.c.obj            iotjs_binding_helper.c.obj    iotjs_module_timer.c.obj        nextafter.c.obj
OTHER_OBJS += atan.c.obj                     fabs.c.obj             iotjs.c.obj                   iotjs_string.c.obj              pow.c.obj
OTHER_OBJS += atanh.c.obj                    finite.c.obj           iotjs_debuglog.c.obj          iotjs_string_ext.c.obj          scalbn.c.obj
OTHER_OBJS += cbrt.c.obj                     floor.c.obj            iotjs_env.c.obj               iotjs_systemio-nuttx.c.obj      sinh.c.obj
OTHER_OBJS += ceil.c.obj                     fmod.c.obj             iotjs_js.c.obj                iotjs_util.c.obj                sqrt.c.obj
OTHER_OBJS += copysign.c.obj                 handler-assert.c.obj   iotjs_module_buffer.c.obj     iotjs_uv_handle.c.obj           tanh.c.obj
OTHER_OBJS += cosh.c.obj                     handler-gc.c.obj       iotjs_module.c.obj            iotjs_uv_request.c.obj          trig.c.obj

OTHER_OBJS := $(addprefix build/,$(OTHER_OBJS))

# FIXME dirty hack to get "ar" only without "rcs"
AR_ONLY := $(firstword $(AR))

$(IOTJS_DIR)/iotjs/.built: $(IOTJS_DIR)/iotjs
	$(call DELDIR, $(IOTJS_DIR)/iotjs/build)
	cd $(IOTJS_DIR)/iotjs && \
	./tools/build.py \
	--target-arch=$(IOTJS_ARCH) \
	--target-os=nuttx \
	--nuttx-home=../../../../incubator-nuttx \
	--jerry-heaplimit=48 --buildtype=$(IOJS_BUILD_TYPE) \
	--external-include-dir=$(APPDIR)/system/libuv/libuv-1.38.1/include
	$(Q) touch $@

# FIXME build issues in iotjs. Maybe some wrong dependencies in this project build
$(IOTJS_BUILD_PATH)/lib/%.a: $(IOTJS_DIR)/iotjs/.built

%.c: $(IOTJS_DIR)/iotjs/.built
	$(Q) touch $@

build/%.c.obj: $(IOTJS_DIR)/iotjs/.built
	$(Q) mkdir -p build/
	$(foreach iotjs_lib,$(IOTJS_LIBS),$(shell cd build/ && $(AR_ONLY) x $(iotjs_lib)))

context:: $(IOTJS_DIR)/iotjs

clean::
	$(call DELFILE, $(OBJS) $(IOTJS_DIR)/iotjs/.built)
	$(call DELDIR, build/)

# distclean::
# $(call DELDIR, $(IOTJS_DIR)/iotjs)

include $(APPDIR)/Application.mk
