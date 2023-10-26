.. _AddNewSchemes:
  
****************************************
Connecting a new scheme to CCPP
****************************************

This chapter contains a brief description on how to add a new :term:`scheme` to the :term:`CCPP Physics` pool. Aside from the basic design elements for interoperability (:term:`entry points <entry point>`, metadata files), most CCPP requirements simply follow from coding best practices.

     .. note:: The instructions in this chapter assume the user is implementing this scheme for use with the CCPP Single-Column model (:term:`SCM`); not only is the SCM more lightweight than a full 3D NWP model for development purposes, but using the SCM as a :term:`host model` is a requirement for all new CCPP schemes for testing purposes. For implementation in other host models, especially for adding new variables, some modifications to a host model’s metadata may be required; see :numref:`Chapter %s <Host-side Coding>` for details

==============================
Preparing a scheme for CCPP
==============================
There are a few steps that can be taken to prepare a scheme for addition to CCPP prior to starting the process of implementing it in the CCPP framework:

1. Remove/refactor any incompatible features described in :numref:`Section %s <CodingRules>`. This includes updating Fortran code to at least Fortran 90 standards, removing STOP and GOTO statements, removing common blocks, and refactoring any other disallowed features.
2. Make an inventory of all variables that are inputs and/or outputs to the scheme. Check the file ``ccpp-framework/doc/DevelopersGuide/CCPP_VARIABLES_SCM.pdf`` to see if each variable has already been implemented in the single column model. If there are variables that are not available, see `Section %s <AddingNewVariables>`.

=============================
Implementing a scheme in CCPP
=============================

There are, broadly speaking, two philosophies for connecting an existing physics scheme to the CCPP framework: 

1. Refactor the existing scheme to CCPP format standards, using ``pre_`` and ``post_`` :term:`interstitial schemes <interstitial scheme>` to interface to and from the existing scheme if necessary.
2. Create a driver scheme as an interface from the existing scheme's Fortran module to the CCPP framework.

.. figure:: _static/ccpp_scheme_diagram.png

   *Diagram of the methods described in this section*

Method 1 is the preferred method of adapting a scheme to CCPP. This involves making modifications to the original scheme so that it is CCPP-compliant (see :numref:`Chapter %s <CompliantPhysParams>`), containing only subroutines that correspond to CCPP entry points (i.e. ``schemename_init``, ``schemename_run``, etc.). It should be accompanied by appropriate metadata files (see :numref:`Section %s <MetadataRules>`), and it must be updated to remove any disallowed features as listed in :numref:`Section %s <CodingRules>`. However, there are cases where method 1 may not be possible (for example, in schemes that are also used in non-CCPP-compliant models), in which case, method 2 can be employed.

While method 1 is preferred, there are cases where method 1 may not be possible: for example, in schemes that are shared with other, non-CCPP hosts, and so require specialized, model-specific drivers, and might be beholden to different coding standards required by another model. In cases such as this, method 2 may be employed.

Method 2 involves fewer changes to the original scheme’s Fortran module: A CCPP-compliant driver module (see :numref:`Chapter %s <CompliantPhysParams>`) handles defining the inputs to and outputs from the scheme module in terms of state variables, constants, and tendencies provided by the model as defined in the scheme’s .meta file. The calculation of variables that are not available directly from the model, and conversion of scheme output back into the variables expected by CCPP, should be handled by interstitial schemes (``schemename_pre`` and ``schemename_post``). While this method puts most CCPP-required features in the driver and interstitial subroutines, the original scheme must still be updated to remove STOP statements, common blocks, or any other disallowed features as listed in :numref:`Section %s <CodingRules>`. 

For both methods, optional interstitial schemes can be used for code that can not be handled within the scheme itself. For example, if different code needs to be run for coupling with other schemes or in different orders (e.g. because of dependencies on other schemes and/or the order the scheme is run in the :term:`SDF`), or if variables needed by the scheme must be derived from variables provided by the host. See  :numref:`Chapter %s <CompliantPhysParams>` for more details on primary and interstitial schemes.

     .. note:: Depending on the complexity of the scheme and how it works together with other schemes, multiple interstitial schemes may be necessary. 

------------------------------
Adding new variables to CCPP
------------------------------

To prepare a scheme for this conversion to CCPP-compliance, the first step is to identify the input variables required for the new scheme and check if they are already available for use in the CCPP. This can be done by checking the metadata information in ``CCPP_typedefs.meta`` or by checking the file ``ccpp-framework/doc/DevelopersGuide/CCPP_VARIABLES_SCM.pdf`` generated by ``ccpp_prebuild.py``. If all quantities needed by the scheme are already available as variables in CCPP, they can be invoked in the scheme’s metadata file, and the rest of this subsection can be skipped.

     .. note:: The instructions in this chapter assume the user is implementing this scheme in the CCPP Single-Column model (SCM). Other host model variables can be found in different files; see :numref:`Chapter %s <Host-side Coding>` for details

If an input variable needed by the scheme is not available, first consider if it can be calculated from the existing CCPP variables. If so, an :term:`interstitial scheme` (such as ``schemename_pre``; see  :numref:`Chapter %s <CompliantPhysParams>` for more details) can be created to calculate the variable(s). If this path is taken, the variable must be defined (but not initialized) in the host model, as the memory for this variable must be allocated by the host. Instructions for how to add variables on the host model side can be found in :numref:`Chapter %s <Host-side Coding>`.

     .. note:: The CCPP Framework is capable of performing automatic unit conversions between variables provided by the host model and variables required by the new scheme. See :numref:`Section %s <AutomaticUnitConversions>` for details.

If an entirely new variable needs to be added, consult the CCPP standard names dictionary and the rules for creating new standard names at https://github.com/escomp/CCPPStandardNames. If in doubt, use the GitHub discussions page in the CCPP Framework repository (https://github.com/ncar/ccpp-framework) to discuss the suggested new standard name(s) with the CCPP developers.

     .. note:: It is important to keep in mind that not all data types are persistent in memory. If the value of a variable must be remembered from one call to the next, it should not be in the interstitial or diagnostic data types. Most variables in the interstitial data type are reset (to zero or other initial values) at the beginning of a physics :term:`group` and do not persist from one :term:`set` to another or from one group to another. The diagnostic data type is periodically reset because it is used to accumulate variables for given time intervals. However, there is a small subset of interstitial variables that are set at creation time and are not reset; these are typically dimensions used in other interstitial variables.

For variables that can be set via namelist, the ``GFS_control_type`` Derived Data Type (DDT) should be used. In this case, it is also important to modify the namelist file to include the new variable.

If information from the previous timestep is needed, it is important to identify if the host model provides this information, or if it needs to be stored as a special variable. For example, in the Model for Prediction Across Scales (MPAS), variables containing the values of several quantities in the preceding timesteps are available. When that is not the case, as in the :term:`UFS Atmosphere`, interstitial schemes are needed to access these quantities.

     .. note:: As an example, the reader is referred to the `GF convective scheme <https://dtcenter.ucar.edu/GMTB/v6.0.0/sci_doc/_c_u__g_f.html>`_, which makes use of interstitials to obtain the previous timestep information. 

Consider allocating the new variable only when needed (i.e. when the new scheme is used and/or when a certain control flag is set). If this is a viable option, following the existing examples in ``CCPP_typedefs.F90`` and ``GFS_typedefs.meta`` for allocating the variable and setting the ``active`` attribute in the metadata correctly.

==============================
Tips
==============================


* Identify the variables required for the new scheme and check if they are already available for use in the CCPP by checking the metadata information in ``GFS_typedefs.meta`` or by perusing file ``ccpp-framework/doc/DevelopersGuide/CCPP_VARIABLES_{FV3,SCM}.pdf`` generated by ``ccpp_prebuild.py``.

    * If the variables are already available, they can be invoked in the scheme’s metadata file and one can skip the rest of this subsection. If the variable required is not available, consider if it can be calculated from the existing variables in the CCPP. If so, an :term:`interstitial scheme` (such as ``scheme_pre``; see more in :numref:`Chapter %s <CompliantPhysParams>`) can be created to calculate the variable. However, the variable must be defined but not initialized in the :term:`host model` as the memory for this variable must be allocated on the host model side.  Instructions for how to add variables to the host model side is described in :numref:`Chapter %s <Host-side Coding>`.

     .. note:: The :term:`CCPP framework` is capable of performing automatic unit conversions between variables provided by the host model and variables required by the new scheme. See :numref:`Section %s <AutomaticUnitConversions>` for details.

    * If new namelist variables need to be added, the ``GFS_control_type`` DDT should be used. In this case, it is also important to modify the namelist file ``input.nml`` to include the new variable.

    * It is important to note that not all data types are persistent in memory. Most variables in the interstitial data type are reset (to zero or other initial values) at the beginning of a physics :term:`group` and do not persist from one :term:`set` to another or from one group to another. The diagnostic data type is periodically reset because it is used to accumulate variables for given time intervals.  However, there is a small subset of interstitial variables that are set at creation time and are not reset; these are typically dimensions used in other interstitial variables. 

     .. note:: If the value of a variable must be remembered from one call to the next, it should not be in the interstitial or diagnostic data types.

    * If information from the previous timestep is needed, it is important to identify if the host model readily provides this information. For example, in the Model for Prediction Across Scales (MPAS), variables containing the values of several quantities in the preceding timesteps are available. When that is not the case, as in the :term:`UFS Atmosphere`, interstitial schemes are needed to compute these variables. As an example, the reader is referred to the `GF convective scheme <https://dtcenter.ucar.edu/GMTB/v6.0.0/sci_doc/_c_u__g_f.html>`_, which makes use of interstitials to obtain the previous timestep information.

    * Consider allocating the new variable only when needed (i.e. when the new scheme is used and/or when a certain control flag is set). If this is a viable option, following the existing examples in ``GFS_typedefs.F90`` and ``GFS_typedefs.meta`` for allocating the variable and setting the ``active`` attribute in the metadata correctly.

* If an entirely new variable needs to be added, consult the CCPP :term:`standard names<standard name>` dictionary and the rules for creating new standard names at https://github.com/escomp/CCPPStandardNames. If in doubt, use the GitHub discussions page in the CCPP Framework repository (https://github.com/ncar/ccpp-framework) to discuss the suggested new standard name(s) with the CCPP developers.

* Examine scheme-specific and :term:`suite` interstitials to see what needs to be replaced/changed; then check existing scheme interstitial and determine what needs to replicated. Identify if your new scheme requires additional interstitial code that must be run before or after the scheme and that cannot be part of the scheme itself, for example because of dependencies on other schemes and/or the order the scheme is run in the :term:`SDF`.

* Follow the guidelines outlined in :numref:`Chapter %s <CompliantPhysParams>` to make your scheme CCPP-compliant. Make sure to use an uppercase suffix ``.F90`` to enable C preprocessing.

* Locate the CCPP *prebuild* configuration files for the target host model, for example:

    * ``ufs-weather-model/FV3/ccpp/config/ccpp_prebuild_config.py`` for the :term:`UFS Atmosphere`
    * ``ccpp-scm/ccpp/config/ccpp_prebuild_config.py`` for the :term:`SCM`

* Add the new scheme to the Python dictionary in ``ccpp_prebuild_config.py`` using the same path
  as the existing schemes:

  .. code-block:: console

    SCHEME_FILES = [ ...
    ’../some_relative_path/existing_scheme.F90’,
    ’../some_relative_path/new_scheme.F90’,
    ...]

* Place new scheme in the same location as existing schemes in the CCPP directory structure, e.g., ``../some_relative_path/new_scheme.F90``.

* Edit the SDF and add the new scheme at the place it should be run. SDFs are located in

    * ``ufs-weather-model/FV3/ccpp/suites`` for the UFS Atmosphere
    * ``ccpp-scm/ccpp/suites`` for the SCM

* Before running, check for consistency between the namelist and the SDF. There is no default consistency check between the SDF and the namelist unless the developer adds one. Errors may result in segmentation faults in running something you did not intend to run if the arrays are not allocated.

* Test and debug the new scheme:

    * Typical problems include segmentation faults related to variables and array allocations.
    * Make sure the SDF and namelist are compatible. Inconsistencies may result in segmentation faults because arrays are not allocated or in unintended scheme(s) being executed.
    * A scheme called GFS_debug (``GFS_debug.F90``) may be added to the SDF where needed to print state variables and interstitial variables. If needed, edit the scheme beforehand to add new variables that need to be printed.
    * Check the *prebuild* script for success/failure and associated messages; run the *prebuild* script with the `--debug` and `--verbose` flags.
    * Compile code in DEBUG mode, run through debugger if necessary (gdb, Allinea DDT, totalview, ...).
    * Use memory check utilities such as valgrind.
    * Double-check the metadata file associated with your scheme to make sure that all information, including standard names and units, correspond to the correct local variables.

* Done. Note that no further modifications of the build system are required, since the *CCPP Framework* will autogenerate the necessary makefiles that allow the host model to compile the scheme.
