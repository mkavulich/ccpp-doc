.. _Host-side Coding:

**************************************************
Host Side Coding
**************************************************

This chapter describes the connection of a host model with the pool of :term:`CCPP-Physics` schemes through the :term:`CCPP-Framework`. 

In several places, references are made to an Interoperable Physics Driver (IPD). The IPD was originally developed by EMC and later expanded by NOAA GFDL with the goal of connecting GFS physics to various models. A top motivation for its development was the dycore test that led to the selection of FV3 as the dycore for the :term:`UFS`. Designed in a fundamentally different way than the :term:`CCPP`, the IPD will be phased out in the near future in favor of the CCPP as a single way to interface with physics in the UFS. To enable a smooth transition, several of the CCPP components must interact with the IPD and, as such, parts of the CCPP code in the UFS currently carry the tag “IPD”.

==================================================
Variable Requirements on the Host Model Side
==================================================

All variables required to communicate between the host model and the physics, as well as to communicate between physics schemes, need to be allocated by the host model. An exception is variables ``errflg``, ``errmsg``, ``loop_cnt``, ``blk_no``, and ``thrd_no``, which are allocated by the CCPP-Framework, as explained in :numref:`Section %s <DataStructureTransfer>`. A list of all variables required for the current pool of physics can be found in ``ccpp/framework/doc/DevelopersGuide/CCPP_VARIABLES_XYZ.pdf`` (XYZ: SCM, FV3). 

At present, only two types of variable definitions are supported by the CCPP-Framework:

* Standard Fortran variables (character, integer, logical, real) defined in a module or in the main program. For character variables, a fixed length is required. All others can have a kind attribute of a kind type defined by the host model.
* Derived data types (DDTs) defined in a module or the main program. While the use of DDTs as arguments to physics schemes in general is discouraged (see :numref:`Section %s <IOVariableRules>`), it is perfectly acceptable for the host model to define the variables requested by physics schemes as components of DDTs and pass these components to CCPP by using the correct local_name (e.g., ``myddt%thecomponentIwant``; see :numref:`Section %s <VariableTablesHostModel>`.)

.. _VariableTablesHostModel:

==================================================
Metadata for Variable in the Host Model
==================================================

To establish the link between host model variables and physics scheme variables, the host model must provide metadata information similar to those presented in :numref:`Section %s <MetadataRules>`. The host model can have multiple metadata files (``.meta``), each with the required ``[ccpp-table-properties]`` section and the related ``[ccpp-arg-table]`` sections. The host model Fortran files contain three-line snippets to indicate the location for insertion of the metadata information contained in the corresponding section in the ``.meta`` file.

.. _SnippetMetadata:

.. code-block:: fortran

   !!> \section arg_table_example_vardefs
   !! \htmlinclude example_vardefs.html
   !!

For each variable required by the pool of CCPP-Physics schemes, one and only one entry must exist on the host model side. The connection between a variable in the host model and in the physics scheme is made through its ``standard_name``.

The following requirements must be met when defining metadata for variables in the host model (see also :ref:`Listing 6.1 <example_vardefs>` 
and :ref:`Listing 6.2 <example_vardefs_meta>` for examples of host model metadata).

* The ``standard_name`` must match that of the target variable in the physics scheme.
* The type, kind, shape and size of the variable (as defined in the host model Fortran code) must match that of the target variable.
* The attributes ``units``, ``rank``, ``type`` and ``kind`` in the host model metadata must match those in the physics scheme metadata.
* The attribute ``active`` is used to allocate variables under certain conditions.  It must be written as a Fortran expression that equates to ``.true.`` or ``.false.``, using the CCPP standard names of variables. ``active`` attributes for all variables are ``.true.`` by default. See :numref:`Section %s <ActiveAttribute>` for details.
* The attributes ``optional`` and ``intent`` must be set to ``F`` and ``none``, respectively.
* The ``local_name`` of the variable must be set to the name the host model cap uses to refer to the variable.
* The metadata section that exposes a DDT to the CCPP (as opposed to the section that describes the components of a DDT) must be in the same module where the memory for the DDT is allocated. If the DDT is a module variable, then it must be exposed via the module’s metadata section, which must have the same name as the module.
* Metadata sections describing module variables must be placed inside the module.
* Metadata sections describing components of DDTs must be placed immediately before the type definition and have the same name as the DDT.

.. _example_vardefs:

.. code-block:: fortran

       module example_vardefs
 
         implicit none

   !!> \section arg_table_example_vardefs
   !! \htmlinclude example_vardefs.html
   !!

         integer, parameter           :: r15 = selected_real_kind(15)
         integer                      :: ex_int
         real(kind=8), dimension(:,:) :: ex_real1
         character(len=64)            :: errmsg
         logical                      :: errflg

   !!> \section arg_table_example_ddt
   !! \htmlinclude example_ddt.html
   !!
 
         type ex_ddt
           logical              :: l
           real, dimension(:,:) :: r
         end type ex_ddt
    
         type(ex_ddt) :: ext
    
       end module example_vardefs


*Listing 6.1: Example host model file with reference to metadata. In this example, both the definition and the declaration (memory allocation) of a DDT* ``ext`` *(of type* ``ex_ddt`` *) are in the same module.*

.. _example_vardefs_meta:

.. code-block:: fortran

   ########################################################################
   [ccpp-table-properties]
     name = arg_table_example_vardefs
     type = module

   [ccpp-arg-table]
     name = arg_table_example_vardefs
     type = module
   [ex_int]
     standard_name = example_int 
     long_name = ex. int
     units = none
     dimensions = () 
     type = integer
     kind = 
   [ex_real]
     standard_name = example_real
     long_name = ex. real
     units = m
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = kind=8
   [ex_ddt]
     standard_name = example_ddt
     long_name = ex. ddt
     units = DDT
     dimensions = (horizontal_dimension,vertical_dimension)
     type = ex_ddt
     kind =
   [ext]
     standard_name = example_ddt_instance
     long_name = ex. ddt inst
     units = DDT
     dimensions = (horizontal_dimension,vertical_dimension)
     type = ex_ddt
     kind =
   [errmsg]
     standard_name = ccpp_error_message
     long_name = error message for error handling in CCPP
     units = none
     dimensions = ()
     type = character
     kind = len=64
   [errflg]
     standard_name = ccpp_error_flag
     long_name = error flag for error handling in CCPP
     units = flag
     dimensions = ()
     type = integer

   ########################################################################
   [ccpp-table-properties]
     name = arg_table_example_ddt
     type = ddt

   [ccpp-arg-table]
     name = arg_table_example_ddt
     type = ddt
   [ext%1]
     standard_name = example_flag
     long_name = ex. flag
     units = flag
     dimensions = 
     type = logical
     kind =
   [ext%r]
     standard_name = example_real3
     long_name = ex. real
     units = kg
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = r15
   [ext%r(;,1)]
     standard_name = example_slice
     long_name = ex. slice
     units = kg
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = r15
   [nwfa2d]
     standard_name = tendency_of_water_friendly_aerosols_at_surface
     long_name = instantaneous water-friendly sfc aerosol source
     units = kg-1 s-1
     dimensions = (horizontal_dimension)
     type = real
     kind = kind_phys
     active = (flag_for_microphysics_scheme == flag_for_thompson_microphysics_scheme .and. flag_for_aerosol_physics)
   [qgrs(:,:,index_for_water_friendly_aerosols)]
     standard_name = water_friendly_aerosol_number_concentration
     long_name = number concentration of water-friendly aerosols
     units = kg-1
     dimensions = (horizontal_dimension,vertical_dimension)
     active = (index_for_water_friendly_aerosols > 0)
     type = real
     kind = kind_phys

*Listing 6.2: Example host model metadata file (* ``.meta`` *).*

.. _ActiveAttribute:

,,,,,,,,,,,,,,,,
Active Attribute
,,,,,,,,,,,,,,,,

The CCPP must be able to detect when arrays need to be allocated, and when certain tracers must be
present in order to perform operations or tests in the auto-generated caps (e.g. unit conversions,
blocked data structure copies, etc.). This is accomplished with the attribute ``active`` in the
metadata for the host model variables (``GFS_typedefs.meta`` for the UFS Atmosphere or the SCM).

Several arrays in the host model (e.g., ``GFS_typedefs.F90`` in the UFS Atmosphere or the SCM) are
allocated based on certain conditions, for example:

.. code-block:: fortran

    !--- needed for Thompson's aerosol option
    if(Model%imp_physics == Model%imp_physics_thompson .and. Model%ltaerosol) then
      allocate (Coupling%nwfa2d (IM))
      allocate (Coupling%nifa2d (IM))
      Coupling%nwfa2d   = clear_val
      Coupling%nifa2d   = clear_val
    endif

Other examples are the elements in the tracer array, where their presence depends on the corresponding
index being larger than zero. For example:

.. code-block:: fortran

    integer              :: ntwa            !< tracer index for water friendly aerosol
    ...
    Model%ntwa             = get_tracer_index(Model%tracer_names, 'liq_aero', ...)      
    ...
    if (Model%ntwa>0) then
      ! do something with qgrs(:,:,Model%ntwa)
    end if

The ``active`` attribute is a conditional statement that, if true, will allow the corresponding variable
to be allocated.  It must be written as a Fortran expression that equates to ``.true.`` or ``.false.``,
using the CCPP standard names of variables. Active attributes for all variables are ``.true.`` by default. 

If a developer adds a new variable that is only allocated under certain conditions, or changes the conditions
under which an existing variable is allocated, a corresponding change must be made in the metadata for the
host model variables (``GFS_typedefs.meta`` for the UFS Atmosphere or the SCM). See variables ``nwfa2d``
and ``qgrs`` in :ref:`Listing 6.2 <example_vardefs_meta>` for an example.

========================================================
CCPP Variables in the SCM and UFS Atmosphere Host Models
========================================================

While the use of standard Fortran variables is preferred, in the current implementation of the CCPP in the UFS Atmosphere and in the SCM almost all data is contained in DDTs for organizational purposes. In the case of the SCM, DDTs are defined in ``gmtb_scm_type_defs.f90`` and ``GFS_typedefs.F90``, and in the case of the UFS Atmosphere, they are defined in both ``GFS_typedefs.F90`` and ``CCPP_typedefs.F90``.  The current implementation of the CCPP in both host models uses the following set of DDTs:

* ``GFS_init_type`` 		variables to allow proper initialization of GFS physics
* ``GFS_statein_type``	prognostic state data provided by dycore to physics
* ``GFS_stateout_type``	prognostic state after physical parameterizations
* ``GFS_sfcprop_type``	surface properties read in and/or updated by climatology, obs, physics
* ``GFS_coupling_type``	fields from/to coupling with other components, e.g., land/ice/ocean
* ``GFS_control_type``	control parameters input from a namelist and/or derived from others
* ``GFS_grid_type``		grid data needed for interpolations and length-scale calculations
* ``GFS_tbd_type``		data not yet assigned to a defined container
* ``GFS_cldprop_type``	cloud properties and tendencies needed by radiation from physics
* ``GFS_radtend_type``	radiation tendencies needed by physics
* ``GFS_diag_type``		fields targeted for diagnostic output to disk
* ``GFS_interstitial_type``	fields used to communicate variables among schemes in the slow physics group required to replace interstitial code in ``GFS_{physics, radiation}_driver.F90`` in CCPP
* ``GFS_data_type``	combined type of all of the above except ``GFS_control_type`` and ``GFS_interstitial_type``
* ``CCPP_interstitial_type`` fields used to communicate variables among schemes in the fast physics group

The DDT descriptions provide an idea of what physics variables go into which data type.  ``GFS_diag_type`` can contain variables that accumulate over a certain amount of time and are then zeroed out. Variables that require persistence from one timestep to another should not be included in the ``GFS_diag_type`` nor the ``GFS_interstitial_type`` DDTs. Similarly, variables that need to be shared between groups cannot be included in the ``GFS_interstitial_type`` DDT. Although this memory management is somewhat arbitrary, new variables provided by the host model or derived in an interstitial scheme should be put in a DDT with other similar variables.

Each DDT contains a create method that allocates the data defined using the metadata. For example, the ``GFS_stateout_type`` contains:

.. code-block:: fortran

 type GFS_stateout_type

    !-- Out (physics only)
    real (kind=kind_phys), pointer :: gu0 (:,:)   => null()  !< updated zonal wind
    real (kind=kind_phys), pointer :: gv0 (:,:)   => null()  !< updated meridional wind
    real (kind=kind_phys), pointer :: gt0 (:,:)   => null()  !< updated temperature
    real (kind=kind_phys), pointer :: gq0 (:,:,:) => null()  !< updated tracers

    contains
      procedure :: create  => stateout_create  !<   allocate array data
  end type GFS_stateout_type

In this example, ``gu0``, ``gv0``, ``gt0``, and ``gq0`` are defined in the host-side metadata section, and when the subroutine ``stateout_create`` is called, these arrays are allocated and initialized to zero.  With the CCPP, it is possible to not only refer to components of DDTs, but also to slices of arrays with provided metadata as long as these are contiguous in memory. An example of an array slice from the ``GFS_stateout_type`` looks like:

.. code-block:: fortran

  ########################################################################
  [ccpp-table-properties]
     name = GFS_stateout_type
     type = ddt
     dependencies =

   [ccpp-arg-table]
     name = GFS_stateout_type
     type = ddt
   [gq0(:,:,index_for_snow_water)]
     standard_name = snow_water_mixing_ratio_updated_by_physics
     long_name = moist (dry+vapor, no condensates) mixing ratio of snow water updated by physics
     units = kg kg-1
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = kind_phys

Array slices can be used by physics schemes that only require certain values from an array. 

.. _CCPP_API:

========================================================
CCPP API 
========================================================

The CCPP Application Programming Interface (API) is comprised of a set of clearly defined methods used to communicate variables between the host model and the physics and to run the physics. The bulk of the CCPP API is located in the CCPP-Framework, and is described in file ``ccpp_static_api.F90``.  Subroutines ``ccpp_physics_init``, ``ccpp_physics_finalize``, and ``ccpp_physics_run`` (described below) are contained in ``ccpp_static_api.F90``.  ``ccpp_static_api.F90`` is auto-generated when the script ``ccpp_prebuild.py`` is run for the build.

.. _DataStructureTransfer:

,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Data Structure to Transfer Variables between Dynamics and Physics 
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

The ``cdata`` structure is used for holding five variables that must always be available to the physics schemes. These variables are listed in a metadata table in ``ccpp/framework/src/ccpp_types.meta`` (:ref:`Listing 6.3 <MandatoryVariables>`). 


* Error flag for handling in CCPP (``errmsg``).
* Error message associated with the error flag (``errflg``).
* Loop counter for subcycling loops (``loop_cnt``).
* Number of block for explicit data blocking in CCPP (``blk_no``).
* Number of thread for threading in CCPP (``thrd_no``).

.. _MandatoryVariables:

.. code-block:: fortran
  [ccpp-table-properties]
    name = ccpp_types
    type = module
    dependencies =

  [ccpp-arg-table]
    name = ccpp_types
    type = module
  [ccpp_t]
    standard_name = ccpp_t
    long_name = definition of type ccpp_t
    units = DDT
    dimensions = ()
    type = ccpp_t
  
  ########################################################################
  [ccpp-table-properties]
    name = ccpp_t
    type = ddt 
    dependencies =
    
  [ccpp-arg-table]
    name = ccpp_t 
    type = ddt
  [errflg]
    standard_name = ccpp_error_flag
    long_name = error flag for error handling in CCPP
    units = flag
    dimensions = () 
    type = integer
  [errmsg]
    standard_name = ccpp_error_message
    long_name = error message for error handling in CCPP
    units = none
    dimensions = () 
    type = character
    kind = len=512
  [loop_cnt]
    standard_name = ccpp_loop_counter
    long_name = loop counter for subcycling loops in CCPP
    units = index
    dimensions = ()
    type = integer
  [blk_no]
    standard_name = ccpp_block_number
    long_name = number of block for explicit data blocking in CCPP
    units = index
    dimensions = ()
    type = integer
  [thrd_no]
    standard_name = ccpp_thread_number
    long_name = number of thread for threading in CCPP
    units = index
    dimensions = ()
    type = integer

*Listing 6.3: Mandatory variables provided by the CCPP-Framework from* ``ccpp/framework/src/ccpp_types.meta`` *.
These variables must not be defined by the host model.*

Two of the variables are mandatory and must be passed to every physics scheme: ``errmsg`` and ``errflg``. The variables ``loop_cnt``, ``blk_no``, and ``thrd_no`` can be passed to the schemes if required, but are not mandatory.  The ``cdata`` structure is only used to hold these five variables, since the host model variables are directly passed to the physics without the need for an intermediate data structure.

Note that ``cdata`` is not restricted to being a scalar but can be a multidimensional array, depending on the needs of the host model. For example, a model that uses a one-dimensional array of blocks for better cache-reuse may require ``cdata`` to be a one-dimensional array of the same size. Another example of a multi-dimensional array of ``cdata`` is in the SCM, which uses a one-dimensional cdata array for N independent columns. 

Due to a restriction in the Fortran language, there are no standard pointers that are generic pointers, such as the C language allows. The CCPP system therefore has an underlying set of pointers in the C language that are used to point to the original data within the host application cap. The user does not see this C data structure, but deals only with the public face of the Fortran ``cdata`` DDT. The type ``ccpp_t`` is defined in ``ccpp/framework/src/ccpp_types.meta`` and declared in ``ccpp/framework/src/ccpp_types.F90``.

,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Initializing and Finalizing the CCPP
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

At the beginning of each run, the ``cdata`` structure needs to be set up. Similarly, at the end of each run, it needs to be terminated. This is done with subroutines ``ccpp_init`` and ``ccpp_finalize``. These subroutines should not be confused with ``ccpp_physics_init`` and ``ccpp_physics_finalize``, which were described in :numref:`Chapter %s <SuiteGroupCaps>`.

Note that optional arguments are denoted with square brackets.

.. _SuiteInitSubroutine:

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Suite Initialization Subroutine 	
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The suite initialization subroutine, ``ccpp_init``, takes three mandatory and two optional arguments. The mandatory arguments are the name of the suite (of type character), the name of the ``cdata`` variable that must be allocated at this point, and an integer used for the error status. Note that the suite initialization routine ``ccpp_init`` parses the SDF corresponding to the given suite name and initializes the state of the suite and its schemes. This process must be repeated for every element of a multi-dimensional ``cdata``. For performance reasons, it is possible to avoid repeated reads of the SDF and to have a single state of the suite shared between the elements of ``cdata``. To do so, specify an optional argument variable called ``cdata_target = X`` in the call to ``ccpp_init``, where ``X`` refers to the instance of ``cdata`` that has already been initialized.

For a given suite name ``XYZ``, the name of the suite definition file is inferred as ``suite_XYZ.xml``, and the file is expected to be present in the current run directory. It is possible to specify the optional argument ``is_filename=.true.`` to ``ccpp_init``, which will treat the suite name as an actual file name (with or without the path to it).

Typical calls to ``ccpp_init`` are below, where ``ccpp_suite`` is the name of the suite, and ``ccpp_sdf_filepath`` the actual SDF filename, with or without a path to it.

.. code-block:: fortran

 call ccpp_init(trim(ccpp_suite), cdata, ierr)
 call ccpp_init(trim(ccpp_suite), cdata2, ierr, [cdata_target=cdata])
 call ccpp_init(trim(ccpp_sdf_filepath), cdata, ierr, [is_filename=.true.])

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Suite Finalization Subroutine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The suite finalization subroutine, ``ccpp_finalize``, takes two arguments, the name of the ``cdata`` variable that must be de-allocated at this point, and an integer used for the error status. A typical call to ``ccpp_finalize`` is below:

.. code-block:: fortran

 call ccpp_finalize(cdata, ierr)

If a specific data instance was used in a call to ``ccpp_init``, as in the above example in :numref:`Section %s <SuiteInitSubroutine>`, then this data instance must be finalized last:

.. code-block:: fortran

 call ccpp_finalize(cdata2, ierr)
 call ccpp_finalize(cdata, ierr)

,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Running the Physics
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

The physics is invoked by calling subroutine ``ccpp_physics_run``. This subroutine is part of the CCPP API and is auto-generated. This subroutine is capable of executing the physics with varying granularity, that is, a single group, or an entire suite can be run with a single subroutine call. Typical calls to ``ccpp_physics_run`` are below,where ``suite_name`` is mandatory and ``group_name`` is optional:

.. code-block:: fortran

 call ccpp_physics_run(cdata, suite_name, [group_name], ierr=ierr)

,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
Initializing and Finalizing the Physics
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

Many (but not all) physical parameterizations need to be initialized, which includes functions such as reading lookup tables, reading input datasets, computing derived quantities, broadcasting information to all MPI ranks, etc. Initialization procedures are typically done for the entire domain, that is, they are not subdivided by blocks. Similarly, many (but not all) parameterizations need to be finalized, which includes functions such as deallocating variables, resetting flags from *initialized* to *non-initiaIized*, etc. Initialization and finalization functions are each performed once per run, before the first call to the physics and after the last call to the physics, respectively.

The initialization and finalization can be invoked for a single group, or for the entire suite. In both cases, subroutines ``ccpp_physics_init`` and ``ccpp_physics_finalize`` are used and the arguments passed to those subroutines determine the type of initialization.

These subroutines should not be confused with ``ccpp_init`` and ``ccpp_finalize``, which were explained previously.

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Subroutine ``ccpp_physics_init``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This subroutine is part of the CCPP API and is auto-generated. It cannot contain thread-dependent information but can have block-dependent information. A typical call to ``ccpp_physics_init`` is:

.. code-block:: fortran

 call ccpp_physics_init(cdata, suite_name, [group_name], ierr=ierr)

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Subroutine ``ccpp_physics_finalize``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This subroutine is part of the CCPP API and is auto-generated. A typical call to ``ccpp_physics_finalize`` is:

.. code-block:: fortran

 call ccpp_physics_finalize(cdata, suite_name, [group_name], ierr=ierr)

========================================================
Host Caps
========================================================

The purpose of the host model *cap* is to abstract away the communication between the host model and the CCPP-Physics schemes. While CCPP calls can be placed directly inside the host model code (as is done for the relatively simple SCM), it is recommended to separate the *cap* in its own module for clarity and simplicity (as is done for the UFS Atmosphere). While the details of implementation will be specific to each host model, the host model *cap* is responsible for the following general functions:

* Allocating memory for variables needed by physics

  * All variables needed to communicate between the host model and the physics, and all variables needed to communicate among physics schemes, need to be allocated by the host model. The latter, for example for interstitial variables used exclusively for communication between the physics schemes, are typically allocated in the *cap*. 


* Allocating the ``cdata`` structure(s)					


* Calling the suite initialization subroutine				

  * The suite must be initialized using ``ccpp_init``.

* Populating the ``cdata`` structure(s)

  * The autogenerated caps for the physics (groups and suite caps) are automatically given memory access to
    the host model variables and they can be used directly, without the need for a data structure containing
    pointers to the actual variables (which is what cdata is).

* Providing interfaces to call the CCPP

  * The *cap* must provide functions or subroutines that can be called at the appropriate places in the host model time integration loop and that internally call ``ccpp_init``, ``ccpp_physics_init``, ``ccpp_physics_run``, ``ccpp_physics_finalize`` and ``ccpp_finalize``, and handle any errors returned See :ref:`Listing 6.4 <example_ccpp_host_cap>`. 

.. _example_ccpp_host_cap:

.. code-block:: fortran
 
 module example_ccpp_host_cap
  
  use ccpp_api,           only: ccpp_t, ccpp_init, ccpp_finalize
  use ccpp_static_api,    only: ccpp_physics_init, ccpp_physics_run,     &
                                ccpp_physics_finalize

   implicit none
   ! CCPP data structure
   type(ccpp_t), save, target :: cdata
   public :: physics_init, physics_run, physics_finalize
 contains
  
  subroutine physics_init(ccpp_suite_name)
    character(len=*), intent(in) :: ccpp_suite_name
    integer :: ierr
    ierr = 0

    ! Initialize the CCPP framework, parse SDF
    call ccpp_init(trim(ccpp_suite_name), cdata, ierr=ierr)
    if (ierr/=0) then
      write(*,'(a)') "An error occurred in ccpp_init"
      stop
    end if

    ! Initialize CCPP physics (run all _init routines)
    call ccpp_physics_init(cdata, suite_name=trim(ccpp_suite_name),      &
                           ierr=ierr)
    ! error handling as above

  end subroutine physics_init

  subroutine physics_run(ccpp_suite_name, group)
    ! Optional argument group can be used to run a group of schemes      &
    ! defined in the SDF. Otherwise, run entire suite.
    character(len=*),           intent(in) :: ccpp_suite_name
    character(len=*), optional, intent(in) :: group

    integer :: ierr
    ierr = 0

    if (present(group)) then
       call ccpp_physics_run(cdata, suite_name=trim(ccpp_suite_name),    &
                             group_name=group, ierr=ierr)
    else
       call ccpp_physics_run(cdata, suite_name=trim(ccpp_suite_name),    &
                             ierr=ierr)
    end if
    ! error handling as above

  end subroutine physics_run

  subroutine physics_finalize(ccpp_suite_name)
    character(len=*), intent(in) :: ccpp_suite_name
    integer :: ierr
    ierr = 0

    ! Finalize CCPP physics (run all _finalize routines)
    call ccpp_physics_finalize(cdata, suite_name=trim(ccpp_suite_name),  &
                               ierr=ierr)
    ! error handling as above
    call ccpp_finalize(cdata, ierr=ierr)
    ! error handling as above

  end subroutine physics_finalize

 end module example_ccpp_host_cap

*Listing 6.4: Fortran template for a CCPP host model cap from* ``ccpp/framework/doc/DevelopersGuide/host_cap_template.F90``.

The following sections describe two implementations of host model caps to serve as examples. For each of the functions listed above, a description for how it is implemented in each host model is included.

,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
SCM Host Cap
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

The cap functions for the SCM are mainly implemented in:

``gmtb-scm/scm/src/gmtb_scm.F90``

With smaller parts in:

``gmtb-scm/scm/src/gmtb_scm_type_defs.f90``

``gmtb-scm/scm/src/gmtb_scm_setup.f90``

``gmtb-scm/scm/src/gmtb_scm_time_integration.f90``


The host model *cap* is responsible for:

* Allocating memory for variables needed by physics 

  All variables and constants required by the physics have metadata provided on the host-side, ``arg_table_physics_type`` and ``arg_table_gmtb_scm_physical_constants``, which are implemented in ``gmtb_scm_type_defs.f90`` and ``gmtb_scm_physical_constants.f90``. To mimic the UFS Atmosphere and to hopefully reduce code maintenance, currently, the SCM uses GFS DDTs as sub-types within the physics DDT.

  In ``gmtb_scm_type_defs.f90``, the physics DDT has a create type-bound procedure (see subroutine ``physics_create`` and ``type physics_type``), which allocates GFS sub-DDTs and other physics variables and initializes them with zeros. ``physics%create`` is called from ``gmtb_scm.F90`` after the initial SCM state has been set up.

* Allocating the cdata structure 

  The SCM uses a one-dimensional ``cdata`` array for N independent columns, i.e. in ``gmtb_scm.F90``:

  ``allocate(cdata(scm_state%n_cols))``

* Calling the suite initialization subroutine 

  Within ``scm_state%n_cols`` loop in ``gmtb_scm.F90`` after initial SCM state setup and before first timestep, the suite initialization subroutine ``ccpp_init`` is called for each column with own instance of ``cdata``, and takes three arguments, the name of the runtime SDF, the name of the ``cdata`` variable that must be allocated at this point, and ``ierr``. 
 
* Populating the cdata structure 

  Within the same ``scm_state%n_cols`` loop, but after the ``ccpp_init`` call, the ``cdata`` structure is filled in with real initialized values:

 * ``physics%Init_parm`` (GFS DDT for setting up suite) are filled in from ``scm_state%``

 * call ``GFS_suite_setup()``: similar to ``GFS_initialize()`` in the UFS Atmosphere, is called and includes:

  * ``%init/%create`` calls for GFS DDTs

  * initialization for other variables in physics DDT

  * init calls for legacy non-ccpp schemes

 * call ``physics%associate()``: to associate pointers in physics DDT with targets in ``scm_state``, which contains variables that are modified by the SCM “dycore” (i.e. forcing).

  This include file is auto-generated from ``ccpp/scripts/ccpp_prebuild.py``, which parses tables in ``gmtb_scm_type_defs.f90``.

* Providing interfaces to call the CCPP

 * Calling ``ccpp_physics_init()``

  Within the same ``scm_state%n_cols`` loop but after ``cdata`` is filled, the physics initialization routines (``*_init()``) associated with the physics suite, group, and/or schemes are called at each column.

 * Calling ``ccpp_physics_run()``

  At the first timestep, if the forward scheme is selected (i.e. ``scm_state%time_scheme == 1``), call ``do_time_step()`` to apply forcing and ``ccpp_physics_run()`` calls at each column; if the leapfrog scheme is selected (i.e. ``scm_state%time_scheme == 2``), call ``ccpp_physics_run()`` directly at each column.

  At a later time integration, call ``do_time_step()`` to apply forcing and ``ccpp_physics_run()`` calls at each column. Since there is no need to execute anything between physics groups in the SCM, the ``ccpp_physics_run`` call is only given cdata and an error flag as arguments.

 * Calling ``ccpp_physics_finalize()`` and ``ccpp_finalize()``

  ``ccpp_physics_finalize()`` and ``ccpp_finalize()`` are called after the time loop at each column.

,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,
UFS Atmosphere Host Cap
,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,,

This section describes how the host cap is implemented for the UFS Atmosphere build.
For the build that uses CCPP:

.. code-block:: fortran

 #ifdef CCPP
 #endif

* Allocating memory for variables needed by physics

* Allocating the ``cdata`` structures

 * For the current implementation of the UFS Atmosphere, which uses a subset of fast physics processes tightly coupled to the dynamical core, three instances of ``cdata`` exist within the host model: ``cdata_tile`` to hold data for the fast physics, ``cdata_domain`` to hold data needed for all UFS Atmosphere blocks for the slow physics, and ``cdata_block``, an array of ``cdata`` DDTs with dimensions of (``number of blocks``, ``number of threads``) to contain data for individual block/thread combinations for the slow physics. All are defined as module-level variables in the ``CCPP_data module`` of ``CCPP_data.F90``. The ``cdata_block`` array is allocated (since the number of blocks and threads is unknown at compile-time) as part of the ``‘init’`` step of the ``CCPP_step subroutine`` in ``CCPP_driver.F90``. Note: Although the ``cdata`` containers are not used to hold the pointers to the physics variables, they are still used to hold other CCPP-related information.

* Calling the suite initialization subroutine

 * Corresponding to the three instances of ``cdata`` described above, the ``ccpp_init`` subroutine is called within three different contexts, all originating from the ``atmos_model_init`` subroutine of ``atmos_model.F90``:

  * For ``cdata_tile`` (used for the fast physics), the ``ccpp_init`` call is made from the ``atmosphere_init`` subroutine of ``atmosphere.F90``. Note: when fast physics is used, this is the *first* call to ``ccpp_init``, so it reads in the SDF and initializes the suite in addition to setting up the fields for ``cdata_tile``.

  * For ``cdata_domain`` and ``cdata_block`` used in the rest of the physics, the ‘init’ step of the ``CCPP_step`` subroutine in ``CCPP_driver.F90`` is called. Within that subroutine, ``ccpp_init`` is called once to set up ``cdata_domain`` and within a loop for every block/thread combination to set up the components of the ``cdata_block`` array. Note: as mentioned in the CCPP API :numref:`Section %s <CCPP_API>`, when fast physics is used, the SDF has already been read and the suite is already setup, so this step is skipped and the suite information is simply copied from what was already initialized (``cdata_tile``) using the ``cdata_target`` optional argument.

* Providing interfaces to call the CCPP

 * Calling ``ccpp_physics_init``

  * ``ccpp_physics_init`` is autogenerated and contained within ``ccpp_static_api.F90``. As mentioned in the :numref:`CCPP API Section %s <CCPP_API>` , it can be called to initialize groups as defined in the SDFs or the suite as a whole, depending on whether a group name is passed in as an optional argument.

 * Calling ``ccpp_physics_run``

  * ``ccpp_physics_run`` is called from ``ccpp_static_api.F90`` and contains autogenerated caps for groups and the suite as a whole as defined in the SDFs. 

 * calling ``ccpp_physics_finalize`` and ``ccpp_finalize``

  * ``ccpp_physics_finalize`` is autogenerated and contained within ``ccpp_static_api.F90``. As mentioned in the :numref:`CCPP API Section %s <CCPP_API>`, it can be called to finalize groups as defined in the current SDFs or the suite as a whole, depending on whether a group name is passed in as an optional argument.
