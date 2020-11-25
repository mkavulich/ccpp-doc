.. _ParamSpecOutput:

********************************
Parameterization-specific Output
********************************

========
Overview
========

When used with UFS and the SCM, the CCPP offers the capability of outputting tendencies of temperature,
zonal wind, meridional wind, ozone, and specific humidity produced by the parameterizations of selected
suites. This capability is useful for understanding the behavior of the individual parameterizations in
terms of magnitude and spatial distribution of tendencies, which can help model developers debug, refine,
and tune their schemes. 

The CCPP also enables outputting two-dimensional (2D) or three-dimensional (3D) arbitrary diagnostics
from the parameterizations. This capability is targeted to model developers who may benefit from analyzing
intermediate quantities computed in one or more parameterizations. One example of desirable diagnostic is
tendencies from sub-processes within a parameterization, such as the tendencies from condensation,
evaporation, sublimation, etc. from a microphysics parameterization. The output is done using CCPP-provided
2D- and 3D arrays, and the developer can fill positions 1, 2, .., N of the array. Important aspects of the
implementation are that memory is only allocated for the necessary positions of the array and that all
diagnostics are output on physics model levels. An extension to enable output on radiation levels may be
considered in future implementations.

These capabilities have been tested and are expected to work with the following suites:

* UFS: GFSv15p2 and GFSv16beta suites 
* SCM: GFSv15p2, GFSv16beta, and GSD_v1 suites 

==========
Tendencies
==========

This section describes the tendencies available, how to set the model to prepare them and how to output
them. It also contains a list of frequently-asked questions in :numref:`Section %s <AvailTendFAQ>`. 

Available Tendencies
--------------------
At this time, it is possible to output 40 different tendencies. Not all schemes produce all tendencies.
For example, the orographic and convective gravity wave drag (GWD) schemes produce tendencies of
temperature and momentum, but not of specific humidity and ozone. Similarly, only the planetary boundary
layer (PBL), deep and shallow convection, and microphysics schemes produce specific humidity tendencies.
A complete list of the tendencies that can be output is shown in :numref:`Table %s <avail_tendencies>`.

In addition to the tendencies from specific schemes, the output includes tendencies from all physics
schemes (“All” in :numref:`Table %s <avail_tendencies>`) and from all non-physics processes (“None” in
:numref:`Table %s <avail_tendencies>`).  Examples of non-physical processes are dynamical core processes
such as advection and nudging toward climatological fields.

In the supported suites, there are two types of schemes that produce ozone tendencies: PBL and ozone
photochemistry. The total tendency produced by the ozone photochemistry scheme (NRL 2015 scheme) is
subdivided by subprocesses: production and loss (combined as a single subprocess), quantity of ozone
present in the column above a grid cell, influences from temperature, and influences from mixing ratio.
For more information about the NRL 2015 ozone photochemistry scheme, consult the CCPP Scientific
Documentation `here <https://dtcenter.ucar.edu/GMTB/v4.0/sci_doc/GFS_OZPHYS.html>`_. 

.. _avail_tendencies:

.. table:: Complete list of available tendencies

   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   | **Tendency**      | **Associated**   |   **From CCPP scheme**     |  **Array**        | **Units**                     |
   |                   | **Namelist**     |                            |                   |                               |
   |                   | **Variables**    |                            |                   |                               |
   +===================+==================+============================+===================+===============================+
   | Temperature       | ldiag3d          | Long-wave radiation        | dt3dt_lw          | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Short-wave radiation       | dt3dt_sw          | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | PBL                        | dt3dt_pbl         | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Deep convection            | dt3dt_deepcnv     | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Shallow convection         | dt3dt_shalcnv     | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Microphysics               | dt3dt_mp          | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Orographic GWD             | dt3dt_orogwd      | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Rayleigh damping           | dt3dt_rdamp       | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Convective GWD             | dt3dt_cnvgwd      | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | All                        | dt3dt_phys        | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | None                       | dt3dt_nophys      | K s\ :sup:`-1`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   | Meridional Wind   | ldiag3d          | PBL                        | dv3dt_pbl         | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Deep convection            | dv3dt_deepcnv     | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Shallow convection         | dv3dt_shalcnv     | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Microphysics               | dv3dt_mp          | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Orographic GW              | dv3dt_orogwd      | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Rayleigh damping           | dv3dt_rdamp       | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Convective GW              | dv3dt_cnvgmw      | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | All                        | dv3dt_phys        | m s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | None                       | dv3dt_nophys      | n s\ :sup:`-2`                |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   | Specific humidity | ldiag3d, qdiag3d | PBL                        | dq3dt_pbl         | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Deep convection            | dq3dt_deepcnv     | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Shallow convection         | dq3dt_shalcnv     | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Microphysics               | dq3dt_mp          | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | All                        | dq3dt_phys        | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | None                       | dq3dt_nophys      | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   | Ozone             | ldiag3d, qdiag3d | PBL                        | dq3dt_o3pbl       | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Ozone: Production and loss | dq3dt_o3prodloss  | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Ozone: Mixing              | dq3dt_o3mix       | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Ozone: Temperature         | dq3dt_o3temp      | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | Ozone: Column              | dq3dt_o3column    | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | All                        | dq3dt_o3phys      | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+
   |                   |                  | None                       | dq3dt_o3nophys    | kg kg\ :sup:`-1` s\ :sup:`-1` |
   +-------------------+------------------+----------------------------+-------------------+-------------------------------+

Activating Tendencies
---------------------

For performance reasons, the preparation of tendencies for output is off by default in the UFS and
can be turned on via a set of namelist options. Since the SCM is not operational and has a relatively
tiny memory footprint, these tendencies are turned on by default in the SCM. 

There are two namelist variables associated with this capability: ``ldiag3d`` and ``qdiag3d``. To prepare the
tendencies of temperature and momentum, it is necessary to set ``ldiag3d`` to true in the ``&gfs_physics_nml``
portion of the namelist file ``input.nml``. To prepare the tendencies of temperature, momentum, specific
humidity and ozone, it is necessary to set both ``ldiag3d`` and ``qdiag3d`` to true in the ``&gfs_physics_nml``
portion of the namelist file ``input.nml``. The capability to prepare only the tendencies of specific
humidity and ozone is not supported. Recall that these options must be changed from their defaults for
the UFS to activate this functionality, but they are already set by default for the SCM.

Note that there is a third namelist variable, ``lssav``, associated with the output of
parameterization-specific information. The value of ``lssav`` is overwritten to true in the code, so
the value used in the namelist is irrelevant. 

While the tendencies output by the SCM are instantaneous, the tendencies output by the UFS are averaged
over the number of hours specified by the user in variable ``fhzero`` in the ``&gfs_physics_nml`` portion of the
namelist file ``input.nml``. Variable ``fhzero`` must be an integer (it cannot be zero). 

Outputting Tendencies
---------------------

UFS
^^^

When ``ldiag3d`` and ``qdiag3d`` are set to true, the tendencies described in
:numref:`Table %s <avail_tendencies>` are prepared for output. Finer control over which 
variables will actually be output is available through the diag table. The user must edit
the diag table and enter new lines at the end with the variables desired in the output. For
example, adding the line below results in the output of the temperature tendencies due to
long wave radiation:

.. code-block:: console

   "gfs_phys",    "dt3dt_lw",         "dt3dt_lw",         "fv3_history",  "all",  .false.,  "none",  2

Note that some host models, such as the UFS, have a limit of how many fields can be output in a run.
When outputting all tendencies, this limit may have to be increased. In the UFS, this limit is determined
by variable ``max_output_fields`` in namelist section ``&diag_manager_nml`` in file ``input.nml``. 

Further documentation of the ``diag_table`` file can be found in the UFS Weather Model User’s Guide
`here <https://ufs-weather-model.readthedocs.io/en/latest/InputsOutputs.html#diag-table-file>`_.

After running, the requested arrays will be present in the output files.  

SCM
^^^

The default behavior of the SCM is to output instantaneous values of all variables in
:numref:`Table %s <avail_tendencies>`. Tendencies are computed in file ``gmtb_scm_output.F90`` in
the subroutines output_init and output_append. If the values of ``ldiag3d`` or ``qdiag3d`` are set
to false, the variables are still written to output but are given missing values.

.. _AvailTendFAQ:

FAQ
---

What is the meaning of error message ``max_output_fields`` was exceeded?
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If the limit to the number of output fields is exceeded, the job may fail with the following message:
 
.. code-block:: console

   FATAL from PE    24: diag_util_mod::init_output_field: max_output_fields =          300 exceeded.  Increase via diag_manager_nml
 
In this case, increase ``max_output_fields`` in ``input.nml``:
 
.. code-block:: console

   &diag_manager_nml
       prepend_date = .F.
       max_output_fields = 600

Why did I run out of memory when outputting tendencies?
-------------------------------------------------------

Trying to output all tendencies may cause memory problems.  Choose your output variables carefully!

Why did I get a runtime logic error when outputting tendencies?
---------------------------------------------------------------

Setting ``ldiag3d=F`` and ``qdiag3d=T`` will result in an error message:
 
.. code-block:: console

   Logic error in GFS_typedefs.F90: qdiag3d requires ldiag3d
 
If you want to output specific humidity and/or ozone tendencies, you must set both ``ldiag3d`` and ``qdiag3d`` to T.

====================================
Output of Auxiliary Arrays from CCPP
====================================

The output of diagnostics from one or more parameterizations involves changes to the
namelist and code changes in the parameterization(s) (to load the desirable information
onto the CCPP-provided arrays and to add them to the subroutine arguments) and in the
parameterization metadata descriptor file(s) (to provide metadata on the new subroutine
arguments). In the UFS, the namelist is used to control the temporal averaging period.
These code changes are intended to be used by scientists during the development process
and are not intended to be incorporated into the master code. Therefore, developers
must remove any code related to these additional diagnostics before submitting a pull
request to the ccpp-physics repository.

The auxiliary diagnostics  from CCPP are output in arrays:

* aux2d  - auxiliary 2D array for outputting diagnostics
* aux3d  - auxiliary 3D array for outputting diagnostics

and dimensioned by:

* naux2d - number of 2D auxiliary arrays to output for diagnostics
* naux3d - number of 3D auxiliary arrays to output diagnostics

At runtime, these arrays will be written to the output files. Note that auxiliary
arrays can be output from more than one parameterization in a given run.

The UFS and SCM already contain code to declare and initialize the arrays:

* dimensions are declared and initialized in ``GFS_typedefs.F90``
* metadata for these arrays and dimensions are defined in ``GFS_typedefs.meta``
* arrays are populated in ``GFS_diagnostics.F90`` (UFS) or ``gmtb_scm_output.F90`` (SCM)

The remainder of this section describes changes the developer needs to make in the
physics code and  in the host model control files to enable the capability. An 
example (:numref:`Section %s  <CodeModExample>`) and FAQ (:numref:`Section %s <AuxArrayFAQ>`)
are also provided.

Enabling the capability
-----------------------

Physics-side changes
^^^^^^^^^^^^^^^^^^^^

In order to output auxiliary arrays, developers need to change at least the following
two files within the physics (see also example in :numref:`Section %s <CodeModExample>`):

* A CCPP entrypoint scheme
   * Add array(s) and its/their dimension(s) to the list of subroutine arguments
   * Declare array(s) with appropriate intent and dimension(s).  Note that array(s) do not
     need to be allocated by the developer.  This is done automatically in ``GFS_typedefs.F90``.
   * Populate array(s) with desirable diagnostic for output
* The file with metadata for modified scheme(s)
   * Add entries for the array(s) and its/their dimension(s) and provide metadata

Host-side changes
^^^^^^^^^^^^^^^^^

UFS
"""

For the UFS,  developers have to change the following two files on the host side (also see
example provided in :numref:`Section %s <CodeModExample>`)

* Namelist file ``input.nml``
   * Specify how many 2D and 3D arrays will be output using variables ``naux2d`` and ``naux3d``
     in section ``&gfs_physics_nml``, respectively. The maximum allowed number of arrays to
     output is 20 2D and 20 3D arrays.
   * Specify whether the output should be for instantaneous or time-averaged quantities using
     variables ``aux2d_time_avg`` and ``aux_3d_time_avg``. These arrays are dimensioned ``naux2d``
     and ``naux3d``, respectively, and, if not specified in the namelist, take the default value F.
   * Specify the period of averaging for the arrays using variable fhzero (in hours).
* File ``diag_table``
   * Enable output of the arrays at runtime.
   * 2D and 3D arrays are written to the output files.

SCM
"""

Typically, in a 3D model, 2D arrays represent variables with two horizontal dimensions, e.g. x
and y, whereas 3D arrays represent variables with all three spatial dimensions, e.g. x, y, and z.
For the SCM, these arrays are implicitly 1D and 2D, respectively, where the “y” dimension is 1
and the “x” dimension represents the number of independent columns (typically also 1). For
continuity with the UFS Atmosphere, the naming convention 2D and 3D are retained, however.
With this understanding, the namelist files can be modified as in the UFS:
 
* Namelist file ``input.nml``
   * Specify how many 2D and 3D arrays will be output using variables ``naux2d`` and ``naux3d``
     in section ``&gfs_physics_nml``, respectively. The maximum allowed number of arrays to
     output is 20 2D and 20 3D arrays.
   * Unlike the UFS, only instantaneous values are output. Time-averaging can be done through
     post-processing the output. Therefore, the values of ``aux2d_time_avg`` and ``aux_3d_time_avg``
     should not be changed from their default false values. As such, the namelist variable ``fhzero``
     has no effect in the SCM.

.. _CodeModExample:

Recompiling and Examples
------------------------

The developer must recompile the code after making the source code changes to the CCPP scheme(s)
and associated metadata files. Changes in the namelist and diag table can be made after compilation.
At compile and runtime, the developer must pick suites that use the scheme from which output is desired.
 
An example for how to output auxiliary arrays is provided in the rest of this section. The lines that
start with “+” represent lines that were added by the developer to output the diagnostic arrays. In
this example, the developer modified the Grell-Freitas (GF) cumulus scheme to output two 2D arrays
and one 3D array. The 2D arrays are ``aux_2d (:,1)`` and ``aux_2d(:,2)``; the 3D array is ``aux_3d(:,:,1)``.
The 2D array ``aux2d(:,1)`` will be output with an averaging in time in the UFS, while the ``aux2d(:,2)``
and ``aux3d`` arrays will not be averaged. 

In this example, the arrays are populated with bogus information just to demonstrate the capability.
In reality, a developer would populate the array with the actual quantity for which output is desirable. 

.. code-block:: console

   diff --git a/physics/cu_gf_driver.F90 b/physics/cu_gf_driver.F90
   index 927b452..aed7348 100644
   --- a/physics/cu_gf_driver.F90
   +++ b/physics/cu_gf_driver.F90
   @@ -76,7 +76,8 @@ contains
                   flag_for_scnv_generic_tend,flag_for_dcnv_generic_tend,           &
                   du3dt_SCNV,dv3dt_SCNV,dt3dt_SCNV,dq3dt_SCNV,                     &
                   du3dt_DCNV,dv3dt_DCNV,dt3dt_DCNV,dq3dt_DCNV,                     &
   -               ldiag3d,qdiag3d,qci_conv,errmsg,errflg)
   +               ldiag3d,qdiag3d,qci_conv,errmsg,errflg,                          &
   +               naux2d,naux3d,aux2d,aux3d)
    !-------------------------------------------------------------
          implicit none
          integer, parameter :: maxiens=1
   @@ -137,6 +138,11 @@ contains
       integer, intent(in   ) :: imfshalcnv
       character(len=*), intent(out) :: errmsg
       integer,          intent(out) :: errflg
   +
   +   integer, intent(in) :: naux2d,naux3d
   +   real(kind_phys), intent(inout) :: aux2d(:,:)
   +   real(kind_phys), intent(inout) :: aux3d(:,:,:)
   +
    !  define locally for now.
       integer, dimension(im),intent(inout) :: cactiv
       integer, dimension(im) :: k22_shallow,kbcon_shallow,ktop_shallow
   @@ -199,6 +205,11 @@ contains
      ! initialize ccpp error handling variables
         errmsg = ''
         errflg = 0
   +
   +     aux2d(:,1) = aux2d(:,1) + 1
   +     aux2d(:,2) = aux2d(:,2) + 2
   +     aux3d(:,:,1) = aux3d(:,:,1) + 3
   +
    !
    ! Scale specific humidity to dry mixing ratio
    !

The ``cu_gf_driver.meta`` file was modified accordingly:

.. code-block:: console

   diff --git a/physics/cu_gf_driver.meta b/physics/cu_gf_driver.meta
   index 99e6ca6..a738721 100644
   --- a/physics/cu_gf_driver.meta
   +++ b/physics/cu_gf_driver.meta
   @@ -476,3 +476,29 @@
      type = integer
      intent = out
      optional = F
   +[naux2d]
   +  standard_name = number_of_2d_auxiliary_arrays
   +  long_name = number of 2d auxiliary arrays to output (for debugging)
   +  units = count
   +  dimensions = ()
   +  type = integer
   +[naux3d]
   +  standard_name = number_of_3d_auxiliary_arrays
   +  long_name = number of 3d auxiliary arrays to output (for debugging)
   +  units = count
   +  dimensions = ()
   +  type = integer
   +[aux2d]
   +  standard_name = auxiliary_2d_arrays
   +  long_name = auxiliary 2d arrays to output (for debugging)
   +  units = none
   +  dimensions = (horizontal_dimension,number_of_3d_auxiliary_arrays)
   +  type = real
   +  kind = kind_phys
   +[aux3d]
   +  standard_name = auxiliary_3d_arrays
   +  long_name = auxiliary 3d arrays to output (for debugging)
   +  units = none
   +  dimensions = (horizontal_dimension,vertical_dimension,number_of_3d_auxiliary_arrays)
   +  type = real
   +  kind = kind_phys

The following lines were added to the ``&gfs_physics_nml`` section of the namelist file ``input.nml``:
 
.. code-block:: console

       naux2d         = 2
       naux3d         = 1
       aux2d_time_avg = .true., .false.

Recall that for the SCM, ``aux2d_time_avg`` should not be set to true in the namelist.
 
Lastly, the following lines were added to the ``diag_table`` for UFS:
 
.. code-block:: console

   # Auxiliary output
   "gfs_phys",    "aux2d_01",     "aux2d_01",      "fv3_history2d",  "all",  .false.,  "none",  2
   "gfs_phys",    "aux2d_02",     "aux2d_02",      "fv3_history2d",  "all",  .false.,  "none",  2
   "gfs_phys",    "aux3d_01",     "aux3d_01",      "fv3_history",    "all",  .false.,  "none",  

.. _AuxArrayFAQ:

FAQ
^^^

How do I enable the output of diagnostic arrays from multiple parameterizations in a single run?
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

Suppose you want to output two 2D arrays from schemeA and two 2D arrays from schemeB. You should
set the namelist to ``naux2d=4`` and ``naux3d=0``. In the code for schemeA, you should populate
``aux2d(:,1)`` and ``aux2d(:,2)``, while in the code for scheme B you should populate ``aux2d(:,3)``
and ``aux2d(:,4)``. 

