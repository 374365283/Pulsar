# Pulsar
这个文档分为三个部分，第一部分是如何进入已经装好presto的镜像文件，第二部分是我运行tutorial的步骤和报错的位置，第三部分是我在新的容器中安装presto的流程。

## 如何进入已经装好presto的镜像文件
在Pulsar目录中的Pulsar.simg是我根据下面的安装流程安装好了的镜像文件，获取到镜像文件后，使用命令：
singularity -d build --sandbox pulsar Pulsar.simg
就可以获取到容器pulsar，随后：
singularity shell --writable pulsar
就可以进入容器，进入容器后：
source presto.sh
此时，容器中的环境就是我已经安装好presto的环境了。

## 运行tutorial的步骤和报错的位置
接下来开始运行tutorial：
source presto.sh
cd /dev/shm
mkdir test
cd test
wget http://www.cv.nrao.edu/~sransom/GBT_Lband_PSR.fil
readfile GBT_Lband_PSR.fil 
rfifind -time 2.0 -o Lband GBT_Lband_PSR.fil 
rfifind -time 1.0 -o Lband GBT_Lband_PSR.fil 
rfifind -nocompute -time 1.0 -freqsig 6.0 -mask Lband_rfifind.mask -o Lband GBT_Lband_PSR.fil 
prepdata -nobary -o Lband_topo_DM0.00 -dm 0.0 -mask Lband_rfifind.mask -numout 530000 GBT_Lband_PSR.fil
exploredat Lband_topo_DM0.00.dat 
realfft Lband_topo_DM0.00.dat 
explorefft Lband_topo_DM0.00.fft
accelsearch -numharm 4 -zmax 0 Lband_topo_DM0.00.dat
less -S Lband_topo_DM0.00_ACCEL_0
cp Lband_rfifind.inf Lband.inf

Lband.birds文件是我按照ppt上的example创建的(在/dev/shm/test中有):
#Freq  Width #harm grow? bary?
28.760 0.1   2     0     0
60.0   0.05  2     1     0

makezaplist.py Lband.birds
explorefft Lband_topo_DM0.00.fft      
到这个命令开始出错：prepfold -p 1.0 GBT_Lband_PSR.fil

## 在新的容器中安装presto的流程
以下是我在一个新的容器中安装presto的流程：
首先，获取一个镜像文件centos7.sif，并生成一个容器pulsar1，并进入容器：
singularity pull centos：7
singularity -d build --sandbox pulsar1/ centos_7.sif
singularity shell --writable pulsar1/

进入容器后，开始安装流程：
yum -y install vim wget
yum -y groupinstall "Development Tools" 
yum -y install libpng12-devel

在根目录创建文件presto.sh ，并写入以下内容：
export http_proxy=http://proxy.pi.sjtu.edu.cn:3004/
export https_proxy=http://proxy.pi.sjtu.edu.cn:3004/
export http_proxy=http://proxy.pi.sjtu.edu.cn:3004/
export https_proxy=http://proxy.pi.sjtu.edu.cn:3004
export PATH=$PATH:/usr/local/bin:/usr/local/astrosoft/presto/bin:/usr/local/astrosoft/pgplot:/usr/local/astrosoft/presto/bin:/nfshome/mcc/pfits:/usr/local/astrosoft/optimus:/usr/local/astrosoft/fv:/usr/local/astrosoft/psrcat_tar:/usr/local/astrosoft/tempo/src:/usr/local/astrosoft/tempo/bin:/usr/local/astrosoft/libpng/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/astrosoft/presto/lib:/usr/local/astrosoft/pgplot:/usr/local/astrosoft/fftw/lib:/usr/local/astrosoft/cfitsio/lib:/usr/local/astrosoft/libpng/lib
export C_INCLUDE_PATH=$C_INCLUDE_PATH:/usr/local/astrosoft/presto/include:/usr/local/astrosoft/cfitsio/include
export PKG_CONFIG_PATH=/usr/local/astrosoft/cfitsio/lib/pkgconfig:/usr/local/astrosoft/fftw/lib/pkgconfig：/usr/local/astrosoft/libpng/lib/pkgconfig
export PYTHONPATH=/usr/local/astrosoft/presto/python
export PGPLOT_DIR=/usr/local/astrosoft/pgplot
export PGPLOT_FONT=/usr/local/astrosoft/pgplot/grfont.dat
export PGPLOT_DEV=/xwine
export PGPLOT_LIB="-L /usr/X11R6/lib -lX11 -L /usr/local/astrosoft/pgplot -lpgplot"
export PRESTO=/usr/local/astrosoft/presto
export TEMPO=/usr/local/astrosoft/tempo
export PSRCAT_FILE=/usr/local/astrosoft/psrcat_tar/psrcat.db

source presto.sh

安装FFTW3.X 
cd /usr/local/src
wget http://www.fftw.org/fftw-3.3.5.tar.gz
tar xvf fftw-3.3.5.tar.gz
cd fftw-3.3.5
./configure --enable-shared --enable-single --prefix=/usr/local/astrosoft/fftw
make -j && make install

安装PGPLOT （参考https://blog.csdn.net/among12345/article/details/93767772）
cd /usr/local/src
wget ftp://ftp.astro.caltech.edu/pub/pgplot/pgplot5.2.tar.gz
tar xvf pgplot5.2.tar.gz
mkdir /usr/local/astrosoft/pgplot/
cd /usr/local/astrosoft/pgplot/
cp /usr/local/src/pgplot/drivers.list .
drivers.list文件里面选取这几个,把这几行前面的!去掉就可以了.

　　GIDRIV 1 /GIF GIF-format file, landscape
　　GIDRIV 2 /VGIF GIF-format file, portrait
　　NUDRIV 0 /NULL Null device (no output) Std F77
　　PSDRIV 1 /PS PostScript printers, monochrome, landscape Std F77
　　PSDRIV 2 /VPS Postscript printers, monochrome, portrait Std F77
　　PSDRIV 3 /CPS PostScript printers, color, landscape Std F77
　　PSDRIV 4 /VCPS PostScript printers, color, portrait Std F77
　　XWDRIV 1 /XWINDOW Workstations running X Window System C
　　XWDRIV 2 /XSERVE Persistent window on X Window System C

yum  -y install libX11 libX11-devel
yum -y install xorg-x11-drivers.x86_64
cd /usr/local/src/pgplot/src
cp grpckg1.inc grpckg1.inc.backup
cp pgplot.inc pgplot.inc.backup
vim grpckg1.inc
vim pgplot.inc

先备份，再更改
emacs grpckg1.inc & # Replace " PARAMETER (GRIMAX = 8) " in line 29
                    #    by   " PARAMETER (GRIMAX = 32) "
                    
emacs pgplot.inc &  # Replace " PARAMETER (PGMAXD=8) " in line 7
                    #    by   " PARAMETER (PGMAXD=32) "

cd /usr/local/astrosoft/pgplot/
/usr/local/src/pgplot/makemake /usr/local/src/pgplot linux g77_gcc
执行完毕之后会出现如下的文件
drivers.list   grexec.f   grpckg1.inc   makefile   pgplot.inc   rgb.txt

emacs makefile & # Replace "FCOMPL=g77"       in line 25 
                 #   by    "FCOMPL=gfortran" 
                 # 
                 # Replace "FFLAGC=-u -Wall -fPIC -O" in line 26
                 #   by    "FFLAGC=-ffixed-form -ffixed-line-length-none -u -Wall -fPIC -O"

在Makefile中加入 LINUXWSLIBS   = -L/usr/include/X11/lib -lxview -lolgx -lX11 -lm -lf2c -lgfortran 
make
make cpg

安装GLIB
yum -y install glib2-devel 

安装CFITSIO
cd /usr/local/src
wget  http://heasarc.gsfc.nasa.gov/FTP/software/fitsio/c/cfitsio-3.49.tar.gz
tar xvf cfitsio-3.49.tar.gz
cd cfitsio
./configure --prefix=/usr/local/astrosoft/cfitsio
make -j && make install

安装TEMPO
cd /usr/local/src
git clone http://git.code.sf.net/p/tempo/tempo
cd tempo
./prepare
./configure --prefix=/usr/local/astrosoft/tempo
make -j && make install
cd /usr/local/astrosoft/tempo
mkdir src
cp /usr/local/src/tempo/tempo.hlp src/
cp /usr/local/src/tempo/tempo.cfg src/

安装zlib，libpng
cd /usr/local/src
wget https://www.zlib.net/fossils/zlib-1.2.11.tar.gz
tar xvf zlib-1.2.11.tar.gz
cd zlib-1.2.11
 ./configure --prefix=/usr/local/astrosoft/zlib
 make -j 64 && make install
 cd ..
wget https://iweb.dl.sourceforge.net/project/libpng/libpng16/1.6.36/libpng-1.6.36.tar.gz
tar xvf libpng-1.6.36.tar.gz
 cd libpng-1.6.36
 ./configure --prefix=/usr/local/astrosoft/libpng --with-zlib-prefix=/usr/local/astrosoft/zlib LDFLAGS="-L/usr/local/astrosoft/zlib/lib -lz"
 make -j 64 && make install

presto安装
cd /usr/local/astrosoft/presto/src
make makewisdom 
make prep 
make
cd /usr/local/src
## 安装pip,numpy,scipy
wget https://files.pythonhosted.org/packages/c2/f7/c7b501b783e5a74cf1768bc174ee4fb0a8a6ee5af6afa92274ff964703e0/setuptools-40.8.0.zip
wget https://files.pythonhosted.org/packages/4c/4d/88bc9413da11702cbbace3ccc51350ae099bb351febae8acc85fec34f9af/pip-19.0.2.tar.gz
unzip setuptools-40.8.0.zip
cd setuptools-40.8.0
python setup.py install
cd ..
tar xvf pip-19.0.2.tar.gz
cd pip-19.0.2
python setup.py install
pip install numpy scipy

cd /usr/local/astrosoft/presto
vim setup.py
Note: you might need to add “gfortran” to the following list if 
you see errors relating to missing “g” functions….
这时需要下面一步，不然就可以直接make 
将ppgplot_libraries = ["cpgplot", "pgplot", "X11", "png", "m"]改为ppgplot_libraries = ["gfortran" , "cpgplot", "pgplot", "X11", "png", "m"] 
cd /usr/local/astrosoft/presto/python 
make 
这里需要注意，在根据INSTALL文件安装时，可能在make步骤之后还会有有一个make fftfit。这个时候需要看一下Makefile文件的build那下面的最后一行有没有这一行：python fftfit_src/test_fftfit.py。假如有这一行，就不需要make fftfit这一步了。可能看到此文章时他们已经修改了INSTALL文件。总之只要知道这一步和这一行是重复的就可以了。  
cd /usr/local/astrosoft/presto
export  PYTHONPATH=
在根目录的presto.sh中，将export PYTHONPATH=/usr/local/astrosoft/presto/python注释掉。
yum install python-devel
yum install fftw-devel
yum install libpng-devel
pip install .
cd /usr/local/astrosoft/presto
yum -y install tkinter
yum install tk-devel tcl-devel
到这里，我的安装流程结束。



