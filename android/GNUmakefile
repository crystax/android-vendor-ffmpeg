override MYDIR := $(abspath $(dir $(lastword $(MAKEFILE_LIST))))
override TOPDIR := $(abspath $(MYDIR)/..)

sinclude $(TOPDIR)/local.mk

#ABI ?= armeabi-v7a-hard
ABI ?= arm64-v8a

ifeq (,$(strip $(NDK)))
NDK := $(abspath $(wildcard $(TOPDIR)/../../platform/ndk))
endif

#==================================================================================================

define rm-if-exists
$(if $(wildcard $(1)),rm -Rf $(wildcard $(1)))
endef

define rmdir-if-exists
$(if $(wildcard $(1)),rmdir $(wildcard $(1)) >/dev/null 2>&1 || true)
endef

define out-dir
$(or $(strip $(OUTDIR)),$(TOPDIR)/android/out/$(1))
endef

define build-dir
$(call out-dir,$(1))/build
endef

define install-dir
$(call out-dir,$(1))/install
endef

define target-arch
$(strip $(or \
    $(if $(filter armeabi%,$(1)),arm),\
    $(if $(filter arm64-v8a,$(1)),aarch64),\
    $(if $(filter mips,$(1)),mipsel),\
    $(if $(filter mips64,$(1)),mips64el),\
    $(if $(filter x86,$(1)),x86),\
    $(if $(filter x86_64,$(1)),x86_64),\
    $(error Unsupported ABI: '$(1)')\
))
endef

define target-cpu
$(strip $(or \
    $(if $(filter armeabi,$(1)),armv5te),\
    $(if $(filter armeabi-v7a%,$(1)),armv7-a),\
    $(if $(filter arm64-v8a,$(1)),armv8-a),\
    $(if $(filter x86 x86_64,$(1)),atom),\
    $(if $(filter mips,$(1)),mips32r6),\
    $(if $(filter mips64,$(1)),mips64r6),\
    $(error Unsupported ABI: '$(1)')\
))
endef

#==================================================================================================

.PHONY: all
all: build

.PHONY: clean
clean:
	$(call rm-if-exists,$(call out-dir,$(ABI)))
	@$(call rmdir-if-exists,$(dir $(call out-dir,$(ABI))))

.PHONY: rebuild
rebuild:
	$(MAKE) clean
	$(MAKE) all

.PHONY: build
build:
	$(if $(strip $(NDK)),,$(error NDK was not defined!))
	$(if $(strip $(ABI)),,$(error ABI was not defined!))
	@$(strip \
		export NDK=$(NDK) && \
		export ABI=$(ABI) && \
		export OUTDIR=$(call out-dir,$(ABI)) && \
		source $(TOPDIR)/android/envsetup.sh && \
		mkdir -p $(call build-dir,$(ABI)) && \
		cd $(call build-dir,$(ABI)) && \
		$(TOPDIR)/configure \
			--prefix=$(call install-dir,$(ABI)) \
			--enable-cross-compile \
			--arch=$(call target-arch,$(ABI)) \
			--cpu=$(call target-cpu,$(ABI)) \
			--target-os=android \
			--cross-prefix=$${TARGET}- \
			--enable-pic \
			--disable-doc \
			--disable-symver \
			--disable-yasm \
			--enable-static \
			--enable-shared \
			$(if $(filter armeabi,$(ABI)),\
				--disable-vfp \
				--disable-neon \
			)\
			--enable-gpl \
			--enable-libx264 \
			&& \
		make -j && \
		make install && \
		true \
	)
