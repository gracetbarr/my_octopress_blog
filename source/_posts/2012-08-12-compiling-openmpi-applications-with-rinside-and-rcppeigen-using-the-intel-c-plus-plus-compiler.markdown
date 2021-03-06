---
layout: post
title: "Compiling OpenMPI Applications with RInside and RCppEigen using the Intel C++ Compiler"
date: 2012-08-12 23:29
comments: true
categories: [openmpi, c++, icpc, RInside, Rcpp, Eigen, RcppEigen]
---

This post is for those of you trying to use the [Intel C++ compiler](http://software.intel.com/en-us/articles/intel-compilers/), `icpc`, to compile an [OpenMPI](http://www.openmpi.org) application which includes [RInside](http://dirk.eddelbuettel.com/code/rinside.html) and [RcppEigen](http://cran.r-project.org/web/packages/RcppEigen/index.html). One would want to do this if you wanted to 


+ Create some matrices within R
+ Convert those matrices into Eigen matrices
+ Use OpenMPI to do some parallel calculation based on these matrices

This approach can be preferred over, say, using RMpi if you want to do the majority of your coding in C++, but with some matrices generated in R. In my case, I want to have some matrices in C++ which are identical to one's made in R, and so I want to use R's random number generation facilities, and also some optimization routines.

In any case, first make sure that the include files are in the following order:

``` c++ Order of Include Files 
#include "mpi.h"
#include <RInside.h>
#include <Rcpp.h>
#include <RcppEigen.h>
```

If `mpi.h` comes *after* `RInside.h` you will get the following sort of errors

``` sh Example of Error

/usr/local/include/openmpi/ompi/mpi/cxx/datatype.h(142): error: expected a type specifier
    virtual void Free();
                 ^

/usr/local/include/openmpi/ompi/mpi/cxx/datatype.h(142): error: expected a ")"
    virtual void Free();
                 ^

/usr/local/include/openmpi/ompi/mpi/cxx/datatype.h(142): error: expected an identifier
    virtual void Free();
                 ^
...
```

Once this is set up, I use the following compile command. My code is called `sjb_simple_smle_with_Rinside.cpp`.

``` sh icpc compile string

stevejb@ursamajor:~$ icpc -I/usr/local/include -pthread -I/usr/share/R/include -I/usr/lib/R/site-library/Rcpp/include -I/usr/local/lib/R/site-library/RInside/include -O3 -pipe -g -Wall    sjb_simple_smle_with_Rinside.cpp  -pthread -L/usr/local/lib -lmpi_cxx -lmpi -ldl -lm -Wl,--export-dynamic -lrt -lnsl -lutil -L/usr/lib/R/lib -lR -lblas -llapack -L/usr/lib/R/site-library/Rcpp/lib -lRcpp -Wl,-rpath,/usr/lib/R/site-library/Rcpp/lib -L/usr/local/lib/R/site-library/RInside/lib -lRInside -Wl,-rpath,/usr/local/lib/R/site-library/RInside/lib -o sjb_simple_smle_with_Rinside -I/usr/local/lib/R/site-library/RcppEigen/include
```

This compile string mostly came from compile string generated by the `mpi` examples in the `RInside` package.
