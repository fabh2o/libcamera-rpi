# Maintainer: David Runge <dvzrv@archlinux.org>

pkgbase=libcamera
pkgname=(
  libcamera
  libcamera-docs
  libcamera-ipa
  libcamera-tools
  gst-plugin-libcamera
)
pkgver=0.2.0
pkgrel=1
pkgdesc="A complex camera support library for Linux, Android, and ChromeOS"
arch=(x86_64)
url="https://libcamera.org/"
_url=https://git.libcamera.org/libcamera/libcamera.git
makedepends=(
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
  python-jinja
  python-ply
  python-sphinx
  python-pyyaml
  qt5-base
  qt5-tools
  sdl2
  systemd
  texlive-core
)
source=(
  "git+$_url#tag=v$pkgver"
)
sha512sums=('5cd700a500bb79b09a435b0d95c8246d93e971725177314f043fc0cdc3517e44ad21b5d14ce690a251ac9762a5691a3b85c0655211897b76704166bcdf5d2f0a')
b2sums=('61098ac3c98ce166e3ad12c65d9b01a4c705ec6a3a1e4b942dfc2bf577616079de82b6ea266ef6c3faf239103d44e911691892908534810e42dec5bff9b360e2')

_pick() {
  local p="$1" f d; shift
  for f; do
    d="$srcdir/$p/${f#$pkgdir/}"
    mkdir -p "$(dirname "$d")"
    mv "$f" "$d"
    rmdir -p --ignore-fail-on-non-empty "$(dirname "$f")"
  done
}

pkgver() {
  cd $pkgbase
  git describe --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g;s/v//g'
}

prepare() {
  cd $pkgbase

  # add version, so that utils/gen-version.sh may rely on it
  printf "%s\n" "$pkgver" > .tarball-version
}

build() {
  local meson_options=(
    -D v4l2=true
    -D tracing=disabled
    -D test=true
  )

  arch-meson $pkgbase build "${meson_options[@]}"
  meson compile -C build
}

check() {
  meson test -C build || echo "Tests require CLONE_NEWUSER/ CLONE_NEWNET."
}

package_libcamera() {
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
    libcamera-ipa
    libelf
    libunwind
    libyaml
    sh
    systemd-libs libudev.so
  )
  optdepends=(
    'gst-plugin-libcamera: GStreamer plugin'
    'libcamera-docs: for documentation'
    'libcamera-tools: for applications'
  )
  provides=(libcamera.so libcamera-base.so)

  meson install -C build --destdir "$pkgdir"
  install -vDm 644 $pkgbase/LICENSES/{BSD-3-Clause,Linux-syscall-note,MIT}.txt -t "$pkgdir/usr/share/licenses/$pkgname/"

  (
    cd "$pkgdir"
    _pick $pkgbase-docs usr/share/doc
    _pick $pkgbase-ipa usr/lib/libcamera/
    _pick $pkgbase-tools usr/bin/{cam,qcam,lc-compliance}
    _pick gst-plugin-$pkgbase usr/lib/gstreamer-*
  )
}

package_libcamera-docs() {
  pkgdesc+=" - documentation"
  license=(
    CC-BY-4.0
    CC-BY-SA-4.0
    CC0-1.0
  )

  mv -v $pkgname/* "$pkgdir"
  mv -v "$pkgdir/usr/share/doc/$pkgbase-$pkgver/" "$pkgdir/usr/share/doc/$pkgbase/"
  rm -frv "$pkgdir/usr/share/doc/$pkgbase/html/.buildinfo"
}

package_libcamera-ipa() {
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
    libcamera libcamera.so libcamera-base.so
  )
  # stripping requires re-signing of IPA libs, so we do it manually
  options=(!strip)

  strip $pkgname/usr/lib/libcamera/*{.so,proxy}
  for _lib in $pkgname/usr/lib/libcamera/*.so; do
    $pkgbase/src/ipa/ipa-sign.sh "$(find build -type f -iname "*ipa-priv-key.pem")" "$_lib" "$_lib.sign"
  done
  mv -v $pkgname/* "$pkgdir"
  install -vDm 644 $pkgbase/LICENSES/BSD-2-Clause.txt -t "$pkgdir/usr/share/licenses/$pkgname/"
}

package_libcamera-tools() {
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
    libcamera libcamera.so libcamera-base.so
    libdrm
    libevent libevent-2.1.so libevent_pthreads-2.1.so
    libjpeg-turbo libjpeg.so
    libtiff libtiff.so
    libyaml
    qt5-base
    sdl2
  )
  conflicts=("$pkgbase-tests<0.0.1-2")
  replaces=("$pkgbase-tests<0.0.1-2")

  mv -v $pkgname/* "$pkgdir"
  install -vDm 644 $pkgbase/LICENSES/BSD-2-Clause.txt -t "$pkgdir/usr/share/licenses/$pkgname/"
}

package_gst-plugin-libcamera() {
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
    libcamera libcamera.so libcamera-base.so
  )

  mv -v $pkgname/* "$pkgdir"
}
