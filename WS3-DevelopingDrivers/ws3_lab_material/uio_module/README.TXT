--------------------------------------------------------------------------------
SETUP DEVELOPMENT ENVIRONMENT

Ensure that you have your development environment setup properly.

You should have the linaro cross compiler tools built and the linux kernel
source tree installed on your development host.

You should have the following environment variables defined ARCH, CROSS_COMPILE,
and OUT_DIR.  Here's an example of what they may look like:

export ARCH=arm
	This is set to 'arm' for our ARM achitecture processor.
	
export CROSS_COMPILE=<path-to>/toolchain/bin/arm-linux-gnueabihf-
	This is set to the path into the cross compiler toolchain and the
	prefix for the tools.
	
export OUT_DIR=<path-to>/socfpga-3.10-ltsi
	This is set to the path into the linux kernel source tree.

NOTE: the <path-to> text above refers to the absolute path to the installation
      location of the relevant directory.
      
--------------------------------------------------------------------------------
OBTAIN THE UIO TEMPLATE

Copy the file '${OUT_DIR}/drivers/uio/uio_pdrv_genirq.c' from the kernel source
tree into this directory and rename it 'my_uio_pdrv_genirq.c'.

You could run this sequence of commands to do this in a bash shell:
UIO_FILE="${OUT_DIR}/drivers/uio/uio_pdrv_genirq.c"
[ -f "${UIO_FILE}" ] && {
MY_UIO_FILE="my_$(basename ${UIO_FILE})"
cp "${UIO_FILE}" "${MY_UIO_FILE}"
} || {
echo ""
echo "ERROR: could not find file '${UIO_FILE}'"
echo ""
}

--------------------------------------------------------------------------------
EDIT THE UIO TEMPLATE

Edit the 'my_uio_pdrv_genirq.c' file, change the 'DRIVER_NAME' macro from
'uio_pdrv_genirq' to 'my_uio_pdrv_genirq', and add the compatible string entry
'{.compatible = "demo,driver-1.0"},' to the 'uio_of_genirq_match[]' array
declaration.

You could run this sequence of commands to do this in a bash shell:
[ -f "${MY_UIO_FILE}" ] && {
sed -i.bak -e '
/\#define DRIVER_NAME/ s/\"\(.*\)\"/\"my_\1\"/
/uio_of_genirq_match\[\] = / a\
{.compatible = "demo,driver-1.0"},
' ${MY_UIO_FILE}
} || {
echo ""
echo "ERROR: could not find file '${MY_UIO_FILE}'"
echo ""
}

--------------------------------------------------------------------------------
CREATE KBUILD FILE

Create a file named 'Kbuild' with this one line in it:
obj-m  := my_uio_pdrv_genirq.o

You could run this command to create this in a bash shell:
echo -e "obj-m  := my_uio_pdrv_genirq.o" > Kbuild

--------------------------------------------------------------------------------
CREATE MAKEFILE

Create a file named 'Makefile' with these lines in it:
ifndef OUT_DIR
    $(error OUT_DIR is undefined, bad environment, you point OUT_DIR to the linux kernel build output directory)
endif

KDIR ?= $(OUT_DIR)

default:
	$(MAKE) -C $(KDIR) M=$$PWD

clean:
	$(MAKE) -C $(KDIR) M=$$PWD clean

help:
	$(MAKE) -C $(KDIR) M=$$PWD help

modules:
	$(MAKE) -C $(KDIR) M=$$PWD modules

modules_install:
	$(MAKE) -C $(KDIR) M=$$PWD modules_install

You could run this sequence of commands to create this in a bash shell:
echo -e "ifndef OUT_DIR" > Makefile
echo -e "    \$(error OUT_DIR is undefined, bad environment, you point OUT_DIR to the linux kernel build output directory)" >> Makefile
echo -e "endif" >> Makefile
echo -e "" >> Makefile
echo -e "KDIR ?= \$(OUT_DIR)" >> Makefile
echo -e "" >> Makefile
echo -e "default:" >> Makefile
echo -e "\t\$(MAKE) -C \$(KDIR) M=\$\$PWD" >> Makefile
echo -e "" >> Makefile
echo -e "clean:" >> Makefile
echo -e "\t\$(MAKE) -C \$(KDIR) M=\$\$PWD clean" >> Makefile
echo -e "" >> Makefile
echo -e "help:" >> Makefile
echo -e "\t\$(MAKE) -C \$(KDIR) M=\$\$PWD help" >> Makefile
echo -e "" >> Makefile
echo -e "modules:" >> Makefile
echo -e "\t\$(MAKE) -C \$(KDIR) M=\$\$PWD modules" >> Makefile
echo -e "" >> Makefile
echo -e "modules_install:" >> Makefile
echo -e "\t\$(MAKE) -C \$(KDIR) M=\$\$PWD modules_install" >> Makefile

--------------------------------------------------------------------------------
EXECUTE MAKE

Now run the command 'make', this should build the uio_module kernel module.

--------------------------------------------------------------------------------
FINISHED

When this succeeds, you should see the module ELF file my_uio_pdrv_genirq.ko in
this directory.

