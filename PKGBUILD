# Maintainer: Dave Higham <pepedog@archlinuxarm.org>
# Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>
# Maintainer: Oleg Rakhmanov <oleg@archlinuxarm.org>

buildarch=20

pkgbase=linux-raspberrypi-dsd
_commit=7e441f636a07575e77bfe5760905a3f601701f50
_srcname=linux-${_commit}
_kernelname=${pkgbase#linux}
_desc="Raspberry Pi with native DSD support"
pkgver=4.9.60
pkgrel=2
arch=('armv6h' 'armv7h')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git')
options=('!strip')
source=("https://github.com/raspberrypi/linux/archive/${_commit}.tar.gz"
        "https://github.com/allocom/linux/commit/b67b6b46e6d5326fb71dac73a112cbf0dafe9c6b.patch"
        "https://github.com/allocom/linux/commit/91bad8f4e89e7a52c7a88ca9ca2cf6375541babe.patch"
        "https://github.com/allocom/linux/commit/4b46ccea40f9839d13904573e03bc7caee97c53c.patch"
        "https://github.com/Fourdee/linux/commit/70a8155a64fc3fde57f69f91da3b2835823e0061.patch"
	"https://github.com/raspberrypi/linux/pull/1460/commits/153a4f22c623d19b69094844cfab4bf5e3f57d22.patch"
	"https://github.com/raspberrypi/linux/pull/1460/commits/43bb912c8b739d4d5f0f5df9c11f9d69f12dcbdf.patch"
	"https://github.com/raspberrypi/linux/pull/1460/commits/45fbb1dc957953f6871ef4679f998cb6125ae32c.patch"
        'https://archlinuxarm.org/builder/src/brcmfmac43430-sdio.bin' 'https://archlinuxarm.org/builder/src/brcmfmac43430-sdio.txt'
        'config.txt'
        'cmdline.txt'
        'config'
        'linux.preset'
        '99-linux.hook'
        'usb-dsd-quirks.patch'
	'i-sabre-k2m.patch'
	'kernel-alsa-support-for-384khz-sample-rates.patch')
md5sums=('34529bb084f1ea734b714298d2d46ada'
         '93164ce4a1a158e4edf63efb365b1d26'
         '81a1b2f0190bb8487eb42bf8cbe4814f'
         '9263b8b7480cb91a30eb4a61b45701f1'
         'f9345b6513beccc487179eb705b6f0cf'
         '36d73c8927a59c4337c310f7ece4c27e'
         '8f26c727f43df19abeccf7e682554a49'
         '12585e3a94fcdf2c7f7be97a6c522086'
         '4a410ab9a1eefe82e158d36df02b3589'
         '8c3cb6d8f0609b43f09d083b4006ec5a'
         '7c6b37a1353caccf6d3786bb4161c218'
         'fcd90122a2621d0a7d6cdd020da8723d'
         'a7e60c95322f367de2b0bc9add97e3c9'
         '552c43bf6c0225bc213b31ee942b7000'
         '982f9184dfcfbe52110795cf73674334'
         'b72c7042830bbe2f051bc481f2ebe909'
         '917614857efdf6dcce07c09db8d2084e'
         'c91808bf64a135bd93ebdd3d8c5b1575')

prepare() {
  cd "${srcdir}/${_srcname}"

  cat "${srcdir}/config" > ./.config

  msg2 "patching: eliminate noise at the start of DSD"
  patch -Np1 -i ../70a8155a64fc3fde57f69f91da3b2835823e0061.patch

#  msg2 "patching: support for 384kHz"
#  patch -Np1 -i ../d896e4fa1f8ebcf182fb13aa8ceae38ba6b07c9c.patch

#  msg2 "patching: latest stuff from Allo"
#  patch -Np1 -i ../4b46ccea40f9839d13904573e03bc7caee97c53c.patch
#  patch -Np1 -i ../91bad8f4e89e7a52c7a88ca9ca2cf6375541babe.patch
#  patch -Np1 -i ../b67b6b46e6d5326fb71dac73a112cbf0dafe9c6b.patch

  msg2 "patching: 384k support I2S"
  patch -Np1 -i ../153a4f22c623d19b69094844cfab4bf5e3f57d22.patch
  patch -Np1 -i ../43bb912c8b739d4d5f0f5df9c11f9d69f12dcbdf.patch
#  patch -Np1 -i ../45fbb1dc957953f6871ef4679f998cb6125ae32c.patch
  patch -Np1 -i ../kernel-alsa-support-for-384khz-sample-rates.patch

  msg2 "patching: kernel native DSD quirks"
  patch -Np1 -i ../usb-dsd-quirks.patch

  msg2 "patching: add support for Audiophonics I-Sabre K2M"
  patch -Np1 -i ../i-sabre-k2m.patch

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

  mkdir firmware/brcm
  cp ../brcmfmac43430-sdio.{bin,txt} firmware/brcm
}

build() {
  cd "${srcdir}/${_srcname}"

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  #make bcmrpi_defconfig # using RPi defconfig
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #cp ./.config ../${pkgver}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  make ${MAKEFLAGS} zImage modules dtbs
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${pkgver}")
  conflicts=('kernel26' 'linux')
  install=${pkgname}.install
  backup=('boot/config.txt' 'boot/cmdline.txt')
  replaces=('linux-raspberrypi-latest')

  cd "${srcdir}/${_srcname}"

  KARCH=arm

  # get kernel version
  _kernver="$(make kernelrelease)"

  mkdir -p "${pkgdir}"/{lib/modules,lib/firmware}
  make INSTALL_MOD_PATH="${pkgdir}" modules_install
  make INSTALL_DTBS_PATH="${pkgdir}/boot" dtbs_install

  [[ $CARCH == "armv6h" ]] && perl scripts/mkknlimg --dtok arch/$KARCH/boot/zImage "${pkgdir}/boot/kernel.img" \
                           && rm -f "${pkgdir}"/boot/bcm{2836,2709,2710}*.dtb
  [[ $CARCH == "armv7h" ]] && perl scripts/mkknlimg --dtok arch/$KARCH/boot/zImage "${pkgdir}/boot/kernel7.img" \
                           && rm -f "${pkgdir}"/boot/bcm{2835,2708}*.dtb
  cp arch/$KARCH/boot/dts/overlays/README "${pkgdir}/boot/overlays"

  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/g" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i "${startdir}/${pkgname}.install"

  # install mkinitcpio preset file for kernel
  install -D -m644 "${srcdir}/linux.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
  sed \
    -e "1s|'linux.*'|'${pkgbase}'|" \
    -e "s|ALL_kver=.*|ALL_kver=\"${_kernver}\"|" \
    -i "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hook for initramfs regeneration
  sed "s|%PKGBASE%|${pkgbase}|g" "${srcdir}/99-linux.hook" |
    install -D -m644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/99-${pkgbase}.hook"

  # remove build and source links
  rm -f "${pkgdir}"/lib/modules/${_kernver}/{source,build}
  # remove the firmware
  rm -rf "${pkgdir}/lib/firmware"
  # make room for external modules
  ln -s "../extramodules-${_basekernel}${_kernelname:--ARCH}" "${pkgdir}/lib/modules/${_kernver}/extramodules"
  # add real version for building modules and running depmod from post_install/upgrade
  mkdir -p "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}"
  echo "${_kernver}" > "${pkgdir}/lib/modules/extramodules-${_basekernel}${_kernelname:--ARCH}/version"

  # Now we call depmod...
  depmod -b "$pkgdir" -F System.map "$_kernver"

  # move module tree /lib -> /usr/lib
  mkdir -p "${pkgdir}/usr"
  mv "$pkgdir/lib" "$pkgdir/usr"

  # install boot files
  install -m644 ../config.txt ../cmdline.txt "${pkgdir}/boot"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')
  replaces=('linux-raspberrypi-latest-headers')

  install -dm755 "${pkgdir}/usr/lib/modules/${_kernver}"

  cd "${srcdir}/${_srcname}"
  install -D -m644 Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/Makefile"
  install -D -m644 kernel/Makefile \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/kernel/Makefile"
  install -D -m644 .config \
    "${pkgdir}/usr/lib/modules/${_kernver}/build/.config"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/include"

  for i in acpi asm-generic config crypto drm generated keys linux math-emu \
    media net pcmcia scsi soc sound trace uapi video xen; do
    cp -a include/${i} "${pkgdir}/usr/lib/modules/${_kernver}/build/include/"
  done

  # copy arch includes for external modules
  mkdir -p ${pkgdir}/usr/lib/modules/${_kernver}/build/arch/$KARCH
  cp -a arch/$KARCH/include ${pkgdir}/usr/lib/modules/${_kernver}/build/arch/$KARCH/

  # copy files necessary for later builds, like nvidia and vmware
  cp Module.symvers "${pkgdir}/usr/lib/modules/${_kernver}/build"
  cp -a scripts "${pkgdir}/usr/lib/modules/${_kernver}/build"

  # fix permissions on scripts dir
  chmod og-w -R "${pkgdir}/usr/lib/modules/${_kernver}/build/scripts"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/.tmp_versions"

  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel"

  cp arch/${KARCH}/Makefile "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/"

  cp arch/${KARCH}/kernel/asm-offsets.s "${pkgdir}/usr/lib/modules/${_kernver}/build/arch/${KARCH}/kernel/"

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
  cp include/config/dvb/*.h "${pkgdir}/usr/lib/modules/${_kernver}/build/include/config/dvb/"

  # add dvb headers for http://mcentral.de/hg/~mrec/em28xx-new
  # in reference to:
  # http://bugs.archlinux.org/task/13146
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  cp drivers/media/dvb-frontends/lgdt330x.h "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/dvb-frontends/"
  mkdir -p "${pkgdir}/usr/lib/modules/${_kernver}/build/drivers/media/i2c/"
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

  # copy in Kconfig files
  for i in $(find . -name "Kconfig*"); do
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
  rm -rf "${pkgdir}"/usr/lib/modules/${_kernver}/build/arch/{alpha,arc,arm26,arm64,avr32,blackfin,c6x,cris,frv,h8300,hexagon,ia64,m32r,m68k,m68knommu,metag,mips,microblaze,mn10300,openrisc,parisc,powerpc,ppc,s390,score,sh,sh64,sparc,sparc64,tile,unicore32,um,v850,x86,xtensa}
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
