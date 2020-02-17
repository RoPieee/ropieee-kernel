# Maintainer: Dave Higham <pepedog@archlinuxarm.org>
# Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>
# Maintainer: Oleg Rakhmanov <oleg@archlinuxarm.org>

buildarch=20

pkgbase=linux-raspberrypi-dsd
_commit=7688b39276ff9952df381d79de63b258e73971ce
_srcname=linux-${_commit}
_kernelname=${pkgbase#linux}
_desc="Raspberry Pi with native DSD support"
# the real_pkgver is the actual kernel version of the package
# normally this should be the same as pkgver, but if we for some reason need to downgrade
# we can do so by using a 'fake' pkgver of a higher kernel than is actually provided by the real_pkgver
pkgver=4.19.80
real_pkgver=4.14.114
pkgrel=5
arch=('armv6h' 'armv7h')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git')
options=('!strip')
source=("https://github.com/raspberrypi/linux/archive/${_commit}.tar.gz"
# 'https://archlinuxarm.org/builder/src/brcmfmac43430-sdio.bin' 'https://archlinuxarm.org/builder/src/brcmfmac43430-sdio.txt'
        'config.txt'
        'cmdline.txt'
        'config'
        'linux.preset'
        '60-linux.hook'
        '90-linux.hook'
	'kernel-usb-native-dsd-generic-detection.patch'
        'kernel-usb-native-dsd-quirks.patch'
	'kernel-alsa-support-for-384khz-sample-rates.patch'
	'kernel-sound-pcm512x-add-support-for-352k8.patch'
	'kernel-sound-pcm5102a-add-support-for-384k.patch'
	'kernel-drivers-net-usb-ax88179_178a.patch'
	'kernel-add-rtl8812au-network-driver.patch'
	'kernel-add-rtl8192eu-network-driver.patch'
	'kernel-add-audiophonics-i-sabre-driver.patch'
	'i-sabre-k2m.patch')
md5sums=('94544ffbcf329b8d6187fb0cbc5a681c'
         '7c6b37a1353caccf6d3786bb4161c218'
         '7c09a9bcb2ad790100fb5e58b125c159'
         '119a4f6f2d073a98b901630958bf9a1b'
         '86d4a35722b5410e3b29fc92dae15d4b'
         'ce6c81ad1ad1f8b333fd6077d47abdaf'
         'ba6ee1d0a4c28fc35748013b4468c3d3'
         '59723235d523b774488ae5a5bf03f7c9'
         'bdec385a1e07aeb41547b4280a712243'
         'ec0778debc64a779fb674aa1231d5a58'
         'defaeb558fb6ef21d55b4cd7a9e1d51f'
         '0c7adc3f558065e2f2343b973830a51e'
         'db65beb304abbeb55e7f4a5744d3f76f'
         '2d7b6bd883af73a8987c58f20c591391'
         '63682c6e30b071c9b998da234a65a25f'
         '136065bb14c06205c1b5837f369e1a82'
         'f5513f39f793597813d0257f0f12195e')

prepare() {
  cd "${srcdir}/${_srcname}"

  cat "${srcdir}/config" > ./.config

  msg2 "patching: 384k support"
  patch -Np1 -i ../kernel-alsa-support-for-384khz-sample-rates.patch
  patch -Np1 -i ../kernel-sound-pcm512x-add-support-for-352k8.patch
  patch -Np1 -i ../kernel-sound-pcm5102a-add-support-for-384k.patch

  msg2 "patching: kernel USB native DSD quirks"
  patch -Np1 -i ../kernel-usb-native-dsd-quirks.patch

  msg2 "patching: kernel driver used by the Allo USBridge Signature"
  patch -Np1 -i ../kernel-drivers-net-usb-ax88179_178a.patch

  msg2 "Patching: add kernel driver RTL 8812AU"
  patch -Np1 -i ../kernel-add-rtl8812au-network-driver.patch

  msg2 "Patching: add kernel driver RTL 8192EU"
  patch -Np1 -i ../kernel-add-rtl8192eu-network-driver.patch

  msg2 "Patching: add kernel driver Audiophonics I-Sabre"
  patch -Np1 -i ../i-sabre-k2m.patch

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh

#  mkdir firmware/brcm
#  cp ../brcmfmac43430-sdio.{bin,txt} firmware/brcm
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
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7' 'firmware-raspberrypi')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=('kernel26' "linux=${real_pkgver}")
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

  [[ $CARCH == "armv6h" ]] && cp arch/$KARCH/boot/zImage "${pkgdir}/boot/kernel.img" \
                           && rm -f "${pkgdir}"/boot/bcm{2836,2709,2710}*.dtb
  [[ $CARCH == "armv7h" ]] && cp arch/$KARCH/boot/zImage "${pkgdir}/boot/kernel7.img" \
                           && rm -f "${pkgdir}"/boot/bcm{2835,2708}*.dtb
  cp arch/$KARCH/boot/dts/overlays/README "${pkgdir}/boot/overlays"

  # set correct depmod command for install
  sed \
    -e  "s/KERNEL_NAME=.*/KERNEL_NAME=${_kernelname}/g" \
    -e  "s/KERNEL_VERSION=.*/KERNEL_VERSION=${_kernver}/g" \
    -i "${startdir}/${pkgname}.install"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${_kernver}|g
    s|%EXTRAMODULES%|${_extramodules}|g
  "

  # install mkinitcpio preset file for kernel
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

#  install -D -m644 "${srcdir}/linux.preset" "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"
#  sed \
#    -e "1s|'linux.*'|'${pkgbase}'|" \
#    -e "s|ALL_kver=.*|ALL_kver=\"${_kernver}\"|" \
#    -i "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # install pacman hooks
  sed "${_subst}" ../60-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/60-${pkgbase}.hook"
  sed "${_subst}" ../90-linux.hook |
    install -Dm644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/90-${pkgbase}.hook"

#  sed "s|%PKGBASE%|${pkgbase}|g" "${srcdir}/99-linux.hook" |
#    install -D -m644 /dev/stdin "${pkgdir}/usr/share/libalpm/hooks/99-${pkgbase}.hook"

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
  /usr/sbin/depmod -b "$pkgdir" -F System.map "$_kernver"

  # move module tree /lib -> /usr/lib
  mkdir -p "${pkgdir}/usr"
  mv "$pkgdir/lib" "$pkgdir/usr"

  # install boot files
  install -m644 ../config.txt ../cmdline.txt "${pkgdir}/boot"
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${real_pkgver}")
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
