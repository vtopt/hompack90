# ACM TOMS Algorithm 777: HOMPACK90: A suite of Fortran 90 Codes for Globally Convergent Homotopy Algorithms

HOMPACK90 is a suite of FORTRAN 90 subroutines for solving nonlinear
systems of equations by homotopy methods.  There are subroutines for
fixed point, zero finding, and general homotopy curve tracking problems,
utilizing both dense and sparse Jacobian matrices, and implementing
three different algorithms: ODE-based, normal flow, and augmented
Jacobian.  The (driver) subroutines called by the user are given in the
table below, and are well documented internally.  The user need not
be concerned with any other subroutines in HOMPACK90.

```
                  Problem type
 --------|--------|--------|--------|--------|--------|
      x = f(x)    |    F(x) = 0     |rho(a,lambda,x)=0|
 --------|--------|--------|--------|--------|--------|
  dense  | sparse | dense  | sparse | dense  | sparse |  Algorithm
 --------|--------|--------|--------|--------|--------|---------------------
  FIXPDF | FIXPDS | FIXPDF | FIXPDS | FIXPDF | FIXPDS | ODE based
 --------|--------|--------|--------|--------|--------|---------------------
  FIXPNF | FIXPNS | FIXPNF | FIXPNS | FIXPNF | FIXPNS | normal flow
 --------|--------|--------|--------|--------|--------|---------------------
  FIXPQF | FIXPQS | FIXPQF | FIXPQS | FIXPQF | FIXPQS | augmented Jacobian
 --------|--------|--------|--------|--------|--------|---------------------

```

The sparse subroutines use either the packed skyline storage scheme
standard in structural mechanics or the compressed sparse row storage
format, but any sparse storage scheme can be used by replacing some of
the low-level HOMPACK90 routines with user-written routines.  The
stepping subroutines ``STEP??`` or the reverse call
subroutines ``STEPNX`` and ``ROOTNX`` may be of
interest to some users with special curve tracking needs.

 - This code has been re-uploaded with the permission of Dr. Layne Watson.
   All comments and questions should be directed to him (see contact info at
   the bottom of this file).

## Organizational Details

The original source code, exactly as distributed by ACM TOMS, is included in
the ``src`` directory.
The ``src`` directory also contains its own ``README`` and build instructions.
Comments at the top of each subroutine document their proper usage.

HOMPACK90 is organized in two different ways: by
algorithm/problem type and by subroutine level. There are three levels
of subroutines. The top level consists of drivers, one for each problem
type and algorithm type. Normally these drivers are called by the user,
and the user need know nothing beyond them. They allocate storage for
the lower level routines, and all the arrays are variable dimension, so
there is no limit on problem size. The second subroutine level
implements the major components of the algorithms such as stepping
along the homotopy zero curve, computing tangents, and the end game for
the solution at lambda = 1 . A sophisticated user might call these
routines directly to have complete control of the algorithm, or for
some other task such as tracking an arbitrary parametrized curve over
an arbitrary parameter range.  The lowest subroutine level handles the
numerical linear algebra, and includes some LAPACK and BLAS routines.
All the linear algebra and associated data structure handling are
concentrated in these routines, so a user could incorporate his own
data structures by writing his own versions of these low level routines.

The organization of HOMPACK90 by algorithm/problem type is shown in the
above table, which lists the driver name for each algorithm and problem
type.  Using brackets to indicate the three subroutine levels described
above, the natural grouping of the HOMPACK90 routines is:

```
 [FIXPDF] [FODE, ROOT, SINTRP, STEPS] [DGEQPF]

 [FIXPDS] [FODEDS, ROOT, SINTRP, STEPDS] [GMFADS, GMRES,
      GMRILUDS, ILUFDS, ILUSOLVDS, MULTDS, MULT2DS, PCGDS, SOLVDS]

 [FIXPNF] [ROOTNF, STEPNF, TANGNF] [DGEQPF, DORMQR, ROOT]

 [FIXPNS] [ROOTNS, STEPNS, TANGNS] [GMFADS, GMRES, GMRILUDS,
      ILUFDS, ILUSOLVDS,  MULTDS, MULT2DS, PCGDS, ROOT, SOLVDS]

 [FIXPQF] [ROOTQF, STEPQF, TANGQF] [DGEQRF, DORGQR, UPQRQF]

 [FIXPQS] [ROOTNS, STEPQS, TANGNS] [GMFADS, GMRES, GMRILUDS,
      ILUFDS, ILUSOLVDS, MULTDS, MULT2DS, PCGDS, ROOT, SOLVDS]

 [POLSYS1H] [FIXPNF, ROOTNF, STEPNF, TANGNF]
      [DGEQPF, DGEQRF, DORMQR, DIVP, FFUNP, GFUNP, HFUNP, HFUN1P,
       INITP, MULP, OTPUTP, POWP, RHO, RHOJAC, ROOT, SCLGNP, STRPTP]
```

The ``LAPACK`` and ``BLAS`` subroutines used by HOMPACK90
are ``DCOPY, DDOT, DGEMM, DGEMV, DGEQPF, DGEQR2, DGEQRF, DGER, DLAIC1,
DLAMCH, DLAPY2, DLARF, DLARFB, DLARFG, DLARFT, DNRM2, DORG2R, DORGQR,
DORM2R, DORMQR, DSCAL, DSWAP, DTPMV, DTPSV, DTRMM, DTRMV, DTRSV,
IDAMAX, ILAENV, LSAME, XERBLA. ``

The user written subroutines, of which exactly two must be
supplied depending on the driver chosen, are ``F, FJAC, FJACS,
RHO, RHOA, RHOJAC,`` and ``RHOJS,``.  These external
subroutines must conform to the interfaces contained in the module
``HOMOTOPY``. The module ``REAL_PRECISION``
contains machine dependent constants, which must be changed
appropriately before compilation. The module
``HOMPACK90_GLOBAL`` contains global storage, and must be
used by the user written subroutines.

## Testing and Installation

``HOMPACK90`` consists of 4 modules---
``HOMOTOPY`` (contains interfaces for the user written
external subroutines), ``HOMPACK90`` (encapsulates all the
drivers), ``HOMPACK90_GLOBAL`` (global dynamic storage),
``REAL_PRECISION`` (defines precision of all reals)---and
external subroutines, all contained in two files
``HOMPACK90.f`` and ``LAPACK.f``.  The file sample.f
contains templates for the user written subroutines.  There are three
main programs ``MAIN[FPS].f`` for testing, with sample output
given in the files ``MAIN[FPS].out``.  ``MAINF.f``
and ``MAINS.f`` have no input files; ``MAINP.f``
reads a data file ``INNHP.DAT`` and writes the solution in a
file ``OUTHP.DAT`` (for post-processing), since this is
normally how the polynomial system driver ``POLSYS1H`` would
be used.

To test the dense (F), sparse (S), polynomial system (P)
algorithms respectively in ``HOMPACK90``, compile and link in
order the files ``HOMPACK90.f LAPACK.f MAINF.f``
(``MAINS.f, MAINP.f`` respectively).  The modules and
external subroutines in ``HOMPACK90.f`` and
``LAPACK.f`` (``BLAS`` and ``LAPACK``
routines) can be kept in module and object libraries and need not be
recompiled.

## Reference and Contact

To cite this work, use:

```
    @article{alg777,
        author = {Watson, Layne T. and Sosonkina, Maria and Melville, Robert C. and Morgan, Alexander P. and Walker, Homer F.},
        title = {Algorithm 777: HOMPACK90: A Suite of Fortran 90 Codes for Globally Convergent Homotopy Algorithms},
        year = {1997},
        volume = {23},
        number = {4},
        journal = {ACM Trans. Math. Softw.},
        pages = {514???549},
        doi = {10.1145/279232.279235}
    }
```

Inquiries should be directed to

Layne T. Watson,  
Department of Computer Science, VPI&SU,  
Blacksburg, VA 24061-0106;  
(540) 231-7540;  
ltw@vt.edu  
