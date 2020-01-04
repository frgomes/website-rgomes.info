+++
title = "Python stack for GPU programming"
date = 2013-06-01T12:00:00Z
[taxonomies]
categories = ["articles", "recipe"]
tags = ["python", "gpu", "cuda", "copperhead", "build", "sources"]
+++
I've recently started to play with Copperhead, which is a research project by Nvidia which provides GPU programming in a very elegant way on top of Python language.

If you are interested on CUDA/GPU development, I recommend you stick with Nvidia requirements, in particular regarding specific distributions which are supported by Nvidia CUDA Toolkit. Since I don't like Ubuntu, I've tried to install the aforementioned toolkit on Debian, but I faced troubles. In the end, I've built a separate box specially for CUDA/GPU development, based on Ubuntu. My environment consists on:

 * Kubuntu 11.10 ( which is basically Ubuntu 11.10 + KDE4 )
 * Nvidia CUDA Toolkit 5.0 

Long story short, I've decided to build Python and several libraries from sources, since their versions were somewhat old or inconvenient for my purposes. I'm building the stack below from sources:

 * Python 2.7.3
 * Numpy 1.6.2
 * Scypy 0.11
 * HDF5 1.8.10
 * PyTables 2.4.0
 * ViTables 2.1
 * Pandas 0.9.1
 * Copperhead trunk
 * recent versions of IPython and Cython

The strategy I employed consists on:

 * build Python 2.7.3
 * install virtualenv and virtualenvwrapper on top of Python 2.7.3
 * install everything else on top of a virtual environment

You can see the entire script below:

```bash
#!/bin/bash -x

####
#
# We will install this stack onto /opt/local
#
# Notice that I've installed NVidia CUDA5 Toolkit on
# /opt/local/cuda but, for some reason, Copperhead seems to
# require it from /usr/local/cuda. I've circumvented the trouble by
# simply creating a symlink, as you can see in function
# build_copperhead.
#
#####

export PREFIX=/opt/local

function create_opt_local() {
  rm -r -f ~/.virtualenvs
  rm -r -f ${PREFIX}/bin ${PREFIX}/include ${PREFIX}/lib ${PREFIX}/share
  sudo mkdir -p ${PREFIX}/bin ${PREFIX}/include ${PREFIX}/lib ${PREFIX}/share
  sudo chown -R rgomes:rgomes ${PREFIX}
}

#--------------------------------------------------
# essential tools
#--------------------------------------------------
function install_essential_tools() {
  sudo apt-get install zip unzip bzip2 gzip xz-utils wget curl -y
  sudo apt-get install build-essential gcc g++ gfortran -y
  sudo apt-get install autoconf scons pkg-config -y
  sudo apt-get install git bzr mercurial -y
}

#--------------------------------------------------
# downloading everything
#--------------------------------------------------
function download_everything() {
  mkdir -p ~/Downloads
  pushd ~/Downloads
  if [ ! -f install_python_environment.done ] ;then
    wget http://sourceforge.net/projects/boost/files/boost/1.48.0/boost_1_48_0.tar.bz2
    wget http://thrust.googlecode.com/files/thrust-1.6.0.zip
    wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tar.xz
    wget http://downloads.sourceforge.net/project/numpy/NumPy/1.6.2/numpy-1.6.2.tar.gz
    wget http://downloads.sourceforge.net/project/scipy/scipy/0.11.0/scipy-0.11.0.tar.gz
    wget http://www.hdfgroup.org/ftp/HDF5/current/src/hdf5-1.8.10.tar.bz2
    wget http://www.hdfgroup.org/ftp/lib-external/szip/2.1/src/szip-2.1.tar.gz
    wget http://downloads.sourceforge.net/project/pytables/pytables/2.4.0/tables-2.4.0.tar.gz
    wget http://vitables.googlecode.com/files/ViTables-2.1.tar.gz
    wget http://pypi.python.org/packages/source/p/pandas/pandas-0.9.1.tar.gz
    touch install_python_environment.done
  else
    echo "Files already downloaded"
  fi
  popd
}

#--------------------------------------------------
# build Python from sources
#--------------------------------------------------
function build_python() {
  cd ~/sources
  sudo apt-get install libicu44 libicu-dev zlib1g-dev libbz2-dev libncurses5-dev libreadline-gplv2-dev libsqlite3-dev libssl-dev libgdbm-dev -y
  #--
  tar xf ~/Downloads/Python-2.7.3.tar.xz
  cd Python-2.7.3
  export CC=gcc
  export CXX=g++
  export LD_RUN_PATH=${PREFIX}/lib
  ./configure --prefix=${PREFIX} --enable-shared --enable-unicode=ucs4 --with-pydebug
  make -j 8
  #make test
  make install
  unset CC CXX LD_RUN_PATH
}

#--------------------------------------------------
# install package managers
#--------------------------------------------------
function install_package_managers() {
  cd ~/sources
  curl http://python-distribute.org/distribute_setup.py | ${PREFIX}/bin/python
  curl https://raw.github.com/pypa/pip/master/contrib/get-pip.py | ${PREFIX}/bin/python
}

#--------------------------------------------------
# install virtualenv and virtualenvwrapper
#--------------------------------------------------
function install_virtualenv() {
  cd ~/sources
  sudo apt-get install python-virtualenv virtualenvwrapper -y
}

#--------------------------------------------------
# virtual environment for Python 2.7
#--------------------------------------------------
function create_virtualenv_py27() {
  mkvirtualenv --no-site-packages --python=${PREFIX}/bin/python py27
  #--
  echo "export PATH=${PREFIX}/bin:${PREFIX}/cuda/bin:$PATH" >> ~/.virtualenvs/py27/bin/postactivate
  echo "export LD_LIBRARY_PATH=${PREFIX}/lib" >> ~/.virtualenvs/py27/bin/postactivate
}

#--------------------------------------------------
# install other packages with pip
#--------------------------------------------------
function install_packages_1() {
  pip install setuptools --upgrade
  pip install distribute --upgrade
  pip install Sphinx
  pip install Cython
  pip install tornado
  pip install pyzmq
  pip install ipython
  pip install uncertainties
}

#--------------------------------------------------
# build numpy from sources
#--------------------------------------------------
function build_numpy() {
  cd ~/sources
  sudo apt-get install libatlas-base-dev libblas-dev libatlas-base-dev -y
  tar xf ~/Downloads/numpy-1.6.2.tar.gz
  cd numpy-1.6.2
  rm -r -f build
  python --version
  python setup.py --requires
  python setup.py build
  python setup.py install
}

#--------------------------------------------------
# build scipy from sources
#--------------------------------------------------
function build_scipy() {
  cd ~/sources
  tar xf ~/Downloads/scipy-0.11.0.tar.gz
  cd scipy-0.11.0
  rm -r -f build
  python --version
  python setup.py --requires
  python setup.py build
  python setup.py install
}

#--------------------------------------------------
# install other packages with pip
#--------------------------------------------------
function install_packages_2() {
  pip install numexpr
}

#--------------------------------------------------
# build szip from sources
#--------------------------------------------------
function build_szip() {
  cd ~/sources
  sudo apt-get install zlib-bin -y
  #--
  tar xf ~/Downloads/szip-2.1.tar.gz
  cd szip-2.1
  ./configure --prefix=${PREFIX} --enable-encoding
  make clean
  make -j 8
  make install
}

#--------------------------------------------------
# build HDF5 from sources - requires szip
#--------------------------------------------------
function build_hdf5() {
  cd ~/sources
  sudo apt-get build-dep hdf5 -y
  tar xf ~/Downloads/hdf5-1.8.10.tar.bz2
  cd hdf5-1.8.10
  CC=/usr/bin/mpicc ./configure --prefix=${PREFIX} --enable-shared --enable-parallel --with-szlib=${PREFIX}
  make clean
  make -j 8
  make check
  make install
  make check-install
  make clean
}

#--------------------------------------------------
# build pytables from sources
#--------------------------------------------------
function build_pytables() {
  cd ~/sources
  sudo apt-get install libbz2-dev liblzo2-dev -y
  #--
  tar xf ~/Downloads/tables-2.4.0.tar.gz
  cd tables-2.4.0/
  python --version
  python setup.py --requires
  CC=/usr/bin/mpicc HDF5_DIR=${PREFIX} python setup.py install
}

#--------------------------------------------------
# build vitables from sources
#--------------------------------------------------
function build_vitables() {
  cd ~/sources
  tar xf ~/Downloads/ViTables-2.1.tar.gz
  cd ViTables-2.1/
  python --version
  python setup.py --requires
  python setup.py build
  python setup.py install
}

#--------------------------------------------------
# build pandas from sources
#--------------------------------------------------
function build_pandas() {
  cd ~/sources
  tar xf ~/Downloads/pandas-0.9.1.tar.gz
  cd pandas-0.9.1
  rm -r -f build dist
  python --version
  python setup.py --requires
  python setup.py build
  python setup.py install
}

#--------------------------------------------------
# build boost from sources
#--------------------------------------------------
function build_boost() {
  cd ~/sources
  tar xf ~/Downloads/boost_1_48_0.tar.bz2
  cd boost_1_48_0
  #-- install on ${PREFIX}
  mkdir -p ${PREFIX}
  #--
  ./bootstrap.sh --prefix=${PREFIX} --with-python=${PREFIX}/bin/python --with-icu
  ./b2 -j 8
  ./b2 install
}

#--------------------------------------------------
# install thrust library
#--------------------------------------------------
function build_thrust() {
  pushd ${PREFIX}/include
  unzip ~/Downloads/thrust-1.6.0.zip
  popd
}

#--------------------------------------------------
# build copperhead from sources
#--------------------------------------------------
function build_copperhead() {
  cd ~/sources
  git clone http://github.com/copperhead/copperhead.git
  cd copperhead
  #--
  rm -r -f dist stage .sconf_temp .sconsign.dblite config.log
  rm -r -f ${PREFIX}/lib/python2.7/site-packages/copperhead*
  #--
  sudo rm /usr/local/cuda
  sudo ln -s ${PREFIX}/cuda /usr/local/cuda
  sudo ldconfig /usr/local/cuda/lib64
  #--
cat << EOD > siteconf.py
BOOST_INC_DIR = "${PREFIX}/include"
BOOST_LIB_DIR = "${PREFIX}/lib"
BOOST_PYTHON_LIBNAME = "boost_python"
CUDA_INC_DIR = "/usr/local/cuda/include"
CUDA_LIB_DIR = "/usr/local/cuda/lib64"
NP_INC_DIR = "${PREFIX}/lib/python2.7/site-packages/numpy/core/include"
TBB_INC_DIR = None
TBB_LIB_DIR = None
THRUST_DIR = "${PREFIX}/include"
EOD
  cat siteconf.py
  #--
  python --version
  python setup.py install
}


#--------------------------------------------------
# main script
#--------------------------------------------------

create_opt_local
install_essential_tools

download_everything

build_python
install_package_managers

install_virtualenv
source /etc/bash_completion.d/virtualenvwrapper
create_virtualenv_py27
workon py27

install_packages_1
build_numpy
build_scipy
install_packages_2
build_szip
build_hdf5
build_pytables
build_vitables
build_pandas
build_boost
build_thrust
build_copperhead

echo "done."
```

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. Thanks
