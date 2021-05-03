# Maintainer: Oliver Harley <oliver.r.harley+aur@gmail.com>

pkgbase=drake
pkgname=(drake python-drake)
pkgver=0.29.0
_url='https://github.com/RobotLocomotion/drake'
pkgrel=1
pkgdesc=""
arch=('x86_64')
url=""
license=('MIT')
makedepends=(bazel cmake pkgconfig)
checkdepends=(ctest)
depends=(
    coinor-libclp1
    coinor-libcoinutils3v5
    coinor-libipopt1v5
    default-jre
    jupyter-notebook
    libamd2
    libblas3
    libbz2-1.0
    libdouble-conversion3
    libeigen3-dev
    libexpat1
    libfreetype6
    libgflags2.2
    libgfortran5
    libglew2.1
    libglib2.0-0
    libglu1-mesa
    libglx0
    libhdf5-103
    libjpeg-turbo8
    libjsoncpp1
    liblapack3
    libldl2
    liblz4-1
    liblzma5
    libmumps-seq-5.2.1
    libnetcdf15
    libnlopt-cxx0
    libogg0
    libopengl0
    libpng16-16
    libqt5core5a
    libqt5gui5
    libqt5printsupport5
    libqt5widgets5
    libquadmath0
    libspdlog-dev
    libsqlite3-0
    libtheora0
    libtiff5
    libtinyxml2-dev
    libtinyxml2.6.2v5
    libx11-6
    libxml2
    libxt6
    libyaml-cpp-dev
    ocl-icd-libopencl1
    python
    python-ipywidgets
    python-lxml
    python-matplotlib
    python-numpy
    python-pydot
    python-pygame
    python-scipy
    python-tk
    python-tornado
    python-u-msgpack
    python-yaml
    python-zmq
    zlib1g
)

optdepends=()
provides=()
conflicts=()
replaces=()
backup=()
options=()
install=
changelog=
https://github.com/RobotLocomotion/drake/archive/refs/tags/v0.29.0.tar.gz
source("$pkgname-$pkgver.tar.gz::https://github.com/RobotLocomotion/$pkgname/archive/v$pkgver.tar.gz")
noextract=()
md5sums=()
validpgpkeys=()

prepare() {
  # Allow any bazel version
  # echo "*" > drake-${_pkgver}/.bazelversion

  # Get rid of hardcoded versions. Not like we ever cared about what upstream
  # thinks about which versions should be used anyway. ;) (FS#68772)
  sed -i -E "s/Gurobi 9.0.2 EXACT MODULE REQUIRED/Gurobi 9.* MODULE REQUIRED/" CMakeLists.txt
  sed -i -E "s/'([0-9a-z_-]+) .= [0-9].+[0-9]'/'\1'/" drake-${_pkgver}/drake/tools/pip_package/setup.py

  # cp -r drake-${_pkgver} drake-${_pkgver}-opt

  # These environment variables influence the behavior of the configure call below.
  export ${SNOPT_PATH}=/usr/bin/snopt/

  CMAKE_OPTS=(
   -DCMAKE_INSTALL_PREFIX=/usr \
   -DCMAKE_INSTALL_LIBDIR=/lib \
   -DCMAKE_BUILD_TYPE=None \
   -DCMAKE_SKIP_INSTALL_RPATH=YES \
   -DCMAKE_SKIP_RPATH=YES \
   -DWITH_GUROBI=ON \
   -DWITH_MOSEK=ON \
   -DWITH_ROBOTLOCOMOTION_SNOPT=ON -DWITH_SNOPT=OFF -DSNOPT_PATH='' \
   )
  export PYTHON_BIN_PATH=/usr/bin/python
  export USE_DEFAULT_PYTHON_LIB_PATH=1

  # See https://github.com/tensorflow/tensorflow/blob/master/third_party/systemlibs/syslibs_configure.bzl
  # export DRAKE_SYSTEM_LIBS="boringssl,curl,cython,gif,icu,libjpeg_turbo,lmdb,nasm,pcre,png,pybind11,zlib"
  export DRAKE_DOWNLOAD_CLANG=0
  export DRAKE_NCCL_VERSION=2.8
  export DRAKE_IGNORE_MAX_BAZEL_VERSION=1
  export DRAKE_MKL_ROOT=/opt/intel/mkl
  export NCCL_INSTALL_PATH=/usr
  # export GCC_HOST_COMPILER_PATH=/usr/bin/gcc
  export HOST_C_COMPILER=${CC}
  export HOST_CXX_COMPILER=${CXX}
  export CLANG_CUDA_COMPILER_PATH=/usr/bin/clang
  # export DRAKE_CUDA_PATHS=/opt/cuda,/usr/lib,/usr
  # export DRAKE_CUDA_VERSION=$(/opt/cuda/bin/nvcc --version | sed -n 's/^.*release \(.*\),.*/\1/p')
  # export DRAKE_CUDNN_VERSION=$(sed -n 's/^#define CUDNN_MAJOR\s*\(.*\).*/\1/p' /usr/include/cudnn_version.h)
  # export DRAKE_CUDA_COMPUTE_CAPABILITIES=5.2,5.3,6.0,6.1,6.2,7.0,7.2,7.5,8.0,8.6

  # # Required until https://github.com/tensorflow/tensorflow/issues/39467 is fixed.
  # export CC=gcc
  # export CXX=g++

  # Why use sandbox, when you could use a clean-chroot...?
  export BAZEL_ARGS="${BAZEL_ARGS} --spawn_strategy=local "
  # export BAZEL_ARGS="--config=mkl -c opt --copt=-I/usr/include/openssl-1.0 --host_copt=-I/usr/include/openssl-1.0 --linkopt=-l:libssl.so.1.0.0 --linkopt=-l:libcrypto.so.1.0.0 --host_linkopt=-l:libssl.so.1.0.0 --host_linkopt=-l:libcrypto.so.1.0.0"

}

build() {
	echo "Building with cuda and with non-x86-64 optimizations"
	cd "${srcdir}"/drake-${_pkgver}
	export CC_OPT_FLAGS="-march=haswell -O3"
	# export DRAKE_NEED_CUDA=1
	./configure
	bazel \
	build --config=avx2_linux \
	${BAZEL_ARGS[@]} \
	//drake:libdrake.so \
	//drake:libdrake_cc.so \
	//drake:install_headers \
	//drake/tools/pip_package:build_pip_package
	bazel-bin/drake/tools/pip_package/build_pip_package --gpu "${srcdir}"/tmpoptcuda
}

check() {
	cd "$pkgname-$pkgver"
	make -k check
}

package() {
	cd "$pkgname-$pkgver"
	make DESTDIR="$pkgdir/" install
}


