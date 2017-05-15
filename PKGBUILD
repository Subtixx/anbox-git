# Maintainer: Iwan Timmer <irtimmer@gmail.com>

pkgname=('anbox-git' 'anbox-modules-dkms-git')
_pkgname=anbox
pkgver=r559.52f4a7e
pkgrel=1
epoch=1
arch=('x86_64')
url="http://anbox.io/"
license=('GPL3')
makedepends=('cmake' 'git' 'glm' 'dbus-cpp' 'lxc' 'sdl2_image' 'protobuf' 'boost' 'properties-cpp' 'gtest')
source=("git+https://github.com/anbox/anbox.git"
	'anbox-container-manager.service'
	'anbox-session-manager.service'
	'99-anbox.rules'
	'anbox.conf'
	'anbox.desktop'
	'anbox-bridge.network'
	'anbox-bridge.netdev')
sha256sums=('SKIP'
            '49aa34a582de04540a01754976db89f2c05d6170f7192fec0ff14e23d14320d2'
            '1f22dbb5a3ca6925bbf62899cd0f0bbaa0b77c879adcdd12ff9d43adfa61b1d8'
            '210eb93342228168f7bb632c8b93d9bfda6f53f62459a6b74987fa1e17530475'
            '3e07dc524a827c1651857cce28a06c1565bc5188101c140ed213bbafedc5abff'
            '7332d09865be553a259a53819cebddd21f661c7a251d78c2f46acd75c66676b6'
            '44899328725667041e6e84912da81c1d0147b708006eb2c2bb6503f271629ff0'
            '559190df4d6d595480b30d8b13b862081fc4aac52790e33eb24cf7fbcb8003b8')

pkgver() {
  cd "$srcdir/$_pkgname"
  ( set -o pipefail
    git describe --long 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
  )
}

prepare() {
  cd "$srcdir/${_pkgname}"

  # Don't build tests
  truncate -s 0 cmake/FindGMock.cmake
  truncate -s 0 tests/CMakeLists.txt

  # Fix loading translators
  sed -i 's/${CMAKE_INSTALL_PREFIX}\/${ANBOX_TRANSLATOR_INSTALL_DIR}/${ANBOX_TRANSLATOR_INSTALL_DIR}/' CMakeLists.txt
}

build() {
  mkdir -p "$srcdir/${_pkgname}/build"
  cd "$srcdir/${_pkgname}/build"

  cmake .. -DCMAKE_INSTALL_LIBDIR=/usr/lib -DCMAKE_INSTALL_PREFIX=/usr -DCMAKE_BUILD_TYPE=Release
  make
}

package_anbox-git() {
  depends=('dbus-cpp' 'lxc' 'sdl2_image' 'protobuf' 'anbox-image')
  optdepends=('anbox-modules-dkms-git: Required Android kernel modules')
  pkgdesc="Running Android in a container"

  cd "$srcdir/${_pkgname}"
  make -C build DESTDIR="$pkgdir" install

  install -Dm 644 -t $pkgdir/usr/lib/systemd/system $srcdir/anbox-container-manager.service
  install -Dm 644 -t $pkgdir/usr/lib/systemd/user $srcdir/anbox-session-manager.service
  install -Dm 644 $srcdir/anbox-bridge.network $pkgdir/usr/lib/systemd/network/80-anbox-bridge.network
  install -Dm 644 $srcdir/anbox-bridge.netdev $pkgdir/usr/lib/systemd/network/80-anbox-bridge.netdev
  install -Dm 644 -t $pkgdir/usr/lib/udev/rules.d $srcdir/99-anbox.rules
  install -Dm 644 -t $pkgdir/usr/share/applications $srcdir/anbox.desktop
  install -Dm 644 snap/gui/icon.png $pkgdir/usr/share/pixmaps/anbox.png
}

package_anbox-modules-dkms-git() {
  pkgdesc="Required kernel module sources for Android"
  depends=('dkms')

  cd "$srcdir/${_pkgname}"
  modules=(ashmem binder)
  for mod in "${modules[@]}"; do
    install -dm 755 $pkgdir/usr/src
    cp -a kernel/$mod $pkgdir/usr/src/anbox-modules-$mod-$pkgver
  done;

  install -Dm 644 -t $pkgdir/etc/modules-load.d $srcdir/anbox.conf
}
