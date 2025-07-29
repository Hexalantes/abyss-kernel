# Maintainer: Hexalantes <akadarpadpadak@hotmail.com>

pkgname=linux-abyss-headers
pkgver=6.15.8.abyss1
pkgrel=1
pkgdesc="Header files for building modules for the Linux Abyss kernel"
arch=(x86_64)
url="https://github.com/zen-kernel/zen-kernel"
license=(GPL2)
depends=(pahole)
source=()
noextract=()
options=('!strip')

build() {
  echo "Skipping build step (kernel already compiled)"
}

package() {
  cd "$srcdir/zen-kernel"  # Kernel kaynakları bu klasörde olacak

  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  install -Dm644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux -t "$builddir"

  cp -a include scripts "$builddir"
  mkdir -p "$builddir/arch/x86"
  cp -a arch/x86/include "$builddir/arch/x86"


  install -Dm755 tools/objtool/objtool "$builddir/tools/objtool/objtool"
# install -Dm755 tools/bpf/resolve_btfids/resolve_btfids "$builddir/tools/bpf/resolve_btfids/resolve_btfids"

  find "$builddir" -type f -name '*.o' -delete
  find "$builddir" -type l -exec rm -f {} +

  strip -v --strip-debug "$builddir/vmlinux" || true

  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/linux-abyss"
}
