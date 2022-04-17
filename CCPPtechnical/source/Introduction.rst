.. include:: prolog.inc

How to Use this Document
========================

This document contains documentation for the Common Community Physics Package (CCPP). It describes the

* Physics schemes and interstitials
* Suite definition files
* CCPP-compliant parameterizations
* Process to add a new scheme or suite
* Host-side coding
* CCPP code management and governance
* Parameterization-specific output
* Debugging strategies 

This table describes the type changes and symbols used in this guide.

+------------------------+------------------------------+---------------------------------------+
| **Typeface or Symbol** |  **Meaning**                 |  **Example**                          |
+========================+==============================+=======================================+
| ``AaBbCc123``          | The names of commands,       | Edit your ``.bashrc`` |br|            |
|                        | files, and directories; |br| | Use ``ls -a`` to list all files. |br| |
|                        | on-screen computer output    | ``host$ You have mail!``              |
+------------------------+------------------------------+---------------------------------------+

Following these typefaces and conventions, shell commands, code examples, namelist variables, etc.
will be presented in this style:

.. code-block:: console

   mkdir ${TOP_DIR}

In several places in the technical documentation, we need to refer to locations of files or directories in the source code. Since the directory structure depends on the host model, in particular the directories where the ``ccpp-framework`` and ``ccpp-physics`` source code is checked out, and the directory from which the ``ccpp_prebuild.py`` code generator is called, we use the following convention:

1. When describing files relative to the ``ccpp-framework`` or ``ccpp-physics`` top-level, without referring to a specific model, we use ``ccpp-framework/path/to/file/A`` and ``ccpp-physics/path/to/file/B``.
2. When describing specific tasks that depend on the directory structure within the host moodel, for example how to run ``ccpp_prebuild.py``, we explicitly mention the host model and use its directory structure relative to the top-level directory. For the example of the SCM: ``./ccpp/framework/path/to/file/A``.