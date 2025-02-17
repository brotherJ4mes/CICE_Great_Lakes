:tocdepth: 3

.. _dev_infra:


Infrastructure
=======================

Kinds
------------------

**cicecore/shared/ice_kinds_mod.F90** defines the kinds datatypes used in CICE.  These kinds are
used throughout CICE code to define variable types.  The CICE kinds are adopted from the kinds
defined in Icepack for consistency in interfaces.

Constants
------------------

**cicecore/shared/ice_constants.F90** defines several model constants.  Some are hardwired parameters
while others have internal defaults and can be set thru namelist.

Dynamic Array Allocation
-------------------------------

CICE v5 and earlier was implemented using mainly static arrays and required several CPPs to be set to define grid size,
blocks sizes, tracer numbers, and so forth.  With CICE v6 and later, arrays are dynamically allocated and those
parameters are namelist settings.  The following CPPs are no longer used in CICE v6 and later versions,

 -DNXGLOB=100 -DNYGLOB=116 -DBLCKX=25 -DBLCKY=29 -DMXBLCKS=4 -DNICELYR=7 -DNSNWLYR=1 -DNICECAT=5 -DTRAGE=1 -DTRFY=1 -DTRLVL=1 -DTRPND=1 -DTRBRI=0 -DNTRAERO=1  -DTRZS=0 -DNBGCLYR=7 -DTRALG=0 -DTRBGCZ=0 -DTRDOC=0 -DTRDOC=0 -DTRDIC=0 -DTRDON=0 -DTRFED=0 -DTRFEP=0 -DTRZAERO=0 -DTRBGCS=0 -DNUMIN=11 -DNUMAX=99

as they have been migrated to :ref:`tabnamelist`

  nx_global, ny_global, block_size_x, block_size_y, max_blocks, nilyr, nslyr, ncat, nblyr, n_aero, n_zaero, n_algae, n_doc, n_dic, n_don, n_fed, n_fep, numin, numax


Time Manager
------------------

Time manager data is module data in **cicecore/shared/ice_calendar.F90**.  Much of the time manager
data is public and operated on during the model timestepping.  The model timestepping actually takes
place in the **CICE_RunMod.F90** file which is part of the driver code.

The time manager was updated in early 2021.  Additional information about the time manager can be found here, :ref:`timemanagerplus`



Communication
------------------

Two low-level communications packages, mpi and serial, are provided as part of CICE.  This software
provides a middle layer between the model and the underlying libraries.  Only the CICE mpi or
serial directories are compiled with CICE, not both.

**cicedyn/infrastructure/comm/mpi/**
is based on MPI and provides various methods to do halo updates, global sums, gather/scatter, broadcasts
and similar using some fairly generic interfaces to isolate the MPI calls in the code.

**cicedyn/infrastructure/comm/serial/** support the same interfaces, but operates
in shared memory mode with no MPI.  The serial library will be used, by default in the CICE scripts,
if the number of MPI tasks is set to 1.  The serial library allows the model to be run on a single
core or with OpenMP parallelism only without requiring an MPI library.

I/O
------------------

There are three low-level IO packages in CICE, io_netcdf, io_binary, and io_pio.  This software
provides a middle layer between the model and the underlying IO writing.
Only one of the three IO directories can be built with CICE.  The CICE scripts will build with the io_netcdf
by default, but other options can be selecting by setting ``ICE_IOTYPE`` in **cice.settings** in the
case.  This has to be set before CICE is built.

**cicedyn/infrastructure/io/io_netcdf/** is the
default for the standalone CICE model, and it supports writing history and restart files in netcdf
format using standard netcdf calls.  It does this by writing from and reading to the root task and
gathering and scattering fields from the root task to support model parallelism.

**cicedyn/infrastructure/io/io_binary/** supports files in binary format using a gather/scatter
approach and reading to and writing from the root task.

**cicedyn/infrastructure/io/io_pio/** support reading and writing through the pio interface.  pio
is a parallel io library (https://github.com/NCAR/ParallelIO) that supports reading and writing of
binary and netcdf file through various interfaces including netcdf and pnetcdf.  pio is generally
more parallel in memory even when using serial netcdf than the standard gather/scatter methods,
and it provides parallel read/write capabilities by optionally linking and using pnetcdf.
