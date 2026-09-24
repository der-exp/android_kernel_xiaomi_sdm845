# SPDX-License-Identifier: GPL-2.0
#
# Copyright (C) 2015-2019 Jason A. Donenfeld <Jason@zx2c4.com>. All Rights Reserved.

ccflags-y := -D'pr_fmt(fmt)=KBUILD_MODNAME ": " fmt'
ccflags-y += -Wframe-larger-than=2048
ccflags-$(CONFIG_AMNEZIAWG_DEBUG) += -DDEBUG -g
ccflags-$(if $(WIREGUARD_VERSION),y,) += -D'WIREGUARD_VERSION="$(WIREGUARD_VERSION)"'
ccflags-$(if $(OMIT_ENDPOINTS),y,) += -D'OMIT_ENDPOINTS="$(OMIT_ENDPOINTS)"'

amneziawg-y := main.o noise.o device.o peer.o timers.o queueing.o send.o receive.o socket.o peerlookup.o allowedips.o ratelimiter.o cookie.o netlink.o junk.o header_protection.o

ifeq ($(shell [ $(VERSION) -lt 6 -o \( $(VERSION) -eq 6 -a $(PATCHLEVEL) -lt 1 \) ] && echo y),y)
	kbuild-dir := $(if $(filter /%,$(src)),$(src),$(srctree)/$(src))
else
	kbuild-dir := $(src)
endif

ifeq ($(shell [ $(VERSION) -lt 5 -o \( $(VERSION) -eq 5 -a $(PATCHLEVEL) -lt 10 \) ] && echo y),y)
	include $(src)/crypto/Kbuild.include
endif
include $(src)/compat/Kbuild.include

obj-$(if $(KBUILD_EXTMOD),m,$(CONFIG_AMNEZIAWG)) := amneziawg.o

# Встроенный в ядро (=y) модуль живёт в одном vmlinux со встроенным WireGuard, а у них общая
# родословная: одни и те же глобальные функции (wg_device_init, wg_noise_init, …) и один и
# тот же zinc (chacha20, poly1305_init, curve25519, zinc_blake2s_*), и компоновщик находит
# каждое имя дважды. Наружу модуль ничего не отдаёт — EXPORT_SYMBOL в нём нет, а ядро
# вызывает его только через initcall и таблицы операций, — поэтому после сборки составного
# amneziawg.o все его определённые символы делаются локальными. Исходники при этом не
# трогаются, и обновление модуля — это снова копия его src/ без переименований.
#
# .llvm_addrsig удаляется, потому что это таблица номеров символов для ICF, и после правки
# таблицы символов objcopy она указывала бы не туда; ядро без неё собирается так же.
#
# Для модуля (=m) и сборки вне дерева не нужно: у модулей свои пространства имён, а
# init_module/cleanup_module должны остаться глобальными.
ifeq ($(if $(KBUILD_EXTMOD),,$(CONFIG_AMNEZIAWG)),y)
override cmd_link_multi-y = $(LD) $(ld_flags) -r -o $@ $(link_multi_deps) $(cmd_secanalysis) && \
	$(OBJCOPY) --wildcard --localize-symbol='*' --remove-section=.llvm_addrsig $@
endif
