.. include:: ../03-exports/roles.include

.. _cmake_options:

CMake options
=============

*eProsima Discovery Server* provides some CMake options for changing the behavior and configuration of
*Discovery Server* application. These options allow the user to enable/disable certain settings by defining these
options to ON/OFF at the CMake execution.

.. list-table::
    :header-rows: 1

    *   - Option
        - Description
        - Possible values
        - Default
    *   - :class:`COMPILE_EXAMPLES`
        - Build Discovery Server example.
        - ``Release`` |br|
          ``Debug``
        - ``Release``
    *   - :class:`LOG_LEVEL_INFO`
        - Set logging level to Info.
        - ``ON`` |br|
          ``OFF``
        - ``OFF``
    *   - :class:`LOG_LEVEL_WARN`
        - Set logging level to Warning.
        - ``ON`` |br|
          ``OFF``
        - ``OFF``
    *   - :class:`LOG_LEVEL_ERROR`
        - Set logging level to Error.
        - ``ON`` |br|
          ``OFF``
        - ``OFF``
    *   - :class:`SANITIZER`
        - Adds run-time instrumentation to the code. Supported options are:

            - ``Thread`` enables Thread Sanitizer. |br|
            - ``Address`` enables Address Sanitizer.
        - ``OFF`` |br| ``Address`` |br| ``Thread``
        - ``OFF``
