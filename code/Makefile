# Makefile
# Comment/uncomment the following line to disable/enable debugging
# DEBUG = y

# Usage
# make CROSS_COMPILE=<cross_compiler_prefix> KERNEL_DIR=<your_kernel_dir> KERNEL_BUID_OUTPUT=<kernel_buid_output>
#
#make ARCH=arm64 CROSS_COMPILE=/home/zeroway/ESPRESSObin/tool/gcc-linaro-5.2-2015.11-2-x86_64_aarch64-linux-gnu/bin/aarch64-linux-gnu- KERNEL_DIR=/home/zeroway/ESPRESSObin/src/kernel/linux-marvell-linux-4.4.8-armada-17.02-espressobin KERNEL_BUID_OUTPUT=/home/zeroway/ESPRESSObin/src/kernel/linux-marvell-linux-4.4.8-armada-17.02-espressobin/out

# Add your debugging flag (or not) to CFLAGS
ifeq ($(DEBUG),y)
DEBFLAGS = -O -g -DSCULL_DEBUG # "-O" is needed to expand inlines
else
DEBFLAGS = -O2
endif

# codec es8323
obj-m := hello.o


KERNEL_DIR ?= /lib/modules/$(shell uname -r)/build
KERNEL_BUID_OUTPUT ?=$(KERNEL_DIR)
CC = $(CROSS_COMPILE)gcc
LD = $(CROSS_COMPILE)ld
PWD := $(shell pwd)

modules:
	$(MAKE) -C $(KERNEL_DIR) M=$(PWD) O=$(KERNEL_BUID_OUTPUT) modules

clean:
	rm -rf *.o *~ core .depend .*.cmd *.ko *.mod.c .tmp_versions modules.order Module.symvers

depend .depend dep:
	$(CC) $(CFLAGS) -M *.c > .depend

ifeq (.depend,$(wildcard .depend))
include .depend
endif
