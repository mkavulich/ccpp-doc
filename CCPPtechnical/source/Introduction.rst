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

For the latest version of the released code, please visit the `DTC Website <http://www.dtcenter.org/gmtb/users/ccpp>`_

Please send questions and comments to the help desk: gmtb-help@ucar.edu.
When using the CCPP with NOAA's Unified Forecast System, questions can also be posted
in the UFS Forum at https://forums.ufscommunity.org/.

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
