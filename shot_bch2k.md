# Documentation for shot_bch2k.f90

## Overview

The `shot_bch2k.f90` file contains the main Fortran program `SHOTMPI`. This program is designed to calculate the shot noise in a molecular junction. It leverages the Message Passing Interface (MPI) for parallel computation, distributing calculations likely across different energy levels or other parallelizable aspects of the simulation. The program reads various physical parameters, atomic configurations, and pre-computed wave functions, then performs complex calculations to determine shot noise and related current properties, and finally writes the results to output files.

## Key Components

### `PROGRAM SHOTMPI`

This is the main entry point of the application. The program structure can be broadly divided into the following logical blocks:

1.  **Initialization**:
    *   Initializes MPI environment (`MPI_INIT`).
    *   Reads parameters from `param.txt` using a NAMELIST (`PARAM`). This includes simulation parameters like dimensions, energy ranges, Fermi levels, etc.
    *   Allocates and reads atomic coordinates (`ATOM` NAMELIST) and panel definitions (`PANELS` NAMELIST).
    *   Reads energy levels and weights.
    *   Sets up Gaussian quadrature points and weights for Z-integration using `GaussLeg` subroutine.
    *   Calculates Lanczos convergence factors.
2.  **Wave Function Loading**:
    *   Allocates memory for wave function arrays (`PSI0_R`, `PSI_R`, `PSI0_L`, `PSI_L`).
    *   Reads pre-computed wave functions from binary files (e.g., `./wave/<energy_count>r.bin`, `./wave/<energy_count>l.bin`) for different energy levels.
3.  **MPI Task Distribution**:
    *   The root MPI node (myNode == 0) distributes energy levels (`IE` loop) among available MPI worker nodes. Each worker node is assigned a subset of energies to process.
4.  **Main Calculation Loop (per energy `IE` on each node)**:
    *   Sets up Gaussian quadrature for phi and kappa_x integration using `GaussLeg`.
    *   Calculates various scattering matrices and related quantities (e.g., `SILRT`, `SIRRT`, `SILR0`, `SIRR0`, `SILRD`, `SIRRD`). This involves complex multi-dimensional summations over spatial dimensions (LX, LY, LZ, LZP) and wave function components.
    *   Computes contributions to current (`pGIZET`, `pGIZE0`, `pGIZED`) and shot noise (`pGSHTZT`, `pGSHTZ0`, `pGSHTZD`) for the assigned energy `E`.
5.  **MPI Reduction**:
    *   After individual nodes complete their calculations, `MPI_Allreduce` is used to sum up the partial contributions (current, shot noise) from all nodes to get the total values (`GIZT`, `GIZ0`, `GIZD`, `GSHTZT`, `GSHTZ0`, `GSHTZD`).
6.  **Output and Finalization**:
    *   The root node writes the calculated total shot noise, current, and Fano factors to `SZ_bch2K.out`.
    *   Finalizes the MPI environment (`MPI_FINALIZE`).

## Important Variables/Constants

### NAMELIST Variables

*   **`/PARAM/`**: Contains main simulation control parameters.
    *   `NZV`, `lpar1`, `lpar2`, `L`, `ZLEFT`: Geometric or spatial parameters.
    *   `Edn`, `Ndn`, `Eup`, `Nup`: Energy range parameters (down/up, number of points).
    *   `N_X`, `N_Y`, `N_Z`: Discretization points in X, Y, Z dimensions.
    *   `NPHI`, `NKAPAX`: Number of points for phi and kappa_x integrations.
    *   `NE1`, `NE2`, `NE3`: Components of the total number of energy levels.
    *   `Ef_L`, `Ef_R`, `Eb_L`, `Eb_R`: Fermi and band edge energies for Left/Right contacts.
    *   `FFT_MAX_XY`, `FFT_MAX_Z`: Parameters likely related to FFT grid sizes if used elsewhere or in pre-computation steps.
*   **`/ATOM/`**: Contains atomic coordinates.
    *   `RJ(3, 20, NZV)`: Array likely storing X,Y,Z coordinates for up to 20 atoms across `NZV` segments.
*   **`/PANELS/`**: Contains panel definitions for geometry.
    *   `NPAN(3)`, `NORDPN(50,3)`, `EPAN(50,3)`: Parameters defining panels, possibly for boundary element methods or similar geometric discretizations.

### Key Arrays and Variables

*   **`EE(NE)`**: Array of energy levels.
*   **`WGTE(NE)`**: Weights associated with energy levels.
*   **`PSI0_R`, `PSI_R`, `PSI0_L`, `PSI_L`**: Large complex arrays holding right- and left-going wave functions (and their "0" counterparts, possibly incident waves). Dimensions: `(-N_X:N_X, -N_Y:N_Y, -N_Z:N_Z, 1:NPHI, 1:NKAPAX, NES:NE)`.
*   **`ZVAL(NPTZ,2)`, `WGTZ(NPTZ,2)`**: Abscissas and weights for Z-integration.
*   **`PHI(NPHI)`, `WGTPHI(NPHI)`**: Abscissas and weights for phi integration.
*   **`KAPPAX(NKAPAX)`, `WGTKPX(NKAPAX)`**: Abscissas and weights for kappa_x integration.
*   **`SILRT`, `SILR0`, `SILRD`, `SIRRT`, `SIRR0`, `SIRRD`**: Complex arrays holding intermediate results of scattering calculations. Dimensions: `(1:NPHI, 1:NKAPAX, 1:NPHI, 1:NKAPAX, NES:NE)`.
*   **`GIZT`, `GIZ0`, `GIZD`**: Total, bare, and differential current.
*   **`GSHTZT`, `GSHTZ0`, `GSHTZD`**: Total, bare, and differential shot noise.
*   **`myNode`, `nNodes`**: MPI rank of the current process and total number of MPI processes.
*   **`PI`**: Mathematical constant Pi.
*   **`CL`**: A constant calculated from `lpar1`, `lpar2`.

## Usage Examples

1.  **Compilation**: The program is compiled using the provided `makefile`. Typically, one would run:
    ```bash
    make shot_bch2k
    ```
2.  **Input Files**:
    *   `param.txt`: Contains the simulation parameters defined in the NAMELISTs. This file must be present in the working directory from which `shot_bch2k` is run, or in the `./wave/` subdirectory as per `path4` initialization.
    *   Wave function files: Binary files (e.g., `0r.bin`, `0l.bin`, `1r.bin`, etc., where the number corresponds to energy level `IECOUNT`) are expected in a `./wave/` subdirectory. These are read during the setup.
3.  **Execution**: The program is an MPI application and should be run using `mpirun` (or a similar MPI launcher).
    ```bash
    mpirun -np <num_processes> ./shot_bch2k
    ```
    Replace `<num_processes>` with the desired number of MPI processes.
4.  **Output Files**:
    *   `SZ_bch2K.out`: Contains the primary results: total, bare, and differential shot noise; total, bare, and differential current; and corresponding Fano factors.
    *   `print_bch2K`: Contains logging information, especially regarding the distribution of energy levels to MPI nodes.

## Dependencies and Interactions

*   **Internal Dependencies**:
    *   Uses the `glob_var` module to access global parameters like `NPTZ` and `NEMAX`.
    *   Calls the `GaussLeg` subroutine for calculating quadrature points and weights.
*   **External Library Dependencies**:
    *   **MPI**: Heavily relies on an MPI library (e.g., MPICH, OpenMPI, PGI MPI) for parallel processing. The `mpif.h` include file and MPI calls (`MPI_INIT`, `MPI_Comm_size`, `MPI_Comm_rank`, `MPI_Barrier`, `MPI_Send`, `MPI_Recv`, `MPI_BCAST`, `MPI_Allreduce`, `MPI_FINALIZE`) are central to its operation.
*   **File System Interactions**:
    *   Reads `param.txt` (or `./wave/param.txt`).
    *   Reads multiple binary wave function files from the `./wave/` directory.
    *   Writes results to `SZ_bch2K.out`.
    *   Writes log messages to `print_bch2K`.
*   **Build System**: Compiled using a `makefile` with a Fortran compiler (e.g., `mpif90`).

This program forms the core computational engine for shot noise analysis in the described system. Its parallel nature suggests it's intended for computationally intensive simulations.
```
