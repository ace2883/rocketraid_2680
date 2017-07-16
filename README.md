
rocketraid_2680

This is just where I'm keeping work while I ensure the v2.1 HPT Open Source drivers work with the latest kernels.

To use this, you gotta make from the product/linux directory.

Meh, maybe I'll update this more as I go...

Credit where credit is due - this is the culmination of a few days scouring the internet. It's not all mine, but I did the final patching to get this to work with kernel>=4.8.

Currently working with: Kernel 4.9.0-3-amd64,4.9.6,4.9.16,...

RocketRAID 268x SAS Controller Linux Open Source Driver
Copyright (C) 2012 HighPoint Technologies, Inc. All rights reserved.

1. Overview
2. Build the driver as a kernel module
3. Using the driver as a kernel patch
4. Revision History
5. Technical support and service

#############################################################################
1. Overview
#############################################################################

  This package contains Linux driver code for HighPoint RocketRAID 268x
  SAS controller. You can use it to build the driver module for custom
  Linux kernels.

  NO WARRANTY

  THE DRIVER SOURCE CODE HIGHPOINT PROVIDED IS FREE OF CHARGE, AND THERE IS
  NO WARRANTY FOR THE PROGRAM. THERE ARE NO RESTRICTIONS ON THE USE OF THIS
  FREE SOURCE CODE. HIGHPOINT DOES NOT PROVIDE ANY TECHNICAL SUPPORT IF THE
  CODE HAS BEEN CHANGED FROM ORIGINAL SOURCE CODE.

  LIMITATION OF LIABILITY

  IN NO EVENT WILL HIGHPOINT BE LIABLE FOR DIRECT, INDIRECT, SPECIAL,
  INCIDENTAL, OR CONSEQUENTIAL DAMAGES ARISING OUT OF THE USE OF OR
  INABILITY TO USE THIS PRODUCT OR DOCUMENTATION, EVEN IF ADVISED OF THE
  POSSIBILITY OF SUCH DAMAGES. IN PARTICULAR, HIGHPOINT SHALL NOT HAVE
  LIABILITY FOR ANY HARDWARE, SOFTWARE, OR DATA STORED USED WITH THE
  PRODUCT, INCLUDING THE COSTS OF REPAIRING, REPLACING, OR RECOVERING
  SUCH HARDWARE, OR DATA.


#############################################################################
2. Build the driver as a kernel module
#############################################################################

  NOTE: The latest tested kernel version: 4.9.*
  
  1) Install kernel build tools (gcc, binutils, make, etc.)
  
  2) Setup the kernel source/headers
  
     To build driver modules for a specific kernel, you shall use same 
     configuration for the kernel and the driver. Otherwise, the driver may 
     be unable to load, or behave abnormally.
     
     - For Linux kernel 2.6.* and 3.*-
     
     On most distributions based on kernel 2.6 and 3.*, an exploded source tree is not
     required to build a driver against the currently in-use kernel. As long
     as the system has kernel headers setup under /lib/modules/`uname -r`/build,
     you can simply run "make" to build the driver.

     If you want to build the driver against a custom kernel source, you must
     setup the kernel source manually and run "make" under kernel source tree
     to setup kernel headers.

     - For Linux kernel 2.4 -
     
     You need a full kernel source tree to build the driver. If you are building
     a driver for the currently in-use kernel, the kernel source should match
     the version of the running kernel. In addition, you must obtain the config
     file for the running kernel:
     
        For Red Hat or Fedora Linux, you can find the stock kernel config files
        under {kernel-source-dir}/configs, named kernel-*.config. Select the one
        matches your hardware and copy it to {kernel-source-dir}/.config.
        
        For SuSE Linux, the config files for current kernel is under /boot dir
        so you can the kernel source by
        
            # cd /usr/src/linux-{kernel-version}.SuSE
            # cp /boot/vmlinuz.config .config
            # cp /boot/vmlinuz.version.h include/linux/version.h
            # cp /boot/vmlinuz.autoconf.h include/linux/autoconf.h

        For other distributions please refer to the distribution documents to 
        obtain the config file.
        
        If you are building a custom kernel and a driver, you can setup the kernel
        config by yourself, using "make config" or "make menuconfig" commands
        under kernel source directory.
        
     Once the kernel config file is ready, run following commands under kernel
     source directory to setup kernel headers:
     
            # make oldconfig
            # make dep
        
     On some Red Hat versions, {kernel-source-dir}/include/linux/modversions.h
     may be incorrect after "make dep", you need check if there is a line 
     "#include <linux/rhconfig.h>" before "#include <linux/modsetver.h>" in that
     file. If not, please add the line manually.

  3) Build the driver
  
     After you have configured the kernel source/headers you can build the driver
     by following commands (assume you have extracted the driver source under
     directory rocketraid_2680):

        # cd rocketraid_2680/product/rr2680/linux/
        # make
        
     You can append below options to "make" command:
     
        KERNELDIR=...
           Specify kernel source directory. If not specified the driver will
           use /lib/modules/{kernel-version}/build/ as the default location.
           This option is needed if you have a manually configured kernel
           source tree.
     
        CROSS_COMPILE=...
           Specify cross compiler prefix.
           
        ARCH=...
           Specify target system machine architecture.
           Currently only i386 and x86_64 is supported.

  4) Install/upgrade the driver
  
     If you are building driver for currently in-use kernel you can use
     "make install" to install or upgrade the driver:

        # cd rocketraid_2680/product/rr2680/linux/
        # make install
        
     The KERNELDIR=... parameter may be required, e.g.
     
        # make install KERNELDIR=/usr/src/linux-2.6

     "make install" command will copy the driver module to the directory
     /lib/modules/`uname -r`/kernel/drivers/scsi, and update the initrd file
     if it contains an old version driver.
        
     If your system is installed on the controller, a reboot is required
     to make the new driver take effect.
     
     Please note "make install" may not work on some distributions. In that
     case you have to install the driver manually.
     
     After the driver is installed, it can be loaded manually by

        # modprobe rr2680
        
     To load the driver automatically during system startup, you can either
     put it into initrd file or configure a /etc/rc.d script to load it.
        
     
#############################################################################
3. Using the driver as a kernel patch
#############################################################################

     You must have a full kernel source tree to use the driver as a patch.
     To patch a kernel source tree, run the command
     
        # cd rocketraid_2680/product/rr2680/linux/
        # make patchkernel KERNELDIR=<kernel-source-dir>
        
     For an unconfigured 2.6 kernel source tree, include/linux/version.h may
     not exist so you should add "KERNEL_VER=2.6" to the command:

        # make patchkernel KERNELDIR=<kernel-source-dir> KERNEL_VER=2.6
        
     Then you can configure the driver into kernel during the kernel 
     configuration process (e.g. "make menuconfig"). It is listed under
     scsi low level drivers.
     
     Below is an example to make and install a kernel with the driver built-in:
     
        # cd /usr/src/linux-2.6.25
        # make menuconfig
        
        Select "Device Drivers --->" and press enter.
        Select "SCSI device support", then press 'Y' to make it built-in.
        Select "SCSI disk support" then press 'Y' to make it build-in.
        Select "SCSI low-level drivers --->" and press enter.
        Select "HighPoint RocketRAID 268x support" and press 'Y'.
        Exit and save the kernel configuration.
        
        # make
        # make modules_install
        # make install
        
        Then you can reboot from the new kernel.
     
        
#############################################################################
4. Revision History
#############################################################################

   * fixed 4.9.*
   * add DKMS

   v2.1 02/24/2014
     * If failed to install the required build toolchain, display a warning
      message.
     * Modify the modprobe config file, so SLES could load third party module
       by default.
     * Support openSUSE 12.1 systemd program.

   v2.1 01/15/2014
     * Support openSUSE 12.3 and 13.1.
     * Source updated to support WebGUI communication without /proc/scsi entry.

   v2.0 01/13/2014
     * Minor changes to support Debian Linux 7.x, quiet installation.

   v2.0 01/08/2014
     * Minor changes to support Fedora Linux 19+.

   v2.0 01/04/2014
     * Support Kernel 3.12.5.
     * Executable file to install and build source package.
     * Automatically install required package if it is not installed.

   v2.0 06/26/2013
     * Support Linux kernel up to 3.9.7.

   v1.9.12.0817  08/17/2012
     * Fixed a potential bug about fail to recover array.

   v1.8.12.0713  07/13/2012
     * Fixed a bug about interrupt caused OS lock up on some motherboards.

   v1.7.12.0524  05/24/2012
     * Add support over 2TB HDD(Need to update BIOS to v1.3 or later).
     * Add recover array function(Need to update WebGUI to v2.1 or later).
     * Add fail led when disk down.
     * Support Linux kernel up to 3.0.
     * Fix error handling.

   v1.6.11.0316  03/16/2011
     * Support Linux Kernel up to 2.6.37.

   v1.3.09.1010  10/10/2009
     * Support Linux Kernel up to 2.6.31.

   v1.2.09.0917  09/17/2009
     * Fixup compatibility with ACCUSYS ACS90000 expander.
     * Support SAS disk spindown standby.
     * Support 48-bit address for ide passthrough.
     * Support RocketRAID 2684 SAS controller.

   v1.0.08.0822  08/22/2008
     * First release.
