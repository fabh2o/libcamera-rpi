# Maintainer: Fabrice Monasterio <arch@fabricemonasterio.dev>
# From the archlinux package libcamera maintained by David Runge <dvzrv@archlinux.org>
# and from http://git.intranifty.net/pkg/libcamera-rpi/ by Ingar <ingar@telenet.be>

pkgbase=libcamera-rpi
_pkgbase=libcamera
pkgname=(
  libcamera-rpi
  libcamera-rpi-docs
  libcamera-rpi-ipa
  libcamera-rpi-tools
  gst-plugin-libcamera-rpi
  python-libcamera-rpi
)
pkgver=0.3.2+rpt20240927
_pkgver=0.3.2
pkgrel=1
pkgdesc="A complex camera support library for Linux, Android, and ChromeOS. RaspberryPi Fork."
arch=(x86_64 i686 aarch64 armv7h)
url="https://github.com/raspberrypi/libcamera.git"
makedepends=(
  boost
  doxygen
  git
  glib2
  graphviz
  gst-plugins-base
  gtest
  libdrm
  libjpeg-turbo
  libtiff
  libyaml
  meson
  pybind11
  python-jinja
  python-ply
  python-sphinx
  python-pyyaml
  qt6-base
  qt6-tools
  sdl2
  systemd
  texlive-core
  texlive-latex
)
# "git+$url#tag=v$pkgver"
source=(
  "https://github.com/raspberrypi/libcamera/releases/download/v${pkgver}/libcamera-${pkgver}.tar.xz"
  "libcamera-rpi-0.3.2-arch.patch"
)
# makepkg -g
sha256sums=(
  '4d6502a2371204f3a54955f4a25f806be88fcc469167f655afbe9b398f827392'
  '5c8f66b75a3d470101bb69c572de93e1d14e29c614dcb0391f3ea526781c260e'
)

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

prepare() {
  mv $_pkgbase-$_pkgver $pkgbase
  cd $pkgbase

  patch -Np1 -i ../libcamera-rpi-0.3.2-arch.patch

  # add version, so that utils/gen-version.sh may rely on it
  printf "%s\n" "$pkgver" > .tarball-version
}

build() {
  local meson_options=(
    --buildtype=release
    --wrap-mode default            # to build libisp
    -D pipelines=rpi/vc4,rpi/pisp  # hardware acceleration and rpi5 image signal processor
    -D ipas=rpi/vc4,rpi/pisp       # https://libcamera.org/guides/ipa.html
    -D pycamera=enabled            # Enable libcamera Python bindings
    -D v4l2=true                   # Compile the V4L2 compatibility layer
    -D tracing=disabled
    -D test=true                   # Compile and include the tests
  )

  arch-meson $pkgbase build "${meson_options[@]}"
  meson compile -C build
}

check() {
  meson test -C build || echo "Tests require CLONE_NEWUSER/ CLONE_NEWNET."
}

package_libcamera-rpi() {
  license=(
    Apache-2.0
    CC0-1.0
    'GPL-2.0-only WITH Linux-syscall-note'
    GPL-2.0-or-later
    LGPL-2.1-or-later
    'GPL-2.0-or-later WITH Linux-syscall-note OR BSD-3-Clause'
    'GPL-2.0-or-later WITH Linux-syscall-note OR MIT'
  )
  depends=(
    gcc-libs
    glibc
    gnutls
    libcamera-rpi-ipa
    libelf
    libunwind
    libyaml
    sh
    systemd-libs libudev.so
  )
  optdepends=(
    'gst-plugin-libcamera-rpi: GStreamer plugin'
    'libcamera-rpi-docs: for documentation'
    'libcamera-rpi-tools: for applications'
  )
  provides=(libcamera.so libcamera-base.so)
  conflicts=(libcamera)

  meson install -C build --destdir "$pkgdir"
  install -vDm 644 $pkgbase/LICENSES/{BSD-3-Clause,Linux-syscall-note,MIT}.txt -t "$pkgdir/usr/share/licenses/$pkgname/"

  (
    cd "$pkgdir"
    _pick $pkgbase-docs usr/share/doc
    _pick $pkgbase-ipa usr/lib/libcamera/
    _pick $pkgbase-tools usr/bin/{cam,qcam,lc-compliance}
    _pick gst-plugin-$pkgbase usr/lib/gstreamer-*
    _pick python-$pkgbase usr/lib/python*
  )
}

package_libcamera-rpi-docs() {
  pkgdesc+=" - documentation"
  license=(
    CC-BY-4.0
    CC-BY-SA-4.0
    CC0-1.0
  )

  mv -v $pkgname/* "$pkgdir"
  mv -v "$pkgdir/usr/share/doc/$_pkgbase-$_pkgver/" "$pkgdir/usr/share/doc/$pkgbase/"
  rm -frv "$pkgdir/usr/share/doc/$pkgbase/html/.buildinfo"
}

package_libcamera-rpi-ipa() {
  pkgdesc+=" - signed IPA"
  license=(
    BSD-2-Clause
    CC-BY-SA-4.0
    CC0-1.0
    GPL-2.0-or-later
    LGPL-2.1-or-later
  )
  depends=(
    gcc-libs
    glibc
    libcamera-rpi libcamera.so libcamera-base.so
  )
  conflicts=("libcamera-ipa")
  # stripping requires re-signing of IPA libs, so we do it manually
  options=(!strip)

  strip $pkgname/usr/lib/libcamera/*{.so,proxy}
  for _lib in $pkgname/usr/lib/libcamera/*.so; do
    $pkgbase/src/ipa/ipa-sign.sh "$(find build -type f -iname "*ipa-priv-key.pem")" "$_lib" "$_lib.sign"
  done
  mv -v $pkgname/* "$pkgdir"
  install -vDm 644 $pkgbase/LICENSES/BSD-2-Clause.txt -t "$pkgdir/usr/share/licenses/$pkgname/"
}

package_libcamera-rpi-tools() {
  pkgdesc+=" - tools"
  license=(
    BSD-2-Clause
    CC0-1.0
    GPL-2.0-or-later
    LGPL-2.1-or-later
  )
  depends=(
    gcc-libs
    glibc
    gtest
    libcamera-rpi libcamera.so libcamera-base.so
    libdrm
    libevent libevent-2.1.so libevent_pthreads-2.1.so
    libjpeg-turbo libjpeg.so
    libtiff libtiff.so
    libyaml
    qt6-base
    sdl2
  )
  conflicts=("libcamera-tools")

  mv -v $pkgname/* "$pkgdir"
  install -vDm 644 $pkgbase/LICENSES/BSD-2-Clause.txt -t "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gst-plugin-libcamera-rpi() {
  pkgdesc="Multimedia graph framework - libcamera plugin"
  license=(
    CC0-1.0
    LGPL-2.1-or-later
  )
  depends=(
    gcc-libs
    glibc
    glib2 libg{lib,object}-2.0.so
    gstreamer
    gst-plugins-base-libs
    libcamera-rpi libcamera.so libcamera-base.so
  )
  conflicts=("gst-plugin-libcamera")

  mv -v $pkgname/* "$pkgdir"
}

package_python-libcamera-rpi() {
  pkgdesc+=" - Python integration"
  license=(
    CC0-1.0
    LGPL-2.1-or-later
  )
  depends=(
    gcc-libs
    glibc
    libcamera-rpi
    python
  )
  conflicts=("python-libcamera")

  mv -v $pkgname/* "$pkgdir"
}
