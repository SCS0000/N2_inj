# N2 Injection at High Pressure — OpenFOAM Setup (CFD-Seminar WS 26/27)

Simulation of cryogenic nitrogen injected into a warm, pressurized nitrogen
atmosphere (experiment of Branam & Mayer, case 4) with the cubic
Peng-Robinson equation of state. Solver: `fluid` via `foamRun`
(OpenFOAM 13), steady state (SIMPLE), k-epsilon RANS with wall functions.

## Operating points

| Case | p_ch [bar] | T_ch [K] | T_in [K] | U_in [m/s] | Purpose |
|------|-----------|----------|----------|------------|---------|
| `operatingPointA` | 39.8 | 297 | 137 | 5.4 | Validation (case 4 in [3]) |
| `operatingPointB` | 35.0 | 297 | 100 | 5.4 | Robustness of h→T conversion |

## Geometry (2D axisymmetric wedge, 5°)

- Injector tube: d = 2.2 mm, included length L1 = 50 mm (adiabatic wall)
- Chamber: D = 80 mm, length L2 = 200 mm (isothermal wall, T_ch)
- Inlet at x = 0, injector exit at x = 50 mm, outlet at x = 250 mm
- Mesh: `system/blockMeshDict`, ~20k cells, radial refinement of the jet
  region (r < 4 mm). For the mesh study, refine all cell counts by a
  factor 1.5 in both spatial directions per refinement level.

## Case structure (per operating point)

```
operatingPointX/
├── 0/          U, p, T, k, epsilon, nut, alphat (initial + boundary conditions)
├── constant/   thermophysicalProperties (PengRobinsonGas + hConst, N2)
│               turbulenceProperties (kEpsilon)
└── system/     blockMeshDict, controlDict, fvSchemes, fvSolution, decomposeParDict
```

Boundary conditions:

- `inlet`: U = 5.4 m/s fixed, T = T_in fixed, p zeroGradient,
  k = 0.1094, epsilon = 38.6 (I = 5 %, l = 0.07 d)
- `outlet`: p = p_ch fixed, all other fields zeroGradient
- `injectorWall` (tube + upstream chamber face): no-slip, adiabatic,
  kqRWallFunction / epsilonWallFunction / nutkWallFunction
- `chamberWall`: no-slip, isothermal T = 297 K, wall functions
- `wedge`: 5° axisymmetric wedge

## Run

```sh
cd operatingPointA
./Allrun                 # blockMesh + foamRun -solver fluid
# or manually:
blockMesh
foamRun -solver fluid
# parallel:
decomposePar && mpirun -np 4 foamRun -solver fluid -parallel
```

## Notes

- **Operating point B** requires the bisection method for the
  enthalpy→temperature conversion (Newton fails near the critical point,
  see project description section 3.3): replace `thermoI.H` with the
  patched version in `src-patch/` and recompile the thermophysical
  libraries with `wmake` (see `src-patch/README.md`).
- **Ideal-gas comparison** (operating point A): set
  `equationOfState perfectGas;` in `constant/thermophysicalProperties`.
- **Post-processing**: axial/radial profiles at x/d = 5, 10, 15, 20, 30
  (x measured from the injector exit, d = 2.2 mm), e.g. with `sample`
  or `foamToVTK` + ParaView; dimensionless density
  rho+ = (rho - rho_ch)/(rho_in - rho_ch).
