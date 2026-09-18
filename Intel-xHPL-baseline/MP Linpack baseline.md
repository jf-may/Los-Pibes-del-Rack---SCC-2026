``` bash
cp -r /opt/intel/oneapi/mkl/latest/share/mkl/benchmarks/mp_linpack ./mp_linpack_pibes-del-rack
cd mp_linpack_pibes-del-rack/
nano HPL.dat
```

``` dat
==============================
HPLinpack benchmark input file
==============================

HPL.out      output file name (if any)
6            device out (6=stdout,7=stderr,file)
1            # of problems sizes (N)
76800        Ns (Multiple of block size. Should be sqrt(80% total ram): 248448)
1            # of NBs
384          NBs
1            PMAP process mapping (0=Row-,1=Column-major)
1            # of process grids (P x Q)
2            Ps
3            Qs
16.0         threshold
3            # of panel fact
1 2 0        PFACTs (0=left, 1=Crout, 2=Right)
1            # of recursive stopping criterium
2            NBMINs (>= 1)
1            # of panels in recursion
2            NDIVs
3            # of recursive panel fact.
1 0 2        RFACTs (0=left, 1=Crout, 2=Right)
6            # of broadcast
0 1 2 3 4 5  BCASTs (0=1rg,1=1rM,2=2rg,3=2rM,4=Lng,5=LnM)
1            # of lookahead depth
1            DEPTHs (>=0)
0            SWAP (0=bin-exch,1=long,2=mix)
1            swapping threshold
1            L1 in (0=transposed,1=no-transposed) form
1            U  in (0=transposed,1=no-transposed) form
0            Equilibration (0=no,1=yes)
8            memory alignment in double (> 0)
```

```
nano runme_intel64_dynamic
```

```
#!/bin/bash
#===============================================================================
# Copyright 2001-2023 Intel Corporation.
#
# This software and the related documents are Intel copyrighted  materials,  and
# your use of  them is  governed by the  express license  under which  they were
# provided to you (License).  Unless the License provides otherwise, you may not
# use, modify, copy, publish, distribute,  disclose or transmit this software or
# the related documents without Intel's prior written permission.
#
# This software and the related documents  are provided as  is,  with no express
# or implied  warranties,  other  than those  that are  expressly stated  in the
# License.
#===============================================================================

# Set total number of MPI processes for the HPL (should be equal to PxQ).
export MPI_PROC_NUM=6

# Set the MPI per node for each node.
# MPI_PER_NODE should be equal to 1 or number of sockets on the system.
# It will be same as -perhost or -ppn paramaters in mpirun/mpiexec.
export MPI_PER_NODE=2

# Set the number of NUMA nodes per MPI. (MPI_PER_NODE * NUMA_PER_MPI)
# should be equal to number of NUMA nodes on the system.
export NUMA_PER_MPI=1

export OUT=xhpl_intel64_dynamic_outputs.txt

export HPL_EXE=xhpl_intel64_dynamic

# Unset this variable to avoid initialization failure
unset I_MPI_OFFLOAD

echo -n "This run was done on: "
date

# Capture some meaningful data for future reference:
echo -n "This run was done on: " >> $OUT
date >> $OUT
echo "HPL.dat: " >> $OUT
cat HPL.dat >> $OUT
echo "Binary name: " >> $OUT
ls -l ${HPL_EXE} >> $OUT
echo "This script: " >> $OUT
cat runme_intel64_dynamic >> $OUT
echo "Environment variables: " >> $OUT
env >> $OUT
echo "Actual run: " >> $OUT

# Environment variables can also be also be set on the Intel(R) MPI Library command
# line using the -genv option (to appear before the -np 1):

mpirun -perhost ${MPI_PER_NODE} -np ${MPI_PROC_NUM} ./runme_intel64_prv "$@" | tee -a $OUT

echo -n "Done: " >> $OUT
date >> $OUT

echo -n "Done: "
date
```

``` bash
module purge
module load intel
module load impi
salloc -N 3 -c 18 --ntasks-per-node=2
export OMP_NUM_THREADS=18
export MKL_NUM_THREADS=18
export I_MPI_PIN_DOMAIN=socket
./runme_intel64_dynamic
```

Results on https://github.com/jf-may/Los-Pibes-del-Rack---SCC-2026/blob/main/xhpl_intel64_dynamic_outputs.txt
