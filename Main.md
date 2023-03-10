
Code from [QuantEco](https://julia.quantecon.org/getting_started_julia/julia_by_example.html#id10)

## Type
**Use float number as soon as you can.** Just as in MMT, calculations on the `Float` type and `Int` type have siginificant difference. It is because the `Int` type recalls a second-level code to define it.

**Convert float to Int**. There are cases that we need `Int` number such as the index in a loop. For example, we want to simulate a continous time-dependent markov chain. We need to discret the time and use `Int((tf-ti)/deltaT)` to determine the total number of steps. But `tf` and `ti` are `float` numbers, and it might raise the errror `Inexact typeError Int()`. The solution is to use `round(Int, xxx)`. 
```julia
ti = 2.3
delta = 0.01
Int(ti/delta) 
> InexactError: Int64(229.99999999999997)

round(Int, ti/delta)
> 230
```

## Loop
Built-in funciton
```julia
n = 1000000
@time ep = rand(n);

0.001511 seconds (2 allocations: 7.629 MiB)
```
Naive sytle
```julia
ep = zeros(n)
@time for i in 1:n
    ep[i] = randn()
end

0.126833 seconds (4.00 M allocations: 76.278 MiB, 6.07% gc time)
```

Better style
```julia
@time begin
ep = zeros(n)
for i in eachindex(ep)
    ep[i] = randn()
end
end

0.124869 seconds (4.00 M allocations: 76.278 MiB, 6.24% gc time)
```
Function `eachindex` does not make the loop run faster, but make the code more flexibility.

Loop directly over arrays. 
```julia
ep_sum = 0.0
@time for ep_val in ep[1:end]
    ep_sum = ep_sum + ep_val
end
ep_mean = ep_sum / n

0.137116 seconds (4.00 M allocations: 83.916 MiB, 10.05% gc time)
```
Loop over indexes. There is a litter slower. 
```julia
@time begin
ep_sum = 0.0
for j in eachindex(ep)
    ep_sum += ep[j]
end
ep_mean = ep_sum / n
end

0.145290 seconds (5.00 M allocations: 91.537 MiB, 9.92% gc time)
```
## Specific feature of Julia
**Natural code**. Julia allows to write code in a natural language manner. For example we want to define a linear function `y(x) = px + q` in Julia. We can write in a `function` cell
```julia
p = 0.1
q = 0.2
function f(x)
    return p * x +q
end
```
or in the following manner
```julia
p = 0.1
q = 0.2
f(x) = p * x + q
``` 

**Broadcast.** Mathematica has wonderful list operations and its functions can act on a list. Julia has the broadcast operator `.` that does the same thing.
```julia
xx = [0., pi/6., pi/2.]
sin.(xx)
# is equivalent to 
# broadcast(sin, xx)
# and map(sin, xx)

3-element Vector{Float64}:
 0.0
 0.49999999999999994
 1.0
```
**Named parameters.** In MMT, there are two types of parameters in a function, the necessary paramters and the additional paramters which provides modification to the function and often have default values. In Julia, the two types of parameters are seperated by `;` in the argument. 
```julia
function binomial_rv(n, p; m=1000)
    xlis = [binomial_r(n, p) for _ in 1:m]
    histogram(xlis)
end
```
**Shorhand notation for If**
Julia allows to write the `if` environment in a more compact form which does not affect the speed. For example, 
```julia
u = rand()
# Usual form
if u < 0.5
    count +=1 
else 
    count = 0
end

# Compact form
count = u < 0.5 ? count + 1 : 0 
```
**Looping without Indices**
To loop through pairs from two sequences, use `zip()`
```julia
men = ("Caesar", "Brutus")
women = ("Portia", "Calpurnia")
for (man, woman) in zip(men, women)
    println("$woman is the wife of $man")
end

Portia is the wife of Caesar
Calpurnia is the wife of Brutus
```
If you need the index as well, use `enumerate`
```julia
for (i, man) in enumerate(men)
    woman = women[i]
    println("$woman is the wife of $man")
end

Portia is the wife of Caesar
Calpurnia is the wife of Brutus
```

**Generators**
For those functionsthat accepts iterators and arrays, use the former to avoid allocating and storing any temporary values. 
```julia
xs = 1:10000
f(x) = x^2
@btime sum([f(x) for x in $xs])
@btime sum(f.(xs))
@btime sum(f(x) for x in xs) %Drop the [] brackets.

8.037 ??s (2 allocations: 78.17 KiB)
9.378 ??s (6 allocations: 78.28 KiB)
138.846 ns (2 allocations: 48 bytes)
```
Notice that the first two cases are nearly identical, but the third has a speedup of over 80x and use far more less memory.

## List Operation
**Take part of a sequence.**
Julia provides `first` and `last` to take the first element and last element of a set. But it is usually not enough. Take a simulation of jump processs under continous measurement for example. We would store the results by a  four-value tuple `(work, meas, reality, t)`. Here `work` means work on the system at time `t`, `meas` for the measurement outcome which can be wrong, and `reality` is for the true state of the system. As we generate a sequence of the four-value tuple, we would like operate on the lists by different conditions. 
The squence of work and time is easy. Just use the `broadcast` operation.
```julia
worklis = first.(wholelis)
timelis = last.(wholelis)
```
The sequence of middle elements can be taken by generator. (Other more elegant way?)
```julia
measlis = [x[2] for x in wholelis]
realitylis = [x[3] for x in wholelis]
```

**Pattern match.**
Say, we want to take all elements that satisfy a rule, Julia provides an elegent way by broadcast 
```julia
x = rand(100)
x[x .> 0.5] # Take all elements that are >0.5.
```
For complex functions, use `findall` function. `findall` return a list of indexes of the desired elements. 
```julia
xlis = rand((100, 100))
xlis(findall(x->x[2] > 0.5, xlis))
```

**Join vectors**. Use `vcat` function. 
```julia
xlis = rand(10)
ylis = rand(12)
zlis = rand(13)
vcat(xlis, ylis, zlis)
```

**Operations in String**.
A quick way to print the value of a variable in a string is to use `$`
```julia
x = 10
"x = $x"

"x = 10"
```

A quick way to concatenate (join) two strings is to `*`
```julia
"Cae"*"sar"

"Caesar"
```
## Tools for Performance Analysis
`@time`: return the time cost. If you want to evalute a body of code, add `begin` and `end`
```julia
@time begin
func...
end
```

`@btime`: return the average cost after many round evaluation. 

`@benchmark` from the package `BenchmarkTools`. Detail version of `btime`. 

`@code_warntype` checks the unstable variable type in your code. The unstable variable is colored in **red**. So you better add annotation such as `xx::Float` 
to your variable. 

`@profview` from the package `ProfileView`. It will generate a graph containing many colored blocks, each corresponding to a line of your code. The bad code is colored in **red**. 

## Import/Export Data
You can use `save_object()` and `load_object()` from package `JDL`, or use `writedlm()` and `readdlm()` from package `DelimitedFiles`. 
For the data saved for Julia, I use `JDL`; for the date saved for MMT, I use `DelimitedFiles`. 
```julia
x = [1, "a"]
save_object("xlis.jdl2", x)
y = load_object("xlis.jdl2")

writedlm("xlis.csv", x)
y = readdlm("xlis.csv)
```

**I/O files to other directory.** See [JuliaDataScience](https://juliadatascience.io/filesystem). First obtain your working directory and then use `joinpath` function. Say your are working on a script `main.jl` under a directory "Project", and you want to save data in a subdirectory "Project/Data". Following code does the job. 
```julia
xlis = rand(10)
root = dirname(@__FILE__)
> "/Users/mac/Nutstore Files/Research/Auxillary Tools/Julia/2023 Information

datapath = joinpath(root, "Data/")
save_object(joinpath(datapath, "xlis.jld2"),xlis)
```

# Issuses Not related to Code
**JupterNoteBook Error:** `_xsrf argument missing from POST` and you can't save the notebook. 
*Soluton:* Refresh your homepage page "http://localhost:8888/tree/[yourAddress]". 
