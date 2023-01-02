
# 20221231
Code from [QuantEco](https://julia.quantecon.org/getting_started_julia/julia_by_example.html#id10)

## Loop
Built-in funciton
```{code-cell} julia
n = 1000000
@time ep = rand(n);

0.001511 seconds (2 allocations: 7.629 MiB)
```
Naive sytle
```{code-cell} julia
ep = zeros(n)
@time for i in 1:n
    ep[i] = randn()
end

0.126833 seconds (4.00 M allocations: 76.278 MiB, 6.07% gc time)
```

Better style
```{code-cell} julia
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
```{code-cell} julia
ep_sum = 0.0
@time for ep_val in ep[1:end]
    ep_sum = ep_sum + ep_val
end
ep_mean = ep_sum / n

0.137116 seconds (4.00 M allocations: 83.916 MiB, 10.05% gc time)
```
Loop over indexes. There is a litter slower. 
```{code-cell} julia
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
Julia allows to write code in a natural language manner. For example we want to define a linear function `y(x) = px + q` in Julia. We can write in a `function` cell
```{code-cell} julia
p = 0.1
q = 0.2
function f(x)
    return p * x +q
end
```
or in the following manner
```{code-cell} julia
p = 0.1
q = 0.2
f(x) = p * x + q
``` 
