# RCC Internal Workshop
Parmanand Sinha
July 17, 2023

---

## UNIX system organization
Things go in standard directories: binaries in /bin, libraries in /lib, configuration in /etc, etc.
But not research programs on Midway. Why?

● Lots of software, much of it is non-standard.

● We want different versions of the same software package available at the same time, but without conflicts.

● Dependency management: different codes may require different versions of libraries.

---

## Source Compile in Linux 

Download the source and install it yourself


 **configure** // Compilation environment setting result: Makefile creation
                     Default install location: /usr/local/***

 **make** // compile. Run Makefile
 **make install** // copy

 Command sequence

 1) Unzip
 2) run configure 
 3) make
 4) make install
 5) Update $PATH
 
$PATH is a list of paths separated by colons containing all the location to look for an executable.

## Source Compile in Linux : Examples

---
Download and compile SQLite:

```
wget https://www.sqlite.org/2022/sqlite-autoconf-3360000.tar.gz
tar xvf sqlite-autoconf-3360000.tar.gz
cd sqlite-autoconf-3360000

./configure --prefix=$HOME/local
make -jN
make install

```
Download and compile GDAL:

```
wget http://download.osgeo.org/gdal/CURRENT/gdal-X.Y.Z.tar.gz
tar xvf gdal-X.Y.Z.tar.gz
cd gdal-X.Y.Z

./configure --prefix=$HOME/local --with-sqlite3=$HOME/local --with-static-proj4=/usr --with-geos=/usr/bin/geos-config
make -jN
make install
```


---
## Midway organization 

Software is installed at /software. 
● Directories are named
\$SOFTNAME-\$SOFTVERS-\$OS-\$ARCH 

● For example, gedit lives at
/software/gedit-2.28-el6-x86_64 

● bin, lib, src, doc under that directory


---

## Environment Modules

‘Environment Modules’ are the mechanism by which much of the software is made available to the users.

What happens when you type “module load openmpi/1.6”? 
● module looks for a file called

/software/modulefiles/openmpi/1.6 (or in \~/privatemodules …)

● Applies changes to the environment based on that file: 
○ load module dependencies

○ adds \$appdir/bin to PATH

○ adds \$appdir/lib to LD_LIBRARY_PATH

○ adds man directory to MANPATH, include directory to CPATH, etc...


---
## Environment Modules

LMOD (Lmod) and Environment modules (TCLmodule) are both environment module systems for managing software packages and their dependencies on high-performance computing (HPC) systems.

TCLmodule are written in ([Tcl](https://www.tcl.tk/)) syntax.

Modules are not the only way of managing software on clusters: increasingly common approaches include:

* Conda package manager (Python-centric but can manage software written in any language);
* Apptainer/Singularity, a means for deploying software in [containers](https://en.wikipedia.org/wiki/Operating-system-level_virtualization).


---
## Environment modules (TCLmodule)

A minimum example of TCL module file for loading Miniconda:

```
#%Module1.0
##
## Miniconda modulefile
##

## Specify the version of Miniconda
set version x.x.x

## Set the path for Miniconda
set home <path-to-miniconda>  # Replace with the actual path

## Add Miniconda binary paths to the shell's PATH variable
prepend-path PATH $home/bin

## Add Miniconda library paths to the LD_LIBRARY_PATH environment variable
prepend-path LD_LIBRARY_PATH $home/lib

## Set environmental variables specific to Miniconda
setenv CONDA_HOME $home
```

----

## Midway Build System

Midway3 Repo: https://git.rcc.uchicago.edu/rcc-staff/midway3-software.git

● Build scripts live in 

Midway2: the SVN repo at pubsw/software/build
Midway3: the git repo at midway3-software/build

● A subdirectory for every software package 

● Build steps:

○ Create subdirectory and populate with configuration files
○ Run the build scripts from top-level directory 
○ Test modulefile, then commit to repository.


---

## The build configuration subfolders

**buildopts.sh** defines important variables for the build, notably BUILDTYPE. **moduleconf.sh** (new) defines the module dependancies and package dependancies of the software.

**source.sh** contains the command(s) required to download the source. 
**test.bats** contains tests to ensure the program was installed correctly. 
**build.sh** is a script that builds and installs the software package. This is only required if BUILDTYPE is *custom*.
**build.sh.pre** and **build.sh.post** are optional scripts that are run before and after a build, respectively.


---

## Type of buildtype

*   configure -- ./configure; make; make install
*   rsync -- use rsync to copy $SRCDIR to $PREFIX
*   noop -- Only create the modulefile. The software is installed in other ways.
*   custom -- use build.sh in the package directory

---

## Module Example: tmux

Content of source.sh

`gitsource https://github.com/tmux/tmux.git`

Content of buildopts.sh 

```
SRCDIR=${SRCDIR}-${SOFTVERS}
BUILDTYPE=configure
CORES=8


export LIBEVENT_CFLAGS="-I${LIBEVENT_HOME}/include"
export LIBEVENT_LIBS="-L${LIBEVENT_HOME}/lib -levent_core" 
```

Content of moduleconf.sh

```
MODULEDESCRIPTION="tmux is a terminal multiplexer, it enables a number of terminals (or windows)
to be accessed and controlled from a single terminal."
MODULEDOCURL="https://github.com/tmux/tmux"
MODULELICENSE="opensource"
MODULETAGS="terminal"
BUILDDEPEND="libevent/2.1.12"
MODULEDEPEND="libevent/2.1.12"
```

## Module Example: tmux

**Run build scripts**

From the top-level folder: eg: for Midway3: midway3-software/build

```
SOFTNAME=tmux
SOFTVERS=3.2
. /get-source $SOFTNAME $SOFTVERS \# Get source for software

. /build $SOFTNAME $SOFTVERS \# compile software to /software/staging

. /build -i $SOFTNAME $SOFTVERS \# rsync from /software/staging to /software
```


---

## What about the modulefile?

● build puts modulefile in \~/privatemodules

● Test with “module load use.own; module load $SOFTNAME”

● If that works:

○ move \~/privatemodules/\$SOFTNAME/\$SOFTVER to midway3-software/modulefiles/\$SOFTNAME

● Commit changes to repository

## custom buildtype module 

**Example: R**

**Content of source.sh**

`curl -O https://cran.rstudio.com/src/base/R-4/R-${$SOFTVER}.tar.gz`

**Content of buildopts.sh**

```
SRCDIR=${SRCDIR}-${SOFTVERS}
BUILDTYPE=custom
```

**Content of build.sh**

```
./configure --enable-R-shlib --enable-optimisations --enable-openmp \
  --enable-BLAS-shlib --with-lapack --with-blas="-lopenblas" \
  --disable-R-profiling --with-pcre1 --prefix=$PREFIX
make
make install
```

**Content of moduleconf.sh**

```
MODULEDOCNAME="R"
MODULEDESCRIPTION="The R language and environment for statistical computing \
and graphics."
MODULELICENSE="GNU General Public License v2"
MODULEDOCURL="https://www.r-project.org"
MODULETAGS="development, general programming, graphics, R, statistics"
SOFTWARESUFFIX=DISTARCH
MODULEDEPEND="openblas/0.3.13 java/15.0.2"
BUILDDEPEND=""
MODULECONFLICT="R"
MODULEUSAGE=''
MODULEEXTRA='
prepend-path LD_LIBRARY_PATH $appdir/lib64/R/lib
prepend-path LIBRARY_PATH $appdir/lib64/R/lib
setenv OMP_NUM_THREADS 1
'
```


---

## Software Management with Spack

[Spack](https://spack.readthedocs.io/en/latest/) is an open source package manager that simplifies building, installing, customizing, and sharing HPC software stacks. 
It is written in pure Python including software build requirements and dependencies.

It follows Nix, GUIX based store model where all the software are installed inside its own filesystem


In midway3, two common repo is available at /software/spack-0.17.0-el8-x86_64/ and /software/spack-dev/ (Preferred for RCC CS)
Step1: source /software/spack-0.17.0-el8-x86_64/share/spack/setup-env.sh 



---
## Basic Usage

``` bash
spack list <software-name> # list a software
spack info <software-name> # show information and variants of a software
spack spec -I <software-name> # show specifications (dependencies)
```

Option `-I` or `--install-status` shows status of the software
dependencies i.e. installed (`+`) or will be installed during the
installation (`-`).

Now let’s install a new software’s from Spack:

``` bash
spack install <software_name> or <software_name@version> or <software_name@version %compiler@version> 
```
In general, `@version` for both software and compiler could be removed.
Spack installs the most stable version by default (see `spack versions
-s <software-name>`). You may find complete list of software that you
can install by using `spack list` or in Spack online [package
list](https://spack.readthedocs.io/en/latest/package_list.html). Also.
we can use `--verbose` option to see more details during the
installation, `--no-cache` to install a package directly from the
source, and `--overwrite` to overwrite an installed package.



---
## Basic Usage

After installation, we can find the software by:

``` bash
spack find # to see all installed software
spack find <software-name> # to find a software (use -lfvp to see hashes, flags, variants and pathes)
spack location -i <software-name> # to find location of a software 
```

And we can load the software:

``` bash
spack load <software_name>
spack find --loaded # see what is loaded
```


---

## Compilers

We can select compiler version and settings. To find and list compilers,
use:

``` bash
spack compiler list
spack compiler find
```

Compilers can be added to the Spack compilers list or removed from the
list by:

``` bash
spack compiler add <compiler-name@ver> # for example $(spack location -i gcc@10.1.0) add gcc 10 compiler that already is installed by Spack
spack compiler remove <compiler-name@ver>
```

Also, we can directly modify `compilers.yaml` file by:

``` bash
spack config edit compilers
```


---
## Personal Spack installation
// Setting up inside scratch directory

git clone --depth=100 --branch=releases/v0.20 https://github.com/spack/spack.git $SCRATCH/midway3/$USER/spack

cd $SCRATCH/midway3/$USER/spack
. share/spack/setup-env.sh

---


---
