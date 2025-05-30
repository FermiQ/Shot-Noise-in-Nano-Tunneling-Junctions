# Documentation for GaussLeg.f90

## Overview

The `GaussLeg.f90` file provides a Fortran subroutine named `GaussLeg` that implements Gaussian-Legendre quadrature. This is a numerical integration technique used to approximate the definite integral of a function. The subroutine calculates the abscissas (nodes) and weights for Gaussian-Legendre quadrature over a given interval [x1, x2] for a specified number of points `n`. This routine is crucial for high-precision numerical integration tasks within the larger project.

## Key Components

### `SUBROUTINE GaussLeg(x1, x2, n, x, w)`

*   **`x1` (Real*8, Input)**: The lower limit of the integration interval.
*   **`x2` (Real*8, Input)**: The upper limit of the integration interval.
*   **`n` (Integer, Input)**: The number of quadrature points (nodes and weights) to calculate.
*   **`x(n)` (Real*8, Output)**: An array containing the `n` calculated abscissas (nodes) of the quadrature.
*   **`w(n)` (Real*8, Output)**: An array containing the `n` calculated weights of the quadrature.

The subroutine uses Newton's method to find the roots of the Legendre polynomial, which are the abscissas for the quadrature. It then calculates the corresponding weights.

## Important Variables/Constants

*   **`EPS` (Real*8, PARAMETER)**: Defined as `3.d-14`. This constant represents the relative precision used as a convergence criterion within Newton's method for finding the roots of Legendre polynomials. It dictates how accurately the roots (abscissas) are determined.

## Usage Examples

To use the `GaussLeg` subroutine, you would typically call it from another Fortran program or subroutine.

```fortran
PROGRAM TestGaussLeg
  IMPLICIT NONE
  INTEGER, PARAMETER :: N_POINTS = 10
  REAL*8 :: lower_limit, upper_limit
  REAL*8 :: nodes(N_POINTS), weights(N_POINTS)
  INTEGER :: i

  lower_limit = 0.0D0
  upper_limit = 1.0D0

  CALL GaussLeg(lower_limit, upper_limit, N_POINTS, nodes, weights)

  PRINT *, "Calculated Nodes and Weights:"
  DO i = 1, N_POINTS
    PRINT *, "Node ", i, ": ", nodes(i), " Weight ", i, ": ", weights(i)
  END DO

END PROGRAM TestGaussLeg
```

This example program calls `GaussLeg` to get 10 quadrature points for the interval [0.0, 1.0] and then prints them.

## Dependencies and Interactions

*   The `GaussLeg` subroutine is a self-contained mathematical routine for calculating Gaussian-Legendre quadrature points and weights.
*   It does not depend on other custom modules or files within this specific project for its internal calculations.
*   It is likely used by other parts of the system, such as the `shot_bch2k.f90` program, to perform numerical integrations required for the physics simulations.
```
