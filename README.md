# Quantikz.jl

[![GitHub Workflow Status](https://img.shields.io/github/workflow/status/Krastanov/Quantikz/CI)](https://github.com/Krastanov/Quantikz/actions?query=workflow%3ACI+branch%3Amain)
[![Test coverage from codecov](https://img.shields.io/codecov/c/gh/Krastanov/Quantikz?label=codecov)](https://codecov.io/gh/Krastanov/Quantikz)

A minimal package for drawing quantum circuits using the `quantikz` and `tikz` TeX libraries.

To install it use:

```julia
] add Quantikz
```

See the [attached notebook](https://github.com/Krastanov/Quantikz/blob/main/Quantikz.ipynb) for examples.

- This library can generate `tex` files with the markup necessary for a given circuit.
- Creating `pdf` files is possible (using the `Tectonic.jl` package so you do not even need to pre-install a `TeX` distribution).
- If you have the `convert` command available on your system, `png` files can also be generated directly. In that case the interactive use of this package is as simple as:

```julia
circuit = [CNOT(1,2),Measurement(2)]
displaycircuit(circuit) # you can set a DPI parameter
```

To save a `png` use:

```julia
savepng(circuit, filename) # you can set a DPI parameter
```

If you do not have a working version of `convert` (which has been problematic on a number of recent operating systems), you can still generate a PDF containing the circuit with:

```julia
savepdf(circuit, filename)
```

You can still view the corresponding TeX string or save it to file with:

```julia
circuit2string(circuit)
savetex(circuit, filename)
```

If you need some of the more advanced features of the `quantikz` TeX macros that are not implemented here yet, you can edit the string directly, or more conveniently, you can generate the 2D array of macros that makes up the string:

```
julia> circuit2table([CNOT(1,2),CNOT(2,3)])
3×4 Array{String,2}:
 "\\qw"  "\\ctrl{1}"  "\\qw"       "\\qw"
 "\\qw"  "\\targ{}"   "\\ctrl{1}"  "\\qw"
 "\\qw"  "\\qw"       "\\targ{}"   "\\qw"
```

`table2string` can be used for the final conversion.

## Built-in quantum circuit operations

`CNOT`, `CPHASE`, `SWAP`, `H`, `P`, `Id`, a generic single qubit gate `U`, a generic measurement `Measurement`, and a parity check measurement `ParityMeasurement`.

The general purpose `MultiControl(controls, ocontrols, targets, targetXs)` can be used to create an arbitrary combination of muticontrol multiqubit gates. Each argument lists the indices of qubits that should get a certain symbol: `controls` is filled circles, `ocontrols` is empty circles, `targets` is the NOT symbol, and `targetXs` is the X symbols.

For named controled gates use `MultiControlU(str, controls, ocontrols, targets)`.

For noise events, you can use `Noise(targets)` or `NoiseAll()`.

For examples of these operations, see the [attached notebook](https://github.com/Krastanov/Quantikz/blob/main/Quantikz.ipynb).

## Custom objects

For your `CustomQuantumOperation` simply define a `QuantikzOp(op::CustomQuantumOperation)` that converts your object to one of the built-in objects.

If you need more freedom for your custom quantum operation, simply define `update_table!(table,step,op::CustomQuantumOperation)` that directly modifies the `quantikz` table and define `affectedqubits(op::CustomQuantumOperation)` that gives the indices of qubits involved in the operation. Instead of returning an array of indices `affectedqubits` can also return the symbol `:all` which tells the layout engine that all qubits are used in this stage of the circuit.

Internally, this library converts the array of circuit operations to a 2D array of `quantikz` macros which is then converted to a single TeX string, which is then compiled with a call to `pdflatex`.

## LaTeX-free alternatives

If all you need is the LaTeX string to put in your manuscript, this library is sufficient: simply run `circuit2string` instead of `displaycircuit`.

For rendering in a notebook, however, this library does require a `convert` binary to be preinstalled. An alternative is the [`YaoPlots.jl`](https://github.com/QuantumBFS/YaoPlots.jl) which can draw directly from Julia as it does not depend on external rendering sofware. `YaoPlots.jl` however does not support LaTeX yet.
