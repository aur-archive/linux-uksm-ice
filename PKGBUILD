# Maintainer: Felix Yan <felixonmars@gmail.com>
# Contributor: Kelvin Ng (qpalz) <kelvin9302104 at gmail dot com>
# Contributor: Piotr Gorski <sir_lucjan@openlinux.pl>

### PATCH AND BUILD OPTIONS
# Set these variables to ANYTHING (yes or no or 1 or 0 or "I like icecream") to enable them
_makenconfig=		# tweak kernel options prior to a build via nconfig
_localmodcfg=		# compile ONLY probed modules
_use_current=		# use the current kernel's .config file
_BFQ_enable_=           # Enable BFQ as the default I/O scheduler
_NUMA_off=yes		# Disable NUMA in kernel config
#MAKEFLAGS=

### DOCS
# DETAILS FOR _localmodcfg="y"
# As of mainline 2.6.32, running with this option will only build the modules that you currently have
# probed in your system VASTLY reducing the number of modules built and the build time to do it.
#
# WARNING - make CERTAIN that all modules are modprobed BEFORE you begin making the pkg!
#
# To keep track of which modules are needed for your specific system/hardware, give my module_db script
# a try: http://aur.archlinux.org/packages.php?ID=41689  Note that if you use my script, this PKGBUILD 
# will auto run the 'sudo modprobed_db reload' for you to probe all the modules you have logged!
#
# More at this wiki page ---> https://wiki.archlinux.org/index.php/Modprobed_db

# DETAILS FOR _use_current="y"
# Enabling this option will use the .config of the RUNNING kernel rather than the ARCH defaults.
# Useful when the package gets updated and you already went through the trouble of customizing your
# config options.  NOT recommended when a new kernel is released, but again, convenient for package bumps.

pkgname=linux-uksm-ice
true && pkgname=(linux-uksm-ice linux-uksm-ice-headers)
_kernelname=-uksm-ice
_srcname=linux-3.13
pkgver=3.13.6
pkgrel=3
arch=('i686' 'x86_64')
url="http://kerneldedup.org/"
license=('GPL2')
options=('!strip')
makedepends=('kmod' 'inetutils' 'bc')
_uksmvernel="0.1.2.2"
_uksmname="v3.13"
_gcc_patch="enable_additional_cpu_optimizations_for_gcc.patch"
_toipatch=tuxonice-for-linux-3.13.5-2014-03-05.patch
_bfqpath="http://algo.ing.unimo.it/people/paolo/disk_sched/patches/3.13.0-v7r2"
source=("http://www.kernel.org/pub/linux/kernel/v3.x/${_srcname}.tar.xz"
        "http://www.kernel.org/pub/linux/kernel/v3.x/patch-${pkgver}.xz"
        "http://repo-ck.com/source/gcc_patch/${_gcc_patch}.gz"
        #"http://kerneldedup.org/download/uksm/${_uksmvernel}/uksm-${_uksmvernel}-for-${_uksmname}.patch" # website often went down
        "uksm-${_uksmvernel}-for-${_uksmname}.patch"
        "${_bfqpath}/0001-block-cgroups-kconfig-build-bits-for-BFQ-v7r2-3.13.patch"
        "${_bfqpath}/0002-block-introduce-the-BFQ-v7r2-I-O-sched-for-3.13.patch"
        "${_bfqpath}/0003-block-bfq-add-Early-Queue-Merge-EQM-to-BFQ-v7r2-for-3.13.0.patch"
        'linux-uksm-ice.preset'
        'change-default-console-loglevel.patch'
        'criu-no-expert.patch'
        '0001-Bluetooth-allocate-static-minor-for-vhci.patch'
        '0001-sunrpc-create-a-new-dummy-pipe-for-gssd-to-hold-open.patch'
        '0002-sunrpc-replace-sunrpc_net-gssd_running-flag-with-a-m.patch'
        '0003-nfs-check-if-gssd-is-running-before-attempting-to-us.patch'
        '0004-rpc_pipe-remove-the-clntXX-dir-if-creating-the-pipe-.patch'
        '0005-sunrpc-add-an-info-file-for-the-dummy-gssd-pipe.patch'
        '0006-rpc_pipe-fix-cleanup-of-dummy-gssd-directory-when-no.patch'
        '0001-syscalls.h-use-gcc-alias-instead-of-assembler-aliase.patch'
        'i8042-fix-aliases.patch'
        'config'
        'config.x86_64'
        "http://tuxonice.net/downloads/all/${_toipatch}.bz2")
        
prepare() {
    cd ${_srcname}

    ### add upstream patch
    patch -Np1 -i "${srcdir}/patch-${pkgver}"

    ### set DEFAULT_CONSOLE_LOGLEVEL to 4 (same value as the 'quiet' kernel param)
    # remove this when a Kconfig knob is made available by upstream
    # (relevant patch sent upstream: https://lkml.org/lkml/2011/7/26/227)
    msg "Patching set DEFAULT_CONSOLE_LOGLEVEL to 4"
    patch -p1 -i "${srcdir}/change-default-console-loglevel.patch"

    # allow Checkpoint/restore (for criu) without EXPERT=y
    patch -p1 -i "${srcdir}/criu-no-expert.patch"

    # fix 15 seocnds nfs delay
    # http://git.linux-nfs.org/?p=trondmy/linux-nfs.git;a=commitdiff;h=4b9a445e3eeb8bd9278b1ae51c1b3a651e370cd6
    patch -p1 -i "${srcdir}/0001-sunrpc-create-a-new-dummy-pipe-for-gssd-to-hold-open.patch"
    # http://git.linux-nfs.org/?p=trondmy/linux-nfs.git;a=commitdiff;h=89f842435c630f8426f414e6030bc2ffea0d6f81
    patch -p1 -i "${srcdir}/0002-sunrpc-replace-sunrpc_net-gssd_running-flag-with-a-m.patch"
    # http://git.linux-nfs.org/?p=trondmy/linux-nfs.git;a=commitdiff;h=6aa23d76a7b549521a03b63b6d5b7880ea87eab7
    patch -p1 -i "${srcdir}/0003-nfs-check-if-gssd-is-running-before-attempting-to-us.patch"

    # fix nfs kernel oops
    # http://git.linux-nfs.org/?p=trondmy/linux-nfs.git;a=commitdiff;h=3396f92f8be606ea485b0a82d4e7749a448b013b
    patch -p1 -i "${srcdir}/0004-rpc_pipe-remove-the-clntXX-dir-if-creating-the-pipe-.patch"
    # http://git.linux-nfs.org/?p=trondmy/linux-nfs.git;a=commitdiff;h=e2f0c83a9de331d9352185ca3642616c13127539
    patch -p1 -i "${srcdir}/0005-sunrpc-add-an-info-file-for-the-dummy-gssd-pipe.patch"
    # http://git.linux-nfs.org/?p=trondmy/linux-nfs.git;a=commitdiff;h=23e66ba97127ff3b064d4c6c5138aa34eafc492f
    patch -p1 -i "${srcdir}/0006-rpc_pipe-fix-cleanup-of-dummy-gssd-directory-when-no.patch"


	# Fix vhci warning in kmod (to restore every kernel maintainer's sanity)
	patch -p1 -i "${srcdir}/0001-Bluetooth-allocate-static-minor-for-vhci.patch"

   # Fix symbols: Revert http://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=83460ec8dcac14142e7860a01fa59c267ac4657c
    patch -Rp1 -i "${srcdir}/0001-syscalls.h-use-gcc-alias-instead-of-assembler-aliase.patch"

    # Fix i8042 aliases
    patch -p1 -i "${srcdir}/i8042-fix-aliases.patch"
    
 # Patch source to enable more gcc CPU optimizatons via the make nconfig
	patch -Np1 -i "${srcdir}/${_gcc_patch}"
    
    msg "Patching source with BFQ patches"
    for p in $(ls ${srcdir}/000{1,2,3}-block*.patch); do
        patch -Np1 -i "$p"
    done

    ### Patch source with UKSM
    msg "Patching with UKSM"
    cp "${srcdir}/uksm-${_uksmvernel}-for-${_uksmname}.patch" ./
    #patch -Np0 -i "${srcdir}/uksm-${_uksmvernel}-for-${_uksmname}.patch.patch"
    patch -Np1 -i ./uksm-${_uksmvernel}-for-${_uksmname}.patch

    # tuxonice patch
    patch -p1 -i "${srcdir}/${_toipatch}"


### Clean tree and copy ARCH config over
    msg "Running make mrproper to clean source tree"
    make mrproper

    if [ "${CARCH}" = "x86_64" ]; then
        cat "${srcdir}/config.x86_64" > ./.config
    else
        cat "${srcdir}/config" > ./.config
    fi

    ### Optionally use running kernel's config
    # code originally by nous; http://aur.archlinux.org/packages.php?ID=40191
    if [ -n "$_use_current" ]; then
	if [[ -s /proc/config.gz ]]; then
	    msg "Extracting config from /proc/config.gz..."
	    # modprobe configs
	    zcat /proc/config.gz > ./.config
	else
	    warning "You kernel was not compiled with IKCONFIG_PROC!"
	    warning "You cannot read the current config!"
	    warning "Aborting!"
	    exit
	fi
    fi

    if [ "${_kernelname}" != "" ]; then
	sed -i "s|CONFIG_LOCALVERSION=.*|CONFIG_LOCALVERSION=\"${_kernelname}\"|g" ./.config
	sed -i "s|CONFIG_LOCALVERSION_AUTO=.*|CONFIG_LOCALVERSION_AUTO=n|" ./.config
    fi

    ### UKSM to be enabled
    sed -i -e s'/CONFIG_KSM=y/CONFIG_KSM=y\nCONFIG_UKSM=y/' ./.config

    ### BFQ to be compiled in but not enabled
    sed -i -e s'/CONFIG_CFQ_GROUP_IOSCHED=y/CONFIG_CFQ_GROUP_IOSCHED=y\nCONFIG_IOSCHED_BFQ=y\nCONFIG_CGROUP_BFQIO=y/' \
        -i -e s'/CONFIG_DEFAULT_CFQ=y/CONFIG_DEFAULT_CFQ=y\n# CONFIG_DEFAULT_BFQ is not set/' ./.config

    ### Optionally enable BFQ as the default io scheduler
    if [ -n "$_BFQ_enable_" ]; 	then
        sed -i -e '/CONFIG_DEFAULT_IOSCHED/ s,cfq,bfq,' \
        -i -e s'/CONFIG_DEFAULT_CFQ=y/# CONFIG_DEFAULT_CFQ is not set\nCONFIG_DEFAULT_BFQ=y/' ./.config
    fi

    # disable NUMA since 99.9% of users do not have multiple CPUs but do have multiple cores in one CPU
    # see, https://bugs.archlinux.org/task/31187
    if [ -n "$_NUMA_off" ]; then
	if [ "${CARCH}" = "x86_64" ]; then
	    sed -i -e 's/CONFIG_NUMA=y/# CONFIG_NUMA is not set/' \
		    -i -e '/CONFIG_AMD_NUMA=y/d' \
		    -i -e '/CONFIG_X86_64_ACPI_NUMA=y/d' \
		    -i -e '/CONFIG_NODES_SPAN_OTHER_NODES=y/d' \
		    -i -e '/# CONFIG_NUMA_EMU is not set/d' \
		    -i -e '/CONFIG_NODES_SHIFT=6/d' \
		    -i -e '/CONFIG_NEED_MULTIPLE_NODES=y/d' \
		    -i -e '/CONFIG_USE_PERCPU_NUMA_NODE_ID=y/d' \
		    -i -e '/CONFIG_ACPI_NUMA=y/d' ./.config
	fi
    fi

    # set extraversion to pkgrel
    sed -ri "s|^(EXTRAVERSION =).*|\1 -${pkgrel}|" Makefile

    # don't run depmod on 'make install'. We'll do this ourselves in packaging
    sed -i '2iexit 0' scripts/depmod.sh

    # get kernel version
    msg "Running make prepare for you to enable patched options of your choosing"
    make prepare

    ### Optionally load needed modules for the make localmodconfig
    # See http://aur.archlinux.org/packages.php?ID=41689
    if [ -n "$_localmodcfg" ]; then
	msg "If you have modprobe_db installed, running it in recall mode now"
	if [ -e /usr/bin/modprobed_db ]; then
	    [[ ! -x /usr/bin/sudo ]] && echo "Cannot call modprobe with sudo. Install via pacman -S sudo and configure to work with this user." && exit 1
	    sudo /usr/bin/modprobed_db recall
	fi
	msg "Running Steven Rostedt's make localmodconfig now"
	make localmodconfig
    fi

    if [ -n "$_makenconfig" ]; then
	msg "Running make nconfig"
	make nconfig
    fi
}

build() {
    cd ${_srcname}
    msg "Running make bzImage and modules"
    make ${MAKEFLAGS} LOCALVERSION= bzImage modules
}

package_linux-uksm-ice() {
    _Kpkgdesc='Linux Kernel and modules with the UKSM patchset featuring the  v1.2.2. and TuxOnIce patchset'
    pkgdesc="${_Kpkgdesc}"
    depends=('coreutils' 'linux-firmware' 'mkinitcpio>=0.7')
    optdepends=('crda' 'nvidia-uksm' 'modprobed_db')
    provides=("linux-uksm-ice=${pkgver}")
    conflicts=('kernel26-uksm')
    replaces=('kernel26-uksm')
    backup=("etc/mkinitcpio.d/linux-uksm-ice.preset")
    install=linux-uksm-ice.install

    cd ${_srcname}

    KARCH=x86

    # get kernel version
    _kernver="$(make LOCALVERSION= kernelrelease)"
    _basekernel=${_kernver%%-*}
    _basekernel=${_basekernel%.*}

    mkdir -p "${pkgdir}"/{lib/modules,lib/firmware,boot}
    make LOCALVERSION= INSTALL_MOD_PATH="${pkgdir}" modules_install
    cp arch/$KARCH/boot/bzImage "${pkgdir}/boot/vmlinuz-linux-uksm-ice"

    # set correct depmod command for install
    cp -f "${startdir}/${install}" "${startdir}/${install}.pkg"
    true && install=${install}.pkg
    sed \
    -e "s/KERNEL_NAME=.*/KERNEL_NAME=-uksm-ice/g" \
    -e "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i "${startdir}/${install}"

    # install mkinitcpio preset file for kernel
    install -D -m644 "${srcdir}/linux-uksm-ice.preset" "${pkgdir}/etc/mkinitcpio.d/linux-uksm-ice.preset"
    sed \
    -e "1s|'linux.*'|'linux-uksm-ice'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"/boot/vmlinuz-linux-uksm-ice\"|" \
    -e "s|default_image=.*|default_image=\"/boot/initramfs-linux-uksm-ice.img\"|" \
    -e "s|fallback_image=.*|fallback_image=\"/boot/initramfs-linux-uksm-ice-fallback.img\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/linux-uksm-ice.preset"

    # remove build and source links
    rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
    # remove the firmware
    rm -rf "${pkgdir}/lib/firmware"
    # gzip -9 all modules to save 100MB of space
    find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;
    # make room for external modules
    ln -s "../extramodules-${_basekernel}${_kernelname:uksm}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
    # add real version for building modules and running depmod from post_install/upgrade
    mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:ukdm}"
    echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:uksm}/version"

    # Now we call depmod...
    depmod -b "${pkgdir}" -F System.map "${_kernver}"

    # move module tree /lib -> /usr/lib
    mkdir -p "${pkgdir}/usr"
    mv "${pkgdir}/lib" "${pkgdir}/usr/"

    # add vmlinux
    install -D -m644 vmlinux "${pkgdir}/usr/lib/modules/${_kernver}/build/vmlinux"
}

package_linux-uksm-ice-headers() {
    _Hpkgdesc='Header files and scripts to build modules for linux-uksm-ice.'
    pkgdesc="${_Hpkgdesc}"
    provides=("linux-uksm-ice-headers=${pkgver}" "linux-headers=${pkgver}")
    conflicts=('kernel26-uksm-ice-headers')
    replaces=('kernel26-uksm-ice-headers')

    install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

    cd ${_srcname}
    install -D -m644 Makefile \
	    "${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
    install -D -m644 kernel/Makefile \
	    "${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
    install -D -m644 .config \
	    "${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

    for i in acpi asm-generic config crypto drm generated keys linux math-emu \
	    media net pcmcia scsi sound trace uapi video xen; do
	cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
    done

    # copy arch includes for external modules
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86"
    cp -a arch/x86/include "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/x86/"

    # copy files necessary for later builds, like nvidia and vmware
    cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
    cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

    # fix permissions on scripts dir
    chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

    cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

    if [ "${CARCH}" = "i686" ]; then
	    cp arch/${KARCH}/Makefile_32.cpu "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"
    fi

    cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

    # add headers for lirc package
    # pci
    for i in bt8xx cx88 saa7134; do
	    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/pci/${i}"
	    cp -a drivers/media/pci/${i}/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/pci/${i}"
    done
    # usb
    for i in cpia2 em28xx pwc sn9c102; do
	    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/${i}"
	    cp -a drivers/media/usb/${i}/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/${i}"
    done
    # i2c
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c"
    cp drivers/media/i2c/*.h  "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
    for i in cx25840; do
	    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/${i}"
	    cp -a drivers/media/i2c/${i}/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/${i}"
    done

    # add docbook makefile
    install -D -m644 Documentation/DocBook/Makefile \
	    "${pkgdir}/usr/lib/modules/${_kernver}/build/Documentation/DocBook/Makefile"

    # add dm headers
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"
    cp drivers/md/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/md"

    # add inotify.h
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux"
    cp include/linux/inotify.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/linux/"

    # add wireless headers
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"
    cp net/mac80211/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/net/mac80211/"

    # add dvb headers for external modules
    # in reference to:
    # http://bugs.archlinux.org/task/9912
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core"
    cp drivers/media/dvb-core/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-core/"
    # and...
    # http://bugs.archlinux.org/task/11194
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"
    [[ -e include/config/dvb/ ]] && cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/" 

    # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
    # in reference to:
    # http://bugs.archlinux.org/task/13146
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
    cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
    cp drivers/media/i2c/msp3400-driver.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"

    # add dvb headers
    # in reference to:
    # http://bugs.archlinux.org/task/20402
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb"
    cp drivers/media/usb/dvb-usb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/usb/dvb-usb/"
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends"
    cp drivers/media/dvb-frontends/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners"
    cp drivers/media/tuners/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/tuners/"

    # add xfs and shmem for aufs building
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs"
    mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/mm"
    cp fs/xfs/xfs_sb.h "${pkgdir}/usr/lib/modules/${_kernver}/build/fs/xfs/xfs_sb.h"

    # copy in Kconfig files
    for i in `find . -name "Kconfig*"`; do
	    mkdir -p "${pkgdir}"/usr/lib/modules/${_kernver}/build/`echo ${i} | sed 's|/Kconfig.*||'`
	    cp ${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/${i}"
    done

    chown -R root.root "${pkgdir}/usr/lib/modules/${_kernver}/build"
    find "${pkgdir}/usr/lib/modules/${_kernver}/build" -type d -exec chmod 755 {} \;

    # strip scripts directory
    find "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts" -type f -perm -u+w 2>/dev/null | while read binary ; do
    case "$(file -bi "${binary}")" in
	    *application/x-sharedlib*) # Libraries (.so)
		    /usr/bin/strip ${STRIP_SHARED} "${binary}";;
	    *application/x-archive*) # Libraries (.a)
		    /usr/bin/strip ${STRIP_STATIC} "${binary}";;
	    *application/x-executable*) # Binaries
		    /usr/bin/strip ${STRIP_BINARIES} "${binary}";;
    esac
    done

    # remove unneeded architectures
    rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arm,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,xtensa}
}

# Global pkgdesc and depends are here so that they will be picked up by AUR
pkgdesc='Linux Kernel and modules with the UKSM patchset and TuxOnIce patchset'

sha512sums=('1ba223bb4b885d691a67196d86a8aaf7b4a1c351bf2a762f50f1b0c32da00dd0c28895872a66b49e8d244498d996876609268e64861d28ac4048886ef9f79b87'
            '15e5b32df65816a0183bf8bb84a7c98066984821dbc84e2d2d473a7065643442d138f7790aa2c47fe5f44ae6919fb6035ec824282d56d73affb48e53f0b1c485'
            'a149a6c62c6654b5efe76308037d37b9421114e6257343a22eae84f4489e210927d1db16e5742fd04bb4eea5c8a9c1e541d9bbf82d495fffaae9d645a5fd5d3f'
            '51d86d26bc9e5e68f613bffbfc0df78a9b6a006155f9eab42fb05d88309605976d55056d26e163d181c0350de0ae85c0d1461317940d57924b9e1093fa0e19e1'
            '9565f9bd92fdad408bd1cbcc70441894579e74f7f3d98c231dc8e4c9aa903c397f9d8f7601714bc7b9f7edb4a5e900583e3b15717055746d3c0260a2da7bfcfc'
            'cf23d84dbb16ed5d91995e0d579b0f60f0839a0935812c194207168fd88a52da698db95c0fd588fe52b73ffa45751d512640b810a1f7b18d567e5fbab8f9b37e'
            '7eb1be54038c84b56d4461f4cad307eb71f1059cc5d0fdd1a77294f399a5a62d44510e152bb65273cff8e235c7f0bf12f2df0d5b787fa5acdea7b4bc477ac3f8'
            'f309d639a465b4273292216c796c2aab8c78df1b8f8951ebe2fd9eb0de99fd640170e97c923eaed7df92aa8a7d7aff6a826cacd374f54f088a2e774aaf58d505'
            '502192b5ce94c8254205f5ddb85bf50c5f1e78c768817b10dca3a7716a8c59d5e093842631acb51e3805cbf85522e0a9200942656f11bbb4ea1b7d61e24ddd78'
            '76322671e09bf8b1aeeb5d72ae11e276a985dd89940c78343974134564ca79e8e389f119c33c298781e6cedb7ec448bf60b3f8c509252f3013ed3b667c668f4a'
            'c7dacda2146e3bc101e77434bfa5ec762fa738b7599db500685761e20906bd196652dbd3d75de08b5d46294a3e7526cfd3edc36b3e3855984a4c555911ca9530'
            'f6cc73665598c907505136a3964cf9d446daa3c546b1b1ed3c348205e4e8a8e868c93c9db1c96a8efd640f166fdb619ef985483d1316423740479723034d0e61'
            '04923e49072623343d3b7da68ab869aa4256563c1713a07cc83535cddbe800d01c6074d392c5559522e42cf4312e59648ea62b012e9ab491b4be10388b849682'
            '16c83b2058e0c7162a8ff0d488fba08bc0ef4b0ad042877c6154a0e6c03268b68f8037cea8794c270f58eae8bfa7e10b97e27d2bfa2c0d5524d5f3f1dc37499a'
            'c27ec190000594f764ca2845e779c7341f3e33bd175a16b53633c86d9155f6cb1cc95cdfead3ac1d48ee370ca342c452c54284ab4cf16712aa055b23322638ae'
            '4233ef2efbac9541331cc277028a93489658659d3f9eba67693497e3b4d9fd3a7c570b426b4623de3e05d5f7160a8bf88ff36e345531d611d7eee30987e6e4fe'
            'f2e3bb6be690b37cf241ef13bf6b9fdb2cc98e0d64cc32d3902ef2798169bf49006ce0e06ec24aa3d3868653a9ab6afd1fe3f4065ac5b65c87c6d770a1119e42'
            'c7068c07725ae38985723893045804b9e8b1c356bea77cc190df9f329c68110ab26fe50c91c587823d9a782aadf868724e913909708c478ce197088043b1c0d1'
            '57ae61e2d4c3a202715366965e2d4d32cba3c781ba9fa04d5507fb96b94b2094d017598e7b20f993f7d8ff933c08a1f05d7e9f4eb2bf96fdbce32c69f7b2618a'
            '2f893c8c396ad108d7641fa2fd17b835b8d993b93065a78c3f2542ffabefae3a5f2d03c81e36d8cf2a5430b29b0b09512fbcbe0fd51f78bc555eeedd7f5c855b'
            '2e256d9ae1fe1e33b1d99650dda965b36256eea6c08159542fae0c7da4806c8d8a485a0c88e8a548cce53041a044c2477dcd6d72910fd35efd88c9a8d5e6d7e4'
            '76f5f08aff26c7bedd9dc0e60279b915b60878a9c8a5728400e45c11788b74b64261ca59057182adc560148a7e975110ce1e103168112014d78e57388395b4ea')


