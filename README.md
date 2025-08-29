# Legacy

Compile Glibc for legacy Enterprise Linux Systems


Needs make 4.3 due to An issue with older versions of glibc and make (v4.4+) can cause the stdio-common module to build continuously, appearing as if it is using only one core and hanging the process. This problem is a result of a perturbed build graph in the glibc Makefiles, causing stdio-common to be re-evaluated and rebuilt over and over:

  conda create -n buildlibc gcc gxx make=4.3 pexpect=4.9.0 texinfo -y
  
  conda activate buildlibc

Due to current LSF configuration its version of glibc is on the library path and either the system version, LSF or any other installed version should not be on the library path, you will get an error and it will stop execution:

  LD_LIBRARY_PATH=

Set compiler, dynamic linker, tools and variables:

  BUILD_CC=$CONDA_PREFIX/bin/gcc
  CC=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-gcc
  CXX=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-g++
  AR=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-ar
  AS=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-as
  NM=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-nm
  LD=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-ld
  RANLIB=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-ranlib
  READELF=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-readelf
  STRIP=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-strip
  OBJCOPY=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-objcopy
  OBJDUMP=$CONDA_PREFIX/bin/x86_64-conda-linux-gnu-objdump

Make sure the paths are correct:

  ls $BUILD_CC
  ls $CC
  ls $CXX
  ls $AR
  ls $AS
  ls $NM
  ls $LD
  ls $RANLIB
  ls $READELF
  ls $STRIP
  ls $OBJCOPY
  ls $OBJDUMP

Download and Extract Glibc:

  wget https://ftp.gnu.org/gnu/glibc/glibc-2.28.tar.xz
  
  unxz glibc-2.28.tar.xz
  
  tar -xvf glibc-2.28.tar
  
  cd glibc-2.28/
  
  mkdir build; cd build

Configure:
  MAKE=$CONDA_PREFIX/bin/make ../configure --enable-bind-now --disable-profile --cache-file=config.cache --without-cvs --with-elf --disable-sanity-checks \
    --prefix=/usr/local/app/rcs_bin/grid3.5/glibc-2.28 \
    --host=x86_64-conda-linux-gnu \
    --build=x86_64-conda-linux-gnu \
    --enable-kernel=3.10.0-1160.135.1 \
    --enable-shared \
    --enable-add-ons \
    --disable-multi-arch \
    --enable-obsolete-rpc \
    --enable-obsolete-nsl \
    --enable-stack-protector=strong \
    --enable-tunables \
    --disable-profile \
    --with-headers=/usr/include \
    --with-tls \
    --with-__thread \
    --without-cvs \
    --without-gd \
    --enable-fft=yes

Build:

  make -j 12 CFLAGS="$CFLAGS -Wno-error -O3 -g -Wl,-rpath,/usr/local/app/rcs_bin/grid3.5/glibc-2.28"

Install:

  make install

In this example we will be testing and running with Ollama on Red Hat Enterprise Linux 7 which ships with Glibc 2.17-326:

  conda install patchelf -y
  
  patchelf --remove-rpath /usr/local/app/rcs_bin/grid3.5/glibc-2.28/lib/ld-2.28.so
  
  mkdir llm; cd llm

  curl -LO https://ollama.com/download/ollama-linux-amd64.tgz
  
  tar -zxvf ollama-linux-amd64.tgz
  
  patchelf --set-interpreter /usr/local/app/rcs_bin/grid3.5/glibc-2.28/lib/ld-2.28.so --set-rpath /usr/local/app/rcs_bin/grid3.5/glibc-2.28/lib:/usr/lib:/usr/lib64 bin/ollama
  
  LD_PRELOAD=$CONDA_PREFIX/lib/libstdc++.so.6.0.34 ./bin/ollama serve
  
