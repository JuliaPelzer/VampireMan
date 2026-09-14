VampireMan + Pflotran + PETSc: Troubleshooting Guide
=====================================================

This guide explains how to resolve MPI and library issues when running VampireMan simulations using Pflotran and PETSc.

--------------------------------------------------------------------
Common Symptoms
--------------------------------------------------------------------
You may encounter errors such as:

`pflotran: error while loading shared libraries: libpetsc.so.3.24: cannot open shared object file: No such file or directory`

or

`libmpifort.so.0: cannot open shared object file: No such file or directory`

and later:

```
[unset]: launcher not compatible with PMI1 client
Exit code: 15
```

--------------------------------------------------------------------
Root Causes
--------------------------------------------------------------------
1. PETSc or MPI libraries are missing from LD_LIBRARY_PATH.
2. The MPI launcher (mpirun) is incompatible with the MPI version PETSc was built with.
3. The command-line argument "--" is not supported by your MPI version.
4. The pflotran.in input file is missing or incorrectly referenced.


--------------------------------------------------------------------
Step 1 — Set Up PETSc Environment
--------------------------------------------------------------------
Ensure the PETSc paths are correct:
```
export PETSC_DIR=/home/username/petsc
export PETSC_ARCH=arch-linux-c-opt
export LD_LIBRARY_PATH=$PETSC_DIR/$PETSC_ARCH/lib:$LD_LIBRARY_PATH
```

Check that the PETSc libraries exist:

```
ls $PETSC_DIR/$PETSC_ARCH/lib/libpetsc.so*
```

If Pflotran expects different library names, create symbolic links:

```
cd $PETSC_DIR/$PETSC_ARCH/lib
ln -sf libpetsc.so.3.024 libpetsc.so.3.24
ln -sf libpetsc.so.3.024.0 libpetsc.so.3.24.0
ln -sf libmpifort.so.12 libmpifort.so.0
ln -sf libmpi.so.12 libmpi.so.0
```

--------------------------------------------------------------------
Step 2 — Use PETSc’s Internal MPI Launcher
--------------------------------------------------------------------
PETSc provides its own compatible mpirun. Use it directly:

```
$PETSC_DIR/$PETSC_ARCH/bin/mpirun -n 1 pflotran -pflotranin <input_file>
```

To simplify usage, add this to your environment:

```
export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
alias mpirun='mpirun -x LD_LIBRARY_PATH'
```

--------------------------------------------------------------------
Step 3 — Verify Input File
--------------------------------------------------------------------
Ensure the input file exists:

```
ls datasets_out/<case>/datapoint-0/pflotran.in
```

Test Pflotran manually:

```
mpirun -n 1 pflotran -pflotranin datasets_out/<case>/datapoint-0/pflotran.in
```

If this runs correctly, VampireMan will also work once configured.


--------------------------------------------------------------------
Step 4 — Run VampireMan Without Code Changes
--------------------------------------------------------------------
Set up the environment before running:

```
export PETSC_DIR=/home/username/petsc
export PETSC_ARCH=arch-linux-c-opt
export LD_LIBRARY_PATH=$PETSC_DIR/$PETSC_ARCH/lib:$LD_LIBRARY_PATH
export PATH=$PETSC_DIR/$PETSC_ARCH/bin:$PATH
alias mpirun='mpirun -x LD_LIBRARY_PATH'
```

Then start VampireMan as usual.
