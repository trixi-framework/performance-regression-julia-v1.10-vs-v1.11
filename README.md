# performance-regression-julia-v1.10-vs-v1.11

## Benchmarks

1. Install Julia v1.10.4 and v1.11.0-rc1:
   ```shell
   juliaup add 1.10.4
   juliaup add 1.11.0-rc1
   ```
2. Instantiate:
   ```shell
   julia +1.10.4     --project=v1.10.4     -e 'using Pkg; Pkg.instantiate()'
   julia +1.11.0-rc1 --project=v1.11.0-rc1 -e 'using Pkg; Pkg.instantiate()'
   ```
3. Start Julia with appropriate project directory set:
   ```shell
   julia +1.10.4 --project=v1.10.4
   ```
   or
   ```shell
   julia +1.11.0-rc1 --project=v1.11.0-rc1
   ```
4. Benchmark #1 (with heavy use of LoopVectorization.jl):
   ```julia
   using Trixi
   using BenchmarkTools

   equations = CompressibleEulerEquations3D(1.4)

   volume_flux = flux_ranocha_turbo
   solver = DGSEM(polydeg=3, surface_flux=flux_ranocha_turbo,
                  volume_integral=VolumeIntegralFluxDifferencing(volume_flux))

   coordinates_min = (0.0, 0.0, 0.0)
   coordinates_max = (2.0, 2.0, 2.0)

   trees_per_dimension = (4, 4, 4)

   mesh = P4estMesh(trees_per_dimension, polydeg=1,
                    coordinates_min=coordinates_min, coordinates_max=coordinates_max,
                    periodicity=true, initial_refinement_level=1)

   semi = SemidiscretizationHyperbolic(mesh, equations, initial_condition_convergence_test,
                                       solver)

   u_ode = compute_coefficients(0.0, semi)
   du_ode = similar(u_ode)

   @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
   ```
4. Benchmark #2 (with moderate use of LoopVectorization.jl):
   ```julia
   using Trixi
   using BenchmarkTools

   equations = CompressibleEulerEquations3D(1.4)

   volume_flux = flux_ranocha
   solver = DGSEM(polydeg=3, surface_flux=flux_ranocha,
                  volume_integral=VolumeIntegralFluxDifferencing(volume_flux))

   coordinates_min = (0.0, 0.0, 0.0)
   coordinates_max = (2.0, 2.0, 2.0)

   trees_per_dimension = (4, 4, 4)

   mesh = P4estMesh(trees_per_dimension, polydeg=1,
                    coordinates_min=coordinates_min, coordinates_max=coordinates_max,
                    periodicity=true, initial_refinement_level=1)

   semi = SemidiscretizationHyperbolic(mesh, equations, initial_condition_convergence_test,
                                       solver)

   u_ode = compute_coefficients(0.0, semi)
   du_ode = similar(u_ode)

   @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
   ```

## Results

### Rocinante

#### v1.10.4, benchmark #1
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 1684 samples with 1 evaluation.
 Range (min … max):  2.940 ms …  3.626 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     2.964 ms              ┊ GC (median):    0.00%
 Time  (mean ± σ):   2.965 ms ± 19.381 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

                ▁▁▁ ▂▁▃▃▃▆▅▄█▆▄▄▅▅▃▃▄ ▁
  ▂▃▄▄▄▅▆▅▆▅▄▆█████▇█████████████████▆██▆▇▅▆▄▅▄▃▄▃▃▃▃▃▃▃▂▂▁▂ ▅
  2.94 ms        Histogram: frequency by time        2.99 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```

#### v1.11.0-rc1, benchmark #1
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 1410 samples with 1 evaluation.
 Range (min … max):  3.514 ms …  4.130 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     3.544 ms              ┊ GC (median):    0.00%
 Time  (mean ± σ):   3.546 ms ± 19.719 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

                    ▃▄▇█▅▃▂▆▂▃▂▅▂▁▃▂▁▃▁▁▁▁
  ▁▁▁▁▂▂▃▂▃▂▂▂▁▂▃▃▇███████████████████████▇▇▆▇▆▇▇▄▅▄▄▂▂▂▂▂▂▃ ▄
  3.51 ms        Histogram: frequency by time        3.57 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```

#### v1.10.4, benchmark #2
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 822 samples with 1 evaluation.
 Range (min … max):  6.036 ms …  6.692 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     6.079 ms              ┊ GC (median):    0.00%
 Time  (mean ± σ):   6.082 ms ± 25.094 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

                     ▂▆▅▇██▄▅▆▂▁▁
  ▂▂▁▁▁▁▁▁▁▂▃▂▃▃▃▃▄▆▅█████████████▇▅▅▃▃▃▃▂▂▂▃▂▁▁▂▂▁▁▂▁▂▂▁▂▁▂ ▄
  6.04 ms        Histogram: frequency by time        6.14 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```

#### v1.11.0-rc1, benchmark #2
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 697 samples with 1 evaluation.
 Range (min … max):  7.006 ms …  7.913 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     7.200 ms              ┊ GC (median):    0.00%
 Time  (mean ± σ):   7.175 ms ± 87.784 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

       ▁                    ▅▆▅█▃▃▂
  ▃▄▃▆▇█▆▅▃▄▂▁▂▁▁▂▁▁▁▁▃▃▃▅▅████████▆▅▃▄▂▂▂▁▁▂▁▁▁▁▁▁▁▁▁▁▁▂▁▂▂ ▃
  7.01 ms        Histogram: frequency by time        7.41 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```
