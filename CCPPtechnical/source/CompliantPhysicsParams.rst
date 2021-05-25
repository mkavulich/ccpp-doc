.. _CompliantPhysParams:

****************************************
CCPP-Compliant Physics Parameterizations
****************************************

The rules for a scheme to be considered CCPP-compliant are summarized in this section. It
should be noted that making a scheme CCPP-compliant is a necessary but not guaranteed step
for the acceptance of the scheme in the pool of supported CCPP-Physics. Acceptance is
dependent on scientific innovation, demonstrated value, and compliance with the rules
described below. The criteria for acceptance of a scheme into the CCPP is under development.

It is recommended that parameterizations be comprised of the smallest units that will be used.
For example, if a given set of deep and shallow convection schemes will always be called together
and in a pre-established order, it is acceptable to group them within a single scheme. However, if one
envisions that the deep and shallow convection schemes may someday operate independently, it is
recommended to code two separate schemes to allow more flexibility.

Some schemes in the CCPP have been implemented using a driver as an entry point. In this context,
a driver is defined as a wrapper that sits on top of the actual scheme and provides the CCPP entry
points. In order to minimize the layers of code in the CCPP, the implementation of a driver is
discouraged, that is, it is preferable that the CCPP be composed of atomic parameterizations. One
example is the implementation of the MG microphysics, in which a simple entry point
leads to two versions of the scheme, MG2 and MG3.  A cleaner implementation would be to retire MG2
in favor of MG3, to put MG2 and MG3 as separate schemes, or to create a single scheme that can behave
as MG2 and MG3 depending on namelist options.

The implementation of a driver is reasonable under the following circumstances:

* To preserve schemes that are also distributed outside of the CCPP. For example, the Thompson
  microphysics scheme is distributed both with the Weather Research and Forecasting (WRF) model
  and with the CCPP. Having a driver with CCPP directives allows the Thompson scheme to remain
  intact so that it can be synchronized between the WRF model and the CCPP distributions. See
  more in ``mp_thompson.F90`` in the ``ccpp-physics/physics`` directory.

* To deal with optional arguments. A driver can check whether optional arguments have been
  provided by the host model to either write out a message and return an error code or call a
  subroutine with or without optional arguments. For example, see ``mp_thompson.F90``,
  ``radsw_main.F90``, or ``radlw_main.F90`` in the ``ccpp-physics/physics`` directory.

* To perform unit conversions or array transformations, such as flipping the vertical direction
  and rearranging the index order, for example, ``cu_gf_driver.F90`` in the ``ccpp-physics/physics``
  directory.

Schemes in the CCPP are classified into two categories: primary schemes and interstitial schemes.
A primary scheme is one that updates the state variables and tracers or that
produces tendencies for updating state variables and tracers based on the
representation of major physical processes, such as radiation, convection,
microphysics, etc. Exclusions are:

* Schemes that compute tendencies exclusively for diagnostic purposes.

* Schemes that adjust tendencies for different timesteps (e.g., create radiation
  tendencies based on a radiation scheme called at coarser intervals).

* Schemes that update the model state based on tendencies generated in primary schemes.

Interstitial schemes are modularized pieces of code that
perform data preparation, diagnostics, or other “glue” functions and allow primary schemes to work
together as a suite. They can be categorized as “scheme-specific” or “suite-level”. Scheme-specific
interstitial schemes augment a specific primary scheme (to provide additional functionality).
Suite-level interstitial schemes provide additional functionality on top of a class of primary schemes,
connect two or more schemes together, or provide code for conversions, initializing sums, or applying
tendencies, for example. The rules and guidelines provided in the following sections apply both to
primary and interstitial schemes.

.. _GeneralRules:

General Rules
=============
A CCPP-compliant scheme is in the form of Fortran modules. :ref:`Listing 2.1 <scheme_template>` contains
the template for a CCPP-compliant scheme (``ccpp/framework/doc/DevelopersGuide/scheme_template.F90``),
which includes at least one of these five components: the *_timestep_init*, *_init*,
*_run*, *_finalize*, and *_timestep_finalize* subroutines. Each ``.f`` or ``.F90``
file that contains an entry point(s) for CCPP scheme(s) must be accompanied by a .meta file in the same directory
as described in :numref:`Section %s <MetadataRules>`

.. _scheme_template:
.. literalinclude:: ./_static/scheme_template.F90
   :language: fortran
   :lines: 10-48

*Listing 2.1: Fortran template for a CCPP-compliant scheme showing the _run subroutine. The structure for the other phases (_timestep_init, _init, _finalize, and _timestep_finalize is identical.*

More details are found below:

* Each scheme must be in its own module and must include at least one of the
  following subroutines (entry points): *_timestep_init*, *_init*, *_run*, *_finalize*,
  and *_timestep_finalize*. The module name and the subroutine names must be consistent with the
  scheme name. The *_run* subroutine contains the
  code to execute the scheme. If subroutines *_timestep_init* or *_timestep_finalize* are present,
  they will be executed at the beginning and at the end of the host model physics timestep,
  respectively. If present, the *_init* and *_finalize* subroutines
  associated with a scheme are run before and after the *_run* phase of the scheme.
  The *_init* and *_finalize* phases can be used even if the *_run* phase if absent.
  The *_init* and *_finalize* subroutines may be called more than once depending
  on the host model’s parallelization strategy, and as such must be idempotent (the answer
  must be the same when the subroutine is called multiple times).

* Each ``.f`` or ``.F90`` file with one or more CCPP entry point schemes must be accompanied by a a ``.meta`` file containing
  metadata about the arguments to the scheme(s). For more information, see :numref:`Section %s <MetadataRules>`.

* All schemes must be preceded by the three lines below. These are markup comments used by Doxygen,
  the software employed to create the scientific documentation, to insert an external file containing metadata
  information (in this case, ``schemename_run.html``) in the documentation. See more on this topic in
  :numref:`Section %s <SciDoc>`.

.. code-block:: fortran

   !> \section arg_table_schemename_run Argument Table
   !! \htmlinclude schemename_run.html
   !!

* All external information required by the scheme must be passed in via the argument list. Statements
  such as  ``‘use EXTERNAL_MODULE’`` should not be used for passing in data and all physical constants
  should go through the argument list.

* Note that standard names, variable names, module names, scheme names and subroutine names are all case insensitive.

* Interstitial modules (``scheme_pre`` and ``scheme_post``) can be included if any part of the physics
  scheme must be executed before (``_pre``) or after (``_post``) the ``module scheme`` defined above.

.. _IOVariableRules:

.. _MetadataRules:

Metadata Table Rules
====================

Each CCPP-compliant physics scheme (``.f`` or ``.F90`` file) must have a corresponding metadata file (``.meta``)
that contains information about CCPP entry point schemes and their dependencies.  These files
contain two types of metadata tables: ``ccpp-table-properties`` and ``ccpp-arg-table``, both of which are mandatory.
The contents of these tables are described in the sections below.

Metadata files (``.meta``) are in a relaxed config file format and contain metadata
for one or more CCPP entry point schemes.

ccpp-table-properties
---------------------
The ``[ccpp-table-properties]`` section is required in every metadata file and has four valid entries:

#. ``type``:  In the CCPP Physics, ``type`` can be ``scheme``, ``module``, or ``ddt`` and must match the
   ``type`` in the associated ``[ccpp-arg-table]`` section(s).

#. ``name``:  This depends on the ``type``. For types ``ddt`` and ``module`` (for
   variable/type/kind definitions), ``name`` must match the name of the **single** associated
   ``[ccpp-arg-table]`` section. For type ``scheme``, the name must match the root names of the
   ``[ccpp-arg-table]`` sections for that scheme, without the suffixes
   ``_timestep_init``,``_init``, ``_run``, ``_finalize``, or ``_timestep_finalize``.

#. ``dependencies``: type/kind/variable definitions and physics schemes often depend on code in other files
   (e.g. "use machine" --> depends on machine.F). These dependencies must be listed in a comma-separated list.
   Relative path(s) to those file(s) must be specified here or using the ``relative_path`` entry described below.
   Dependency attributes are additive; multiple lines containing dependencies can be used.

#. ``relative_path``: If specified, the relative path is added to every file listed in the ``dependencies``.

The information in this section table allows the CCPP to compile only the schemes and dependencies needed by the
selected CCPP suite(s).

An example for type and variable definitions in ``GFS_typedefs.meta`` is shown in
:ref:`Listing 2.2 <table-properties-typedefs>`.

.. note::

   A single metadata file may require multiple instances of the [ccpp-table-properties] section.

.. _table-properties-typedefs:
.. code-block:: fortran

   ########################################################################
   [ccpp-table-properties]
     name = GFS_statein_type
     type = ddt
     dependencies =

   [ccpp-arg-table]
     name = GFS_statein_type
     type = ddt
   [phii]
     standard_name = geopotential_at_interface
   ...
   ########################################################################
   [ccpp-table-properties]
     name = GFS_stateout_type
     type = ddt
     dependencies =

   [ccpp-arg-table]
     name = GFS_stateout_type
     type = ddt
   [gu0]
     standard_name = x_wind_updated_by_physics
   ...
   ########################################################################
   [ccpp-table-properties]
     name = GFS_typedefs
     type = module
     relative_path = ../../ccpp/physics/physics
     dependencies = machine.F,physcons.F90,radlw_param.f,radsw_param.f
     dependencies = GFDL_parse_tracers.F90,rte-rrtmgp/rrtmgp/mo_gas_optics_rrtmgp.F90
     dependencies = rte-rrtmgp/rte/mo_optical_props.F90
     dependencies = rte-rrtmgp/extensions/cloud_optics/mo_cloud_optics.F90
     dependencies = rte-rrtmgp/rrtmgp/mo_gas_concentrations.F90
     dependencies = rte-rrtmgp/rte/mo_rte_config.F90
     dependencies = rte-rrtmgp/rte/mo_source_functions.F90

   [ccpp-arg-table]
     name = GFS_typedefs
     type = module
   [GFS_cldprop_type]
     standard_name = GFS_cldprop_type
     long_name = definition of type GFS_cldprop_type
     units = DDT
     dimensions = ()
     type = GFS_cldprop_type
   ...

*Listing 2.2: Example of a CCPP-compliant metadata file showing the use of the [ccpp-table-properties] section and
how it relates to [ccpp-arg-table].*

An example metadata file for the CCPP scheme ``mp_thompson.meta`` is shown in :ref:`Listing 2.3 <table-properties-mp-thompson>`.

.. _table-properties-mp-thompson:
.. code-block:: fortran

   [ccpp-table-properties]
     name = mp_thompson
     type = scheme
     dependencies = machine.F,module_mp_radar.F90,module_mp_thompson.F90
     dependencies = module_mp_thompson_make_number_concentrations.F90

   ########################################################################
   [ccpp-arg-table]
    name = mp_thompson_init
    type = scheme
   ...

   ########################################################################
   [ccpp-arg-table]
     name = mp_thompson_run
     type = scheme
   ...

   ########################################################################
   [ccpp-arg-table]
     name = mp_thompson_finalize
     type = scheme
   ...

*Listing 2.3: Example metadata file for a CCPP-compliant physics scheme using a single
[ccpp-table-properties] and how it defines dependencies for multiple [ccpp-arg-table].
In this example the timestep_init and timestep_finalize phases are not used*

ccpp-arg-table
--------------

For each CCPP compliant scheme, the ``ccpp-arg-table`` should start with this set of lines

.. code-block:: fortran

   [ccpp-arg-table]
    name = <name>
    type = <type>

* ``ccpp-arg-table`` indicates the start of a new metadata section for a given scheme.

* ``<name>`` is name of the corresponding subroutine/module.

* ``<type>`` can be ``scheme``, ``module``, or ``DDT``.

* The metadata must
  describe all input and output arguments to the scheme using the following format:

.. code-block:: fortran

   [varname]
    standard_name = <standard_name>
    long_name = <long_name>
    units = <units>
    rank = <rank>
    dimensions = <dimensions>
    type = <type>
    kind = <kind>
    intent = <intent>
    optional = <optional>

* The ``intent`` argument is only valid in ``scheme`` metadata tables, as it is not applicable to the other ``types``.

* The following attributes are optional: ``long_name``, ``kind``, and ``optional``.

* Lines can be combined using ``|`` as a separator, e.g.,

.. code-block:: console

   type = real | kind = kind_phys

* ``[varname]`` is the local name of the variable in the subroutine.

* The dimensions attribute should be empty parentheses for scalars or contain the ``standard_name`` for the start and end for
  each dimension of an array. ``ccpp_constant_one`` is the assumed start for any dimension which only has a single value.
  For example:

.. code-block:: fortran

   dimensions = ()
   dimensions = (ccpp_constant_one:horizontal_loop_extent, vertical_level_dimension)
   dimensions = (horizontal_dimension,vertical_dimension)
   dimensions = (horizontal_dimension,vertical_dimension_of_ozone_forcing_data,number_of_coefficients_in_ozone_forcing_data)

* The order of arguments in the entry point subroutines must match the order of entries in the metadata file.

* :ref:`Listing 2.4 <meta_template>` contains the template for a CCPP-compliant scheme
  (``ccpp/framework/doc/DevelopersGuide/scheme_template.meta``),

.. _meta_template:
.. literalinclude:: ./_static/scheme_template.meta
   :language: fortran
   :lines: 10-34

*Listing 2.4: Fortran template for a metadata file accompanying a CCPP-compliant scheme.*

Input/output Variable (argument) Rules
======================================

* Variables available for CCPP physics schemes are identified by their unique
  ``standard_name``. While an effort is made to comply with existing ``standard_name``
  definitions of the Climate and Forecast (CF) conventions (http://cfconventions.org), additional names
  are used in the CCPP (see below for further information).

* A list of available standard names and an example of naming conventions can be found in
  ``ccpp/framework/doc/DevelopersGuide/CCPP_VARIABLES_${HOST}.pdf``, where ``${HOST}`` is the
  name of the host model.  Running the CCPP *prebuild* script (described in :numref:`Chapter %s <CCPPPreBuild>`)
  will generate a LaTeX source file that can be compiled to produce
  a PDF file with all variables defined by the host model and requested by the physics schemes.

* A ``standard_name`` cannot be assigned to more than one local variable (``local_name``).
  The ``local_name`` of a variable can be chosen freely and does not have to match the
  ``local_name`` in the host model.

* All variable information (standard_name, units, dimensions) must match the specifications on
  the host model side, but sub-slices can be used/added in the host model. For example, when
  using the UFS Atmosphere as the host model, tendencies are split in ``GFS_typedefs.meta``
  so they can be used in the necessary physics scheme:

.. code-block:: fortran

   [dt3dt(:,:,1)]
     standard_name = cumulative_change_in_temperature_due_to_longwave_radiation
     long_name = cumulative change in temperature due to longwave radiation
     units = K
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = kind_phys
   [dt3dt(:,:,2)]
     standard_name = cumulative_change_in_temperature_due_to_shortwave_radiation
     long_name = cumulative change in temperature due to shortwave radiation
     units = K
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = kind_phys
   [dt3dt(:,:,3)]
     standard_name = cumulative_change_in_temperature_due_to_PBL
     long_name = cumulative change in temperature due to PBL
     units = K
     dimensions = (horizontal_dimension,vertical_dimension)
     type = real
     kind = kind_phys

* The two mandatory variables that any scheme-related subroutine must accept as ``intent(out)`` arguments are
  ``errmsg`` and ``errflg`` (see also coding rules in :numref:`Section %s <CodingRules>`).

* At present, only two types of variable definitions are supported by the CCPP-framework:

   * Standard Intrinsic Fortran variables are preferred (``character``, ``integer``, ``logical``, ``real``).
     For character variables, the length should be specified as ``*`` in order to allow the host model
     to specify the corresponding variable with a length of its own choice. All others can have a
     ``kind`` attribute of a ``kind`` type defined by the host model.
   * Derived data types (DDTs). While the use of DDTs is discouraged, some use cases may
     justify their application (e.g. DDTs for chemistry that contain tracer arrays or information on
     whether tracers are advected). It should be understood that use of DDTs within schemes
     forces their use in host models and potentially limits a scheme’s portability. Where possible,
     DDTs should be broken into components that could be usable for another scheme of the same type.

* It is preferable to have separate variables for physically-distinct quantities. For example,
  an array containing various cloud properties should be split into its individual
  physically-distinct components to facilitate generality. An exception to this rule is if
  there is a need to perform the same operation on an array of otherwise physically-distinct
  variables. For example, tracers that undergo vertical diffusion can be combined into one array
  where necessary. This tactic should be avoided wherever possible, and is not acceptable merely
  as a convenience.

* If a scheme is to make use of CCPP’s subcycling capability, the loop counter can be obtained
  from CCPP as an ``intent(in)`` variable (see a :ref:`mandatory list of variables <MandatoryVariables>`
  that are provided by the CCPP-Framework and/or the host model for this and other purposes).

.. _CodingRules:

Coding Rules
============

* Code must comply to modern Fortran standards (Fortran 90/95/2003).

* Labeled ``end`` statements should be used for modules, subroutines and functions,
  for example, ``module scheme_template → end module scheme_template``.

* Implicit variable declarations are not allowed. The ``implicit none`` statement is mandatory and
  is preferable at the module-level so that it applies to all the subroutines in the module.

* All ``intent(out)`` variables must be set inside the subroutine, including the mandatory
  variables ``errflg`` and ``errmsg``.

* Decomposition-dependent host model data inside the module cannot be permanent,
  i.e. variables that contain domain-dependent data cannot be kept using the ``save`` attribute.

* ``goto`` statements are not alowed.

* ``common`` blocks are not allowed.

* Errors are handled by the host model using the two mandatory arguments ``errmsg`` and
  ``errflg``. In the event of an error, a meaningful error message should be assigned to ``errmsg``
  and set ``errflg`` to a value other than 0, for example:

.. code-block:: bash

   write (errmsg, ‘(*(a))’) ‘Logic error in scheme xyz: …’
   errflg = 1
   return

* Schemes are not allowed to abort/stop the program.

* Schemes are not allowed to perform I/O operations except for reading lookup tables
  or other information needed to initialize the scheme, including stdout and stderr.
  Diagnostic messages are tolerated, but should be minimal.

* Line lengths of no more than 120 characters are suggested for better readability.

Additional coding rules are listed under the *Coding Standards* section of the NOAA NGGPS
Overarching System team document on Code, Data, and Documentation Management for NOAA Environmental
Modeling System (NEMS) Modeling Applications and Suites (available at
https://docs.google.com/document/u/1/d/1bjnyJpJ7T3XeW3zCnhRLTL5a3m4_3XIAUeThUPWD9Tg/edit#heading=h.97v79689onyd).

Parallel Programming Rules
==========================

Most often shared memory (OpenMP: Open Multi-Processing) and MPI (Message Passing Interface)
communication are done outside the physics in which case the physics looping and arrays already
take into account the sizes of the threaded tasks through their input indices and array
dimensions.  The following rules should be observed when including OpenMP or MPI communication
in a physics scheme:

* Shared-memory (OpenMP) parallelization inside a scheme is allowed with the restriction
  that the number of OpenMP threads to use is obtained from the host model as an ``intent(in)``
  argument in the argument list (:ref:`Listing 6.2 <MandatoryVariables>`).

* MPI communication is allowed in the ``_timestep_init``, ``_init``, ``_finalize``,
  and ``_timestep_finalize`` phases for the purpose
  of computing, reading or writing scheme-specific data that is independent of the host
  model’s data decomposition.

* If MPI is used, it is restricted to global communications: barrier, broadcast,
  gather, scatter, reduction.

*  An example of a valid use of MPI is the initial read of a lookup table of aerosol
   properties by one or more MPI processes, and its subsequent broadcast to all processes.
   Several restrictions apply:

   * The implementation of reading and writing of data must be scalable to perform
     efficiently from a few to millions of tasks.
   * The MPI communicator must be provided by the host model as an ``intent(in)``
     argument in the argument list (:ref:`see list of mandatory variables <MandatoryVariables>`).
   * Point-to-point communication is not allowed.
   * The use of MPI_COMM_WORLD is not allowed.

* Calls to MPI and OpenMP functions, and the import of the MPI and OpenMP libraries,
  must be guarded by C preprocessor directives as illustrated in the following listing.
  OpenMP pragmas can be inserted without C preprocessor guards, since they are ignored
  by the compiler if the OpenMP compiler flag is omitted.

.. code-block:: fortran

   #ifdef MPI
     use mpi
   #endif
   #ifdef OPENMP
     use omp_lib
   #endif
   ...
   #ifdef MPI
     call MPI_BARRIER(mpicomm, ierr)
   #endif

   #ifdef OPENMP
     me = OMP_GET_THREAD_NUM()
   #else
     me = 0
   #endif

* For Fortran coarrays, consult with the CCPP Forum (https://dtcenter.org/forum/ccpp-user-support).

.. include:: ScientificDocRules.inc

.. Bibliography should go at the end of the last chapter

.. bibliography:: references.bib
