# Documentation for glob_var.f90

## Overview

The `glob_var.f90` file defines a Fortran module named `glob_var`. The primary purpose of this module is to declare and provide access to global constants that are used across different parts of the larger simulation project. Centralizing these parameters in a global module allows for easier management and modification of key simulation parameters.

## Key Components

### `MODULE glob_var`

This is the main Fortran module defined in the file. It encapsulates the global parameters.

## Important Variables/Constants

The module defines the following integer parameters:

*   **`NPTZ` (integer, PARAMETER)**: Initialized to `32`. Based on its usage in `shot_bch2k.f90` for `ZVAL(NPTZ,2)` and `WGTZ(NPTZ,2)`, this likely represents the number of points used for Gaussian quadrature integration along the Z-axis or a related spatial dimension.
*   **`NEMAX` (integer, PARAMETER)**: Initialized to `8`. In `shot_bch2k.f90`, arrays like `NIE(0:NEMAX)` and `IELIST(0:NEMAX-1,NEMAX)` are dimensioned with `NEMAX`. This suggests `NEMAX` likely defines the maximum number of energy levels or states considered in the calculations.

## Usage Examples

To use these global parameters in another Fortran file (e.g., a subroutine or program), you would include the `USE glob_var` statement.

```fortran
PROGRAM ExampleUsage
  USE glob_var ! Makes NPTZ and NEMAX available
  IMPLICIT NONE

  REAL*8 :: some_array(NPTZ)
  INTEGER :: energy_states(0:NEMAX-1)
  INTEGER :: i

  PRINT *, "Number of Z-axis points (NPTZ): ", NPTZ
  PRINT *, "Maximum number of energy states (NEMAX): ", NEMAX

  DO i = 1, NPTZ
    some_array(i) = DBLE(i) * 0.1D0
  END DO

  DO i = 0, NEMAX-1
    energy_states(i) = i + 1
  END DO

  PRINT *, "Initialized some_array and energy_states."

END PROGRAM ExampleUsage
```
This example demonstrates how `NPTZ` and `NEMAX` can be accessed and used to declare arrays or control loops once the `glob_var` module is imported.

## Dependencies and Interactions

*   The `glob_var` module is self-contained and does not depend on other custom modules or files for its definition.
*   It is designed to be used by other programs and modules within the project that require access to these global parameters. For example, `shot_bch2k.f90` uses this module extensively.
*   Changes to the parameter values in `glob_var.f90` will affect all parts of the system that use these parameters, requiring a recompile of dependent files.
```
