# Running ITensor and Julia Codes

## Basic Example Code Template

The basic outline of a code which uses the ITensor library is as follows

```julia
using ITensors

let
  # ... your own code goes here ...
  # For example:
  i = Index(2,"i")
  j = Index(3,"j")
  T = randomITensor(i,j)
  @show T
end
```

The reason we recommend the `let...end` block is that global scope 
 in Julia can have some surprising behaviors, at least through Julia version 1.4. 
Putting your code into a `let` block avoids these issues.

## Running a Script

Now say you put the above code into a file named `code.jl`. Then you can run
this code on the command line as follows

```
$ julia code.jl
```

This script-like mode of running Julia is convenient for running longer jobs,
such as on a cluster.

## Running Interactively

However, sometimes you want to do rapid development when first writing and 
testing a code. For this kind of work, the long startup and compilation times
currently incurred by the 1.x versions of Julia can be a nuisance. Fortunately
there is a nice solution: repeatedly load your code into a running Julia session.

To set up this kind of session, take the following steps:

1. Enter the interactive mode of Julia, by inputting the command `julia` on the command line. You will now be in the Julia "REPL" (read-eval-print loop) with the prompt `julia>` on the left of your screen.

2. To run a code such as the `code.jl` file discussed above, input the command
   ```
   julia> include("code.jl")
   ```
   Note that you must be in the same folder as `code.jl` for this to work; otherwise input the entire path to the `code.jl` file. The code will run and you will see its output in the REPL.

3. Now say you want to modify and re-run the code. To do this, just edit the file in an editor in another window, without closing your Julia session. Now run the command 
   ```
   julia> include("code.jl")
   ```
   again and your updated code will run, but this time skipping any of the precompilation overhead incurred on previous steps.

The above steps to running a code interactively has a big advantage that you only have to pay the startup time of compiling ITensor and other libraries you are using once. Further changes to your code only incur very small extra compilation times, facilitating rapid development.

## Compiling an ITensor System Image

The above strategy of running code in the Julia REPL (interactive mode) works well, but still incurs a large start-up penalty for the first run of your code. Fortunately there is a nice way around this issue too: compiling ITensors.jl and making a system image built by the [PackageCompiler.jl](https://github.com/JuliaLang/PackageCompiler.jl) library.

To use this approach, we have provided a convenient one-line command:

```julia
julia> using ITensors; ITensors.compile()
```

Once ITensors.jl is installed, you can just run this command in an interactive Julia session. It can take a few minutes to run, but you only have to run it once for a given version of ITensors.jl. When it is done, it will create a file `sys_itensors.so` in the directory `~/.julia/sysimages/`.

To use the compiled system image together with Julia, run the `julia` command (for interactive mode or scripts) in the following way:

```
$ julia --sysimage ~/.julia/sysimages/sys_itensors.so
```

A convenient thing to do is to make an alias in your shell for this command. To do this, edit your `.bashrc` or `.zshrc` or similar file for the shell you use by adding the following line:

```
alias julia_itensors="julia --sysimage ~/.julia/sysimages/sys_itensors.so -e \"using ITensors\" -i "
```

where of course you can use the command name you like when defining the alias. Now running commands like `julia_itensors code.jl` or `julia_itensors` to start an interactive session will have the ITensor system image pre-loaded and you will notice significantly faster startup times. The arguments `-e \"using ITensors\" -i` make it so that running `julia_itensors` also loads the ITensor library as soon as Julia starts up, so that you don't have to type `using ITensors` every time.