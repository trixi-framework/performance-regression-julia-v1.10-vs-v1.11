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

### Rocinante (AMD Ryzen Threadripper 3990X)

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

#### Versioninfo
##### v1.10.4
```
Julia Version 1.10.4
Commit 48d4fd48430 (2024-06-04 10:41 UTC)
Build Info:
  Official https://julialang.org/ release
Platform Info:
  OS: Linux (x86_64-linux-gnu)
  CPU: 128 × AMD Ryzen Threadripper 3990X 64-Core Processor
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-15.0.7 (ORCJIT, znver2)
Threads: 1 default, 0 interactive, 1 GC (on 128 virtual cores)
```

##### v1.11.0-rc1
```
Julia Version 1.11.0-rc1
Commit 3a35aec36d1 (2024-06-25 10:23 UTC)
Build Info:
  Official https://julialang.org/ release
Platform Info:
  OS: Linux (x86_64-linux-gnu)
  CPU: 128 × AMD Ryzen Threadripper 3990X 64-Core Processor
  WORD_SIZE: 64
  LLVM: libLLVM-16.0.6 (ORCJIT, znver2)
Threads: 1 default, 0 interactive, 1 GC (on 128 virtual cores)
```


### Macbook Pro (Apple M3 Pro)

#### v1.10.4, benchmark #1
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 2932 samples with 1 evaluation.
 Range (min … max):  1.635 ms … 1.788 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     1.703 ms             ┊ GC (median):    0.00%
 Time  (mean ± σ):   1.704 ms ± 5.435 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

                                             ▃█▇▄▁
  ▂▁▁▁▁▁▁▁▁▁▁▁▂▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▂▂▃▆█████▆▅▅▄▄▃▃▂▂ ▃
  1.63 ms        Histogram: frequency by time       1.72 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.

julia>
```

#### v1.11.0-rc1, benchmark #1
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 2510 samples with 1 evaluation.
 Range (min … max):  1.953 ms … 2.076 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     1.990 ms             ┊ GC (median):    0.00%
 Time  (mean ± σ):   1.992 ms ± 6.347 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

                                   ▅▅▆▆█▅▂
  ▂▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▁▂▃▅▇██████████▇▇▆▅▆▅▄▅▃▃▃▃▂▃▂ ▃
  1.95 ms        Histogram: frequency by time       2.01 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```

#### v1.10.4, benchmark #2
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 1621 samples with 1 evaluation.
 Range (min … max):  2.855 ms …   5.195 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     3.008 ms               ┊ GC (median):    0.00%
 Time  (mean ± σ):   3.084 ms ± 205.531 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

  ▁ ▁▃▆▅█▆▅▃▄▃▆▅▄▃▂▂▂▁▁▁▁▁                                    ▁
  █████████████████████████▇▇▇▇▇██▇▅█▅█▅▆█▆▇▇▇▆▇▅▅▅▆▁▆▅▆▄▄▄▄▄ █
  2.86 ms      Histogram: log(frequency) by time      3.89 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```

#### v1.11.0-rc1, benchmark #2
```julia
julia> @benchmark Trixi.rhs!($du_ode, $u_ode, $semi, 0.0)
BenchmarkTools.Trial: 1409 samples with 1 evaluation.
 Range (min … max):  3.483 ms …  4.041 ms  ┊ GC (min … max): 0.00% … 0.00%
 Time  (median):     3.544 ms              ┊ GC (median):    0.00%
 Time  (mean ± σ):   3.549 ms ± 33.919 μs  ┊ GC (mean ± σ):  0.00% ± 0.00%

                 ▃▃▃▃ ▂▁▂      ▅█▅▁
  ▂▁▂▂▂▂▂▃▄▃▃▅▅▆█████████▄▆▅▃▅██████▆▇▆▄▂▂▂▂▂▂▂▁▂▂▂▂▁▁▁▁▁▁▁▂ ▄
  3.48 ms        Histogram: frequency by time        3.64 ms <

 Memory estimate: 0 bytes, allocs estimate: 0.
```

#### Versioninfo
##### v1.10.4
```
Julia Version 1.10.4
Commit 48d4fd48430 (2024-06-04 10:41 UTC)
Build Info:
  Official https://julialang.org/ release
Platform Info:
  OS: macOS (arm64-apple-darwin22.4.0)
  CPU: 11 × Apple M3 Pro
  WORD_SIZE: 64
  LIBM: libopenlibm
  LLVM: libLLVM-15.0.7 (ORCJIT, apple-m1)
Threads: 1 default, 0 interactive, 1 GC (on 5 virtual cores)
```

##### v1.11.0-rc1
```
Julia Version 1.11.0-rc1
Commit 3a35aec36d1 (2024-06-25 10:23 UTC)
Build Info:
  Official https://julialang.org/ release
Platform Info:
  OS: macOS (arm64-apple-darwin22.4.0)
  CPU: 11 × Apple M3 Pro
  WORD_SIZE: 64
  LLVM: libLLVM-16.0.6 (ORCJIT, apple-m3)
Threads: 1 default, 0 interactive, 1 GC (on 5 virtual cores)
```
