# Contributor: hauptmech <hauptmech@gmail.com>
# Contributor: Xyne <xyne@archlinux.ca>
# Maintainer: saxonbeta <saxonbeta at gmail___com
pkgname=elmerfem-svn
pkgver=7093
pkgrel=1
pkgdesc="Simulation tool for CFD, FEM, electromagnetics, heat transfer and others featuring a PDE solver."
arch=('i686' 'x86_64')
url="http://www.csc.fi/english/pages/elmer"
license=('GPL')
depends=('qwt' 'opencascade' 'vtk5' 'suitesparse' 'arpack')
makedepends=('subversion' 'gcc-fortran')
provides=(elmerfem)
conflicts=('elmer_umfpack' 'elmerfem-git')
options=(!ccache !emptydirs)

source=('svn+https://svn.code.sf.net/p/elmerfem/code/trunk/'
	 'qwt1.patch' 'qwt2.patch')
sha256sums=(SKIP 'd34fc127e3409f8ee2b9029305b1869449b4f84964ac4f5065a97c5c6388a43d'
		'8376098f4428a14db9ac78ef679b590315a710e670fdfbcb51787f70613eff54')

prepare(){
  patch  -p0 < qwt1.patch
  patch -p0 --binary < qwt2.patch
}

build() {
  
  cd "$srcdir/trunk"
  export CC=mpicc
  export CXX=mpicxx
  export FC=mpif90
  export F77=mpif90

  export SRC="$srcdir/trunk"
  export CPPFLAGS_ORIG="$CPPFLAGS"
  export LIBS_ORIG="$LIBS"
  export CPPFLAGS="${CPPFLAGS} -I${SRC}/eio/include -I${SRC}/hutiter/include  -I${SRC}/matc/src -I/usr/include -DUSE_INTERP_ERRORLINE -DUSE_INTERP_RESULT"
  export LDFLAGS="${LDFLAGS} -L${SRC}/matc/src -L${SRC}/eio/src -L${SRC}/elmergrid/src/metis -L${SRC}/umfpack/src/amd -L${SRC}/umfpack/src/umfpack -L${SRC}/hutiter/src"

  modules="matc umfpack mathlibs elmerparam elmergrid meshgen2d eio hutiter"
  for m in $modules; do
    msg "compiling ${m}"
    cd $m
    ./configure --prefix=/usr --with-mpi=yes --with-mpi-lib-dir=/usr/lib/openmpi/ 
    make
    cd ..
  done

  #Hack because tool isn't reading flags
  cp hutiter/include/*.h fem/src
  cp hutiter/include/*.h fem/src/modules

  export LIBS="$LIBS -lamd"
  echo "===Compiling fem ==="
  cd fem
  ./configure --prefix=/usr \
        --with-matc="$srcdir"/trunk/matc/src/libmatc.a \
        --with-huti="$srcdir"/trunk/hutiter/src/libhuti.a \
        --with-eiof="$srcdir"/trunk/eio/src/libeiof.a \
        --with-umfpack="$srcdir"/trunk/umfpack/src/umfpack/libumfpack.a
  make
  export LIBS="$LIBS_ORIG"
  cd ..

  cd ElmerGUI
sed -i ElmerGUI.pri -f - <<'==='
s|\(QWT_INCLUDEPATH = /usr/include/qwt\)-qt4|\1|
s|\(QWT_LIBS = -lqwt\)-qt4|\1|
s|\(VTK_INCLUDEPATH = /usr/include/vtk-5\)\.2|\1.10|
s|VTK_LIBPATH = /usr/lib|&/vtk-5.10|
s|OCC_INCLUDEPATH = /usr/include/opencascade|OCC_INCLUDEPATH = /opt/opencascade/inc|
s|OCC_LIBPATH = /usr/lib|OCC_LIBPATH = /opt/opencascade/lib|
===
  qmake-qt4
  #qwt6 patches
sed -i Application/src/convergenceview.h -f - <<'==='
s|<qwt_data.h>|<qwt_series_data.h>\n#include <qwt_interval.h>\n#include <qwt_compat.h>|
===
sed -i Application/src/convergenceview.cpp -f - <<'==='
s|plot->clear();|//&|
s|d_curve->setRawData|d_curve->setRawSamples|
===

  MAKEFLAGS=-j1
  make
  cd ..
  cd ElmerGUIlogger
  qmake-qt4
  make
  cd ..

  cd ElmerGUItester
  qmake-qt4
  make
  cd ..

  cd post
  export LIBS="$LIBS -lX11"
  ./configure --prefix=/usr
  make
  cd ..

  cd front
  ./configure --prefix=/usr
  make
  cd ..


  export LDFLAGS="$LDFLAGS_ORIG"
  export LIBS="$LIBS_ORIG"
  export CPPFLAGS="$CPPFLAGS_ORIG"

}

package() {
  cd "$srcdir/trunk"
  modules="matc umfpack mathlibs elmergrid meshgen2d eio hutiter fem front post"
  for m in $modules; do
    cd $m
    make DESTDIR="$pkgdir/" install
    cd ..
  done
  modules="ElmerGUI"
  for m in $modules; do
    cd $m
    make INSTALL_ROOT="$pkgdir/" install
    cd ..
  done
  #Produce a arch compliant package
  mkdir -p $pkgdir/usr/lib
  mv $pkgdir/usr/share/elmersolver/lib/*.so $pkgdir/usr/lib/
  mv $pkgdir/usr/local/bin/* $pkgdir/usr/bin/
  
  
  rm -- "$pkgdir"/usr/{lib/libarpack.a,include/amd.h,include/umfpack.h,lib/libamd.a,lib/libumfpack.a}

}
