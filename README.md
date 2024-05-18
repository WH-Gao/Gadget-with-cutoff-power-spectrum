# Gadget-with-cutoff-power-spectrum

This is a simple application of N-body simulation code Gadget4, which trys to run simple simulation using power spectrum with different cutoff frequncy and repeat result in Primordial fluctuations and nonlinear structure (Little,Winberg and Park,1991MNRAS.253..295L).

## 1.Install Gadget4

Gadget4 is a code for N-body cosmology simulation based on C++. You can find Gadget4 code at http://gitlab.mpcdf.mpg.de/vrs/gadget4. Also a detailed document is provided at https://wwwmpa.mpa-garching.mpg.de/gadget4. There are some simple tips on installing Gadegt4.   
   
First, you need C++ compiler:
```
sudo apt-get install g++
```
### 1)Install related libraries
There are 4 necessary libraries before Gadget4 installation:   
1.**mpi**:Message Passing Interface, you can choose MPICH (https://www.mpich.org) or OpenMPI (http://www.open-mpi.org).   
2.**gsl**:GNU scientific library at http://www.gnu.org/software/gsl.   
3.**fftw**:Fastest Fourier Transform in the West at http://www.fftw.org.   
4.**hdf5**:Hierarchical Data Format at http://hdf.ncsa.uiuc.edu/HDF5.   

Another 2 libraries **hwloc** and **vectorclass** are not essential.   

After downloading from official websites, these libraries can be esaily installed by   
```
./configure -prefix=/home/xxx/lib/libname(mpich/openmpi/gsl/hdf5)
make
make install
```

For **fftw**, you may need   
```
./configure -prefix=/home/xxx/lib/fftw  --enable-float --enable-sse
make
make install
```
to enable single precision.   
If you meet some error here, you may use   
```
sudo make
sudo make install
```
### 2)Modify environment variables
Then, you need modify .bashrc by
```
vim ~/.bashrc
```
and add
```
export LD_LIBRARY_PATH=/home/xxx/lib/mpich/lib:/home/xxx/lib/fftw/lib:/home/xxx/lib/hdf5/lib:/home/xxx/lib/gsl/lib:$LD_LIBRARY_PATH
export PATH=/home/xxx/lib/mpich/bin:/home/xxx/lib/fftw/bin:/home/xxx/lib/hdf5/bin:/home/xxx/lib/gsl/bin:$PATH
```
### 3)Install Gadget4
You can download Gadget4 from http://gitlab.mpcdf.mpg.de/vrs/gadget4 by 
```
git clone http://gitlab.mpcdf.mpg.de/vrs/gadget4
```
Then you need copy Template-Config.sh to Config.sh and Template-Makefile.systype to Makefile.systype.   
Then Uncomment
```
#SYSTYPE="Generic-gcc"
```
in Makefile.systype and modify buildsystem/Makefile.gen.libs:
```
GSL_INCL   = -I/home/xxx/lib/gsl/include
GSL_LIBS   = -L/home/xxx/lib/gsl/lib
FFTW_INCL  = -/home/xxx/lib/fftw/include
FFTW_LIBS  = -L/home/xxx/lib/fftw/lib
HDF5_INCL  = -I/home/xxx/lib/hdf5/include
HDF5_LIBS  = -L/home/xxx/lib/hdf5/lib
```
Finally use 
```
make
```
to generate a executable file Gadget4.   
In this project, you can use DM-L50-N128 in examples:
```
make DIR=examples/DM-L50-N128
```
You may also modify 
```
PYTHON   = python
```
in Makefile into 
```
PYTHON   = python3
```
to avoid some error.

Some installation tutorials can be found at https://kaizokuow.wordpress.com/2020/10/10/install-gadget4/, https://zhuanlan.zhihu.com/p/326781414 or official document https://wwwmpa.mpa-garching.mpg.de/gadget4.

## 2.Run Gadget4 using power spectrum with different cutoff frequncy.
According to Primordial fluctuations and nonlinear structure (Little,Winberg and Park,1991MNRAS.253..295L), you can use Gadget4 to run N-body simulations using power spectrum with different cutoff frequncy.   
Here you can simply use DM-L50-N128 in examples, a simulation from redshift z=63 to z=0.    
There are some parameters you need to change.   
First, change param.txt:
```
BoxSize                   64.0
...
NSample                                           64
GridSize                                          64
···
PowerSpectrumType                                 2
ReNormalizeInputSpectrum                          0
...
PowerSpectrumFile                                 powersp.txt
InputSpectrum_UnitLength_in_cm                    3.085678e24
```
Here, I provide a power spectrum powersp.txt at redshift z=63 based on Planck 2018 cosmological parameter. In this power spectrum file, two columns are $log(k)$ and $log(\Delta^2)$ respectively inorder to satisfy require of Gadget4. $\Delta^2=\frac {k^3 P(k)}{2\pi^2}$ and $log$ to the base 10 here. You need put this power spectrum file into your path.  
According to this paper, in this simulation, the smallest frequncy is fundamental frequncy $\frac{\pi}{32} h Mpc^{-1}$, and the largest frequncy is Nyquist frequncy $\pi h Mpc^{-1}$.   
Then, you can delete part of power spectrum with larger frequncy $k>\frac{\pi}{16},\frac{\pi}{8},\frac{\pi}{4},\frac{\pi}{2} h Mpc^{-1}$, then run
```
mpirun -np 8 ./Gadget4 param.txt
```
Finally, you can get different .hdf5 files at different redshifts. You can change output.txt to choose output redshifts (here parameter in output.txt is scale factor a, you can choose 1.0 to set redshift as z=0).    
You can use h5py in python to read .hdf5 file and get particle information to make a plot. An example is in readhdf5.ipynb.   
Here k_64.png and k_4.png are two results without cutoff and with cutoff at \frac{\pi}{16} for a $128^3$ particles simulation in $128 h Mpc^{-1}$ box respectively.  
You can change BoxSize, Nsample to change resolution of simulation, you can also use different number of nodes to run Gadget4 by change 
```
mpirun -np x ./Gadget4 param.txt
```
number x here. Seed in param.txt can be changed to change initial distribution of dark matter particles.
