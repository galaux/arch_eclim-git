# Maintainer: Guillaume ALAUX <guillaume at alaux dot net>
# Contributor: Andrea Fagiani <andfagiani_at_gmail_dot_com>

pkgname=eclim-git
pkgver=2.4.1.r0.g1de73d8
pkgrel=1
pkgdesc='Provides the ability to integrate Eclipse code editing features into your favorite editor'
url='http://eclim.org/'
license=('GPL3')
arch=('i686' 'x86_64')
depends=('vim' 'eclipse')
makedepends=('apache-ant' 'python2-sphinx')
optdepends=('eclipse-pdt: Eclipse PHP Development Tools support'
            'eclipse-cdt: Eclipse C/C++ Plugin support'
            'eclipse-dltk-core: Eclipse Dynamic Languagues Toolkit support'
            'eclipse-dltk-ruby: Eclipse Ruby support'
            'eclipse-wtp: Eclipse Web Developer Tools support')
conflicts=('eclim')
install=${pkgname}.install
_authorgit=https://github.com/ervandew
source=("${pkgname}::git+${_authorgit}/eclim.git"
        systemd_eclimd.service)
sha256sums=('SKIP'
            'eed00c20b596d9ede3d1417826ce3c8477116f029d80e985ec6d3df868008e0b')

pkgver() {
  cd "${srcdir}/${pkgname}"
  ( set -o pipefail
    git describe --long --tags 2>/dev/null | sed 's/\([^-]*-g\)/r\1/;s/-/./g' ||
    printf "r%s.%s" "$(git rev-list --count HEAD)" "$(git rev-parse --short HEAD)"
  )
}

prepare() {
  cd "${srcdir}/${pkgname}"

  sed -i 's|git@github.com:|https://github.com/|' .gitmodules
  git submodule update --init

  # fix build, thanks to mikezackles
  sed -e "s/'sphinx-build'/'sphinx-build2'/g" \
    -e 's|${user.home}/\.|${vim.files}/|g' \
    -e "s|File(getVariable('eclipse')|File('/usr/share/eclipse/'|g" \
    -i ant/build.gant
}

getEclipseUserHome() {
  _eclipse_ver=$(cat /usr/share/eclipse/.eclipseproduct \
    | grep 'version=' \
    | sed -r 's/^version=([0-9\.]+)$/\1/')
  find ~/.eclipse -mindepth 1 -maxdepth 1 -type d -name "org.eclipse.platform_${_eclipse_ver}*" | head -1
}
_eclipse_user_home=$(getEclipseUserHome)
_ant_opts=''
if [ "x${_eclipse_user_home}" != "x" ]; then
  _ant_opts="-Declipse.local=${_eclipse_user_home}"
fi

build() {
  cd "${srcdir}/${pkgname}"

  # recompiling nailgun to make sure the executable is compliant with our architecture
  pushd org.eclim/nailgun
  ./configure
  make

  popd

  ant -Declipse.home=/usr/share/eclipse \
      -Declipse.local=$(getEclipseUserHome) \
      build
}

package() {
  cd "${srcdir}/${pkgname}"

  ant -Declipse.home=/usr/share/eclipse \
      -Declipse.local=$(getEclipseUserHome) \
      docs
  mkdir -p ${pkgdir}/usr/share/doc
  cp -r build/doc/site ${pkgdir}/usr/share/doc/eclim

  mkdir -p ${pkgdir}/usr/share/vim/vimfiles/eclim/doc
  ant -Declipse.home=/usr/share/eclipse \
      -Declipse.local=$(getEclipseUserHome) \
      -Dvim.files=${pkgdir}/usr/share/vim/vimfiles \
      vimdocs

  mkdir -p ${pkgdir}/usr/share/eclipse
  mkdir -p ${pkgdir}/usr/share/vim/vimfiles
  ant -Declipse.home=${pkgdir}/usr/share/eclipse \
      -Declipse.local=$(getEclipseUserHome) \
      -Declipse.dest=${pkgdir}/usr/share/eclipse \
      -Dvim.files=${pkgdir}/usr/share/vim/vimfiles \
      deploy

  # TODO DO we still need these?
  # fix eclim paths
  sed -e "s|${pkgdir}||g" \
    -i ${pkgdir}/usr/share/vim/vimfiles/eclim/plugin/eclim.vim \
    -i ${pkgdir}/usr/share/eclipse/plugins/org.eclim_*/bin/eclimd \
    -i ${pkgdir}/usr/share/eclipse/plugins/org.eclim_*/plugin.properties

  pushd ${pkgdir}/usr/share/eclipse/
    ln -s $(find . -type f -path *bin/eclimd -executable) eclimd
    ln -s $(find . -type f -path *bin/eclim -executable) eclim
  popd

  # delete doctrees
  rm -fr ${pkgdir}/usr/share/doc/eclim/.doctrees

  # delete Windows stuff
  for i in $(find ${pkgdir} -regex ".*bat\|.*cmd\|.*exe"); do  rm -f $i ; done

  rm ${pkgdir}/usr/share/eclipse/plugins/org.eclim_*/nailgun/config.status

  install -D -m 644 ${srcdir}/systemd_eclimd.service ${pkgdir}/usr/lib/systemd/user/eclimd.service
}
