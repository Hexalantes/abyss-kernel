# Maintainer: Hexalantes <akadarpadpadak@hotmail.com>

pkgbase=linux-abyss
pkgver=6.16.0.abyss1
pkgrel=1
pkgdesc="Linux Abyss kernel (based on zen)"
arch=(x86_64)
url="https://github.com/Hexalantes/abyss-kernel"
license=(GPL2)
makedepends=(
  bc cpio gettext git libelf pahole perl tar xz
)
options=('!strip')

_srcname=zen-kernel
_srctag=v${pkgver%.*}-${pkgver##*.}
source=(
  "$_srcname::git+https://github.com/zen-kernel/zen-kernel#branch=master"
  config
)
validpgpkeys=()
b2sums=('SKIP' 'SKIP')

export KBUILD_BUILD_HOST=abyss
export KBUILD_BUILD_USER=$pkgbase
export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

_make() {
  test -s version
  make KERNELRELEASE="$(<version)" "$@"
}

prepare() {
  cd $_srcname

  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname
  make defconfig
  make -s kernelrelease > version
  make mrproper

  cp ../config .config
  _make olddefconfig
}

build() {
  cd $_srcname
  _make all
}

_package() {
  pkgdesc="The Linux Abyss kernel and modules"
  depends=(kmod)
  optdepends=(
    'wireless-regdb: for setting correct wireless channels'
    'linux-firmware: for firmware loading support'
  )
  provides=(KSMBD-MODULE UKSMD-BUILTIN VHBA-MODULE VIRTUALBOX-GUEST-MODULES WIREGUARD-MODULE)
  cd $_srcname

  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"
  install -Dm644 "$(_make -s image_name)" "$modulesdir/vmlinuz"
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"
  install -Dm644 /dev/null "$pkgdir/etc/mkinitcpio.d/$pkgbase.preset"
  _make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 modules_install
  depmod -b "$pkgdir/usr" "$(cat version)"

  rm -f "$modulesdir"/{source,build}
}

_package-headers() {
  pkgdesc="Header files for building modules for the Linux Abyss kernel"
  depends=(pahole)
  cd $_srcname

  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/x86" -m644 arch/x86/Makefile
  cp -t "$builddir" -a scripts

  install -Dt "$builddir/tools/objtool" tools/objtool/objtool
#  install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids

  cp -t "$builddir" -a include
  cp -t "$builddir/arch/x86" -a arch/x86/include
  install -Dt "$builddir/arch/x86/kernel" -m644 arch/x86/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  find "$builddir"/arch/*/ -maxdepth 0 -type d ! -name x86 -exec rm -r {} +
  rm -r "$builddir/Documentation"
  find -L "$builddir" -type l -delete
  find "$builddir" -type f -name '*.o' -delete

  local file
  while read -rd '' file; do
    case "$(file -Sib "$file")" in
      application/x-sharedlib\;*)        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)          strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)       strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*)   strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  strip -v $STRIP_STATIC "$builddir/vmlinux"

  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=(
  "$pkgbase"
  "$pkgbase-headers"
)
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=2 sw=2 et:
