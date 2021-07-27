# UMAT
## _ABAQUS user Subroutines_
## _Notes:_
The code is written in Fortran _FreeForm_ and requires compiler settings to be adjusted accordingly.
If using the Intel Fortran Compiler, the compiler directive should be added to ABAQUS Environment file.
In ABAQUS previous versions, this file is located in the installation directory under
`solvers_install_dir/os/SMA/site/abaqus_v6.env`
and in newer versions under
`solvers_install_dir/[os_version]/SMA/site/[os_version].env`.  (_Example:_ [os_version] = win86_64)

Edit the `.env` file to add `'/free'` (single quotes included) as a parameter of compile_fortran variable.
For example, it might look like this:
```python
compile_fortran=['ifort',
				 '/free',
                 '/c', '/fpp', '/extend-source', 
                 '/DABQ_WIN86_64',  '/DABQ_FORTRAN',
                 '/iface:cref', '/recursive',
				 '/names:lowercase',
                 '/Qauto',  # <-- important for thread-safety of parallel user subroutines
                 '/align:array64byte',
                 '/Qpc64',                    # set FPU precision to 53 bit significand
                 '/Qprec-div', '/Qprec-sqrt', # improve precision of FP divides and sqrt
                 '/Qfma-',                    # disable floating point fused multiply-add
                 '/fp:precise',               # floating point model: precise 
                 '/Qimf-arch-consistency:true', # math library consistent results
                 '/Qfp-speculation:safe',     # floating point speculations only when safe
                 '/Qprotect-parens',          # honor parenthesis during expression evaluation
                 '/Qfp-stack-check',          # enable stack overflow protection checks
                 '/reentrancy:threaded',      # important for thread-safety
                 #'/Qinit=zero','/Qinit=arrays',  # automatically initialize all arrays to zero
                 #'/Qinit=snan', '/Qinit=arrays', # automatically initialize all arrays to SNAN
                 '/QxSSE3', '/QaxAVX',        # generate SSE3, SSE2, and SSE instructions
                 # '/Od', '/Ob0',  # <-- Disable Optimization for Debugging
                 # '/Zi',          # <-- Debugging Information
                 '/include:%I', '/include:'+abaHomeInc, '%P']
```
It also also worth mentioning that the UMAT file starts with the indicator `!DEC$ FREEFORM`

## Mohr-Coulomb with Tension Cut-off (Smeared, predefined failure plane)
##### Subroutine file: `predef_loc_return.for`
This file includes all the subroutnes needed and no other file is required to compile.
The beggining of the file includes required modules `module class_math` and `module plastic_framework`.

Material properties are introduced under `subroutine propinit(o_props,props)` included in `module plastic_framework`.

### Examples
#### Failure in Tension
##### Input file: `Interface\one_elem_tensile.inp`, Output file: `Interface\one_elem_tensile.odb`
This example includes one element with the interfacial properties sandwitched between to elements with elastic behaviour.
The orientation of failure plane is assumed to be parallel to XZ plane and a tensile loading condition (displacement controlled) is applied, causing the stress in the element to reach the tensiel strength and softening resonse of the material.
Material properties:
| Property|Value|
|:---------------:|:-------------:|
| Bulk Modulus (N/m^2) | 6.4394e+08 |
| Shear Modulus (N/m^2) | 8.0189e+08 |
| Cohesion (N/m^2) | 0.35e+06 |
| Tensile Strength (N/m^2) | 0.24e+06 |
| Internal friction coefficient | 0.83 |
| Shear softening parameter (1/m) | 500 |
| Tensile Fracture Energy Release Rate |1|
| Angle of weak plane with horizontal | 0 |

#### Failure in Shear
##### Input file: `Interface\one_elem_shear.inp`, Output file: `Interface\one_elem_shear.odb`
The same model setup and material properties is used to model shear behaviour of a single element. An initial normal load is applied normal to the predefined failure plane, and shear displacement is applied on top.

