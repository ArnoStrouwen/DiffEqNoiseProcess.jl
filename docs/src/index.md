# DiffEqNoiseProcess.jl: Noise Processes for Stochastic Modeling

Noise processes are essential in continuous stochastic modeling. The `NoiseProcess`
types are distributionally-exact, meaning they are not solutions of
stochastic differential equations and instead are directly generated according
to their analytical distributions. These processes are used as the noise term
in the SDE and RODE solvers. Additionally, the noise processes themselves can
be simulated and solved using the DiffEq common interface (including the Monte
Carlo interface).

## Installation

To install DiffEqNoiseProcess.jl, use the Julia package manager:

```julia
using Pkg
Pkg.add("DiffEqNoiseProcess")
```

## Using Noise Processes

### Passing a Noise Process to a Problem Type

`AbstractNoiseProcess`es can be passed directly to the problem types to replace
the standard Wiener process (Brownian motion) with your choice of noise. To do
this, simply construct the noise and pass it to the `noise` keyword argument:

```julia
μ = 1.0
σ = 2.0
W = GeometricBrownianMotionProcess(μ,σ,0.0,1.0,1.0)
# ...
# Define f,g,u0,tspan for a SDEProblem
# ...
prob = SDEProblem(f,g,u0,tspan,noise=W)
```

### Basic Interface

The `NoiseProcess` acts like a DiffEq solution. For some noise process `W`, you
can get its `i`th timepoint like `W[i]` and the associated time `W.t[i]`. If the
`NoiseProcess` has a bridging distribution defined, it can be interpolated to
arbitrary time points using `W(t)`. Note that every interpolated value is saved
to the `NoiseProcess` so that way it can stay distributionally correct. A plot
recipe is provided which plots the timeseries.

### Direct Simulation of the Noise Process

Since the `NoiseProcess` types are distribution-exact and do not require the
stochastic differential equation solvers, many times one would like to directly
simulate trajectories from these proecesses. The `NoiseProcess` has a
`NoiseProcessProblem` type:

```julia
NoiseProblem(noise,tspan)
```

for which `solve` works. For example, we can simulate a distributionally-exact
Geometric Brownian Motion solution by:

```julia
μ = 1.0
σ = 2.0
W = GeometricBrownianMotionProcess(μ,σ,0.0,1.0,1.0)
prob = NoiseProblem(W,(0.0,1.0))
sol = solve(prob;dt=0.1)
```

`solve` requires the `dt` is given, the solution it returns is a `NoiseProcess`
which has stepped through the timespan. Because this follows the common interface,
all of the normal functionality works. For example, we can use the Monte Carlo
functionality as follows:

```julia
monte_prob = MonteCarloProblem(prob)
sol = solve(monte_prob;dt=0.1,num_monte=100)
```

simulates 100 Geometric Brownian Motions.

### Direct Interface

Most of the time, a `NoiseProcess` is received from the solution of a stochastic
or random differential equation, in which case `sol.W` gives the `NoiseProcess`
and it is already defined along some timeseries. In other cases, `NoiseProcess`
types are directly simulated (see below). However, `NoiseProcess` types can also
be directly acted on. The basic functionality is given by `calculate_step!`
to calculate a future time point, and `accept_step!` to accept the step. If steps
are rejected, the Rejection Sampling with Memory algorithm is applied to keep
the solution distributionally exact. This kind of stepping is done via:

```julia
W = WienerProcess(0.0,1.0,1.0)
dt = 0.1
W.dt = dt
u = nothing; p = nothing # for state-dependent distributions
calculate_step!(W,dt,u,p)
for i in 1:10
  accept_step!(W,dt,u,p)
end
```

## Contributing

- Please refer to the
  [SciML ColPrac: Contributor's Guide on Collaborative Practices for Community Packages](https://github.com/SciML/ColPrac/blob/master/README.md)
  for guidance on PRs, issues, and other matters relating to contributing to ModelingToolkit.
- There are a few community forums:
    - the #diffeq-bridged channel in the [Julia Slack](https://julialang.org/slack/)
    - [JuliaDiffEq](https://gitter.im/JuliaDiffEq/Lobby) on Gitter
    - on the [Julia Discourse forums](https://discourse.julialang.org)
    - see also [SciML Community page](https://sciml.ai/community/)
