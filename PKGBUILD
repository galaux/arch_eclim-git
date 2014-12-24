# Maintainer: Andrea Fagiani <andfagiani_at_gmail_dot_com>

pkgname=eclim
pkgver=2.4.0
pkgrel=1
pkgdesc="Brings Eclipse functionality to Vim"
url="http://eclim.org/"
license=('GPL3')
arch=(i686 x86_64)
depends=('vim' 'eclipse')
makedepends=('apache-ant' 'python2-sphinx')
optdepends=('eclipse-pdt: Eclipse PHP Development Tools support'
            'eclipse-cdt: Eclipse C/C++ Plugin support'
            'eclipse-dltk-core: Eclipse Dynamic Languagues Toolkit support'
            'eclipse-dltk-ruby: Eclipse Ruby support'
            'eclipse-wtp: Eclipse Web Developer Tools support')
conflicts=('eclim-git')
install=$pkgname.install
source=("http://downloads.sourceforge.net/project/$pkgname/$pkgname/$pkgver/${pkgname}_$pkgver.tar.gz")
md5sums=('51a3bd0342b360271deb6e5e64a8bbec')

prepare() {
  cd $srcdir/${pkgname}_$pkgver

  # fix build, thanks to mikezackles
  sed -e "s/'sphinx-build'/'sphinx-build2'/g" \
    -e 's|${user.home}/\.|${vim.files}/|g' \
    -e "s|File(getVariable('eclipse')|File('/usr/share/eclipse/'|g" \
    -i ant/build.gant

  # Get the ANT_HOME environment variable
  source /etc/profile.d/apache-ant.sh

  chmod +x org.eclim/nailgun/configure bin/sphinx
}

build() {
  cd $srcdir/${pkgname}_$pkgver

  # recompiling nailgun to make sure the executable is compliant with our architecture
  cd org.eclim/nailgun
  ./configure
  make

  cd ../..

  ant -Declipse.home=/usr/share/eclipse \
      -Dvim.files=/usr/share/vim/vimfiles \
      build
}

package() {
  cd $srcdir/${pkgname}_$pkgver

  mkdir -p $pkgdir/usr/share/eclipse
  mkdir -p $pkgdir/usr/share/vim/vimfiles

  ant -Declipse.home=/usr/share/eclipse \
      -Dvim.files=$pkgdir/usr/share/vim/vimfiles \
      docs vimdocs

  ant -Declipse.home=$pkgdir/usr/share/eclipse \
      -Dvim.files=$pkgdir/usr/share/vim/vimfiles \
      deploy

  # copy eclim docs
  mkdir -p $pkgdir/usr/share/doc/
  cp -r build/doc/site $pkgdir/usr/share/doc/eclim

  # fix eclim paths
  sed -e "s|$pkgdir||g" \
    -i $pkgdir/usr/share/vim/vimfiles/eclim/plugin/eclim.vim \
    -i $pkgdir/usr/share/eclipse/plugins/org.eclim_$pkgver/bin/eclimd \
    -i $pkgdir/usr/share/eclipse/plugins/org.eclim_$pkgver/plugin.properties

  # delete doctrees
  rm -fr $pkgdir/usr/share/doc/eclim/.doctrees

  # delete Windows stuff
  for i in $(find $pkgdir -regex ".*bat\|.*cmd\|.*exe"); do  rm -f $i ; done

  rm $pkgdir/usr/share/eclipse/plugins/org.eclim_${pkgver}/nailgun/config.status
}
