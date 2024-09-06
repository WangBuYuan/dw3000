#KDIR ?= /lib/modules/$(shell uname -r)/build
ARCH=arm64
MAKE = make
CROSS_COMPILE=$(CURDIR)/../../../../../../../../prebuilts/gcc/linux-x86/aarch64/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu-
KDIR ?= ../../../../../../../../out/kernel/OBJ/linux-5.10
PROJECTS_BUILD := $(abspath $(CURDIR)/../projects/linux-build)

ifeq ($(PROJECTS_BUILD),$(abspath $(KDIR)/../../../../..))

# Build in projects/linux-build, use a separated build tree.

O := ../../../../../../../../out/kernel/OBJ/linux-5.10
M := .
B := $(abspath $O/$M)

$(info *** Building in $B)

default all: modules

modules modules_install:
	mkdir -p $B
	ln -srTf include $B/include
	ln -srTf Kbuild $B/Kbuild
	$(MAKE) LLVM=1 LLVM_IAS=1 ARCH=${ARCH} -C $(KDIR) CROSS_COMPILE=${CROSS_COMPILE} O=$O M=$M $@

clean help kernelrelease:
	$(MAKE) LLVM=1 LLVM_IAS=1 ARCH=${ARCH} -C $(KDIR) CROSS_COMPILE=${CROSS_COMPILE} O=$O M=$M $@

bindeb-pkg:
	./builddeb -C $(KDIR) CROSS_COMPILE=${CROSS_COMPILE} O=$O M=$M

else # ifeq ($(PROJECTS_BUILD),$(abspath $(KDIR)/../../../../..))

# Regular build, build here.
ifneq ($(KERNEL_SRC),)
# Compile Android external module in OUT directory
# KERNEL_SRC = Kernel path
$(info *** Building Android in $O/$M )

KDIR := $(KERNEL_SRC)

all:
	$(info *** Building Android in $O/$M )
	$(MAKE) LLVM=1 LLVM_IAS=1 ARCH=${ARCH} -C $(KDIR) CROSS_COMPILE=${CROSS_COMPILE} M=$(M) modules $(KBUILD_OPTIONS)

else # ifneq ($(KERNEL_SRC),)
M := $(CURDIR)
endif # ifneq ($(KERNEL_SRC),)

B := $(abspath $O/$M)

default:
	$(MAKE) LLVM=1 LLVM_IAS=1 ARCH=${ARCH} -C $(KDIR) M=$(M) CROSS_COMPILE=${CROSS_COMPILE}
modules modules_install clean help kernelrelease:
	$(MAKE) LLVM=1 LLVM_IAS=1 ARCH=${ARCH} -C $(KDIR) M=$(M) CROSS_COMPILE=${CROSS_COMPILE} $@

endif # ifeq ($(PROJECTS_BUILD),$(abspath $(KDIR)/../../../../..))

# Install generated dw3000.dtbo
dtbo_install: modules_install
	d=`echo $(INSTALL_MOD_PATH)/lib/modules/* | sed 's#lib/modules#boot/dtbs#'` && \
	mkdir -p $$d/overlays && \
	find $B -name '*.dtbo' -print -exec cp -v {} $$d/overlays \;

# Run kunit tests
tests:
	../tools/kunit/run $(KDIR)

# Run coverage
cov:
	../tools/kunit/run $(KDIR)
