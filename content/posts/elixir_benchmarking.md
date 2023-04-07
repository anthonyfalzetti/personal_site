---
title: "Elixir Benchmarking"
date: 2023-04-07T14:39:06-06:00
draft: false
tags:
  - elixir
---

A couple of days ago I faced a straight forward problem:

Given a list of elements
The value of index 2 is a count
Starting with the element after the count
Return a list of lists each with 3 elements (triplet)

Additional note: the count will be >= 48

```elixir
elements = [ "skip", "skip", 4, "list_1_element_1", "list_1_element_2", "list_1_element_3", "list_2_element_1", "list_2_element_2", "list_2_element_3", "list_3_element_1", "list_3_element_2", "list_3_element_3", "list_4_element_1", "list_4_element_2", "list_4_element_3"]

output = [
  [ "list_1_element_1", "list_1_element_2", "list_1_element_3" ],
  [ "list_2_element_1", "list_2_element_2", "list_2_element_3" ],
  [ "list_3_element_1", "list_3_element_2", "list_3_element_3" ],
  [ "list_4_element_1", "list_4_element_2", "list_4_element_3" ]
]
```

2 ways I thought to solve this:

### Iteration
This route would be something like:
1. create a range of integers 0 to count
2. iterate over the range (step)
3. calculate the working index ((step * 3) + count + 1)
4. slice the list from the working index for the next 3 elements

```elixir
  @count_index 2

  def iterate(list) do
    count = Enum.(list, @count_index)
    range = (0..(count - 1))

    Enum.reduce(range, [], fn(iteration, acc) ->
      triplet = find_triplet(list, iteration)
      [triplet|acc]
    end)
    |> Enum.reverse()
  end

  defp find_triplet(list, 0) do
    Enum.slice(list, (@count_index + 1), 3)
  end
  defp find_triplet(list, iteration_number) do
    Enum.slice(list, ((iteration_number * 3) + (@count_index + 1)) , 3)
  end
```

### Chunk It
This route would be something like:
1. Drop all elements of the list upto the count + 1
2. [Enum.chunk_every/2](https://hexdocs.pm/elixir/1.10.4/Enum.html#chunk_every/2)

```elixir
  @count_index 2

  def chunk(list) do
    list
    |> Enum.drop((@count_index + 1))
    |> Enum.chunk_every(3)
  end
```

From a readability standpoint there is a clear winner in my mind: *Chunk it*. It's easy to read, and I can determine what it is going to do at a glance, but is there a tradeoff of memory usage or time?

Method to create some test lists for benchmarking:
```elixir
def create_list(n) do
 elems = (1..n)
 |> Enum.flat_map(fn(_x) -> ["elem1","elem2","elem3"] end)

 ["skip", "skip", n | elems]
end
```

## Benchmarking with Benchee

When looking for the elixir equivalent of ruby's [benchmark](https://ruby-doc.org/stdlib-2.7.1/libdoc/benchmark/rdoc/Benchmark.html) I came across [Benchee](https://github.com/bencheeorg/benchee).  So using that let's see if there are any tradeoffs when using [Chunk it](#chunk-it) vs [Iterate](#iteration). Since benchee can accept and inputs we can do a comparison with sizes.


```elixir
run(
  %{
    "iterate" => fn(input) -> iterate(input) end,
    "chunk" => fn(input) -> chunk(input) end
  },
  inputs: %{
    "small" => create_list(1),
    "Medium" => create_list(24),
    "Biggest" => create_list(48)
  },
  time: 10,
  memory_time: 2
)
```

```
$ mix run lib/triplet_finder/benchmark.2

Benchmark suite executing with the following configuration:
warmup: 2 s
time: 10 s
memory time: 2 s
parallel: 1
inputs: Biggest, Medium, small
Estimated total run time: 1.40 min

Benchmarking chunk with input Biggest...
Benchmarking chunk with input Medium...
Benchmarking chunk with input small...
Benchmarking iterate with input Biggest...
Benchmarking iterate with input Medium...
Benchmarking iterate with input small...

##### With input Biggest #####
Name              ips        average  deviation         median         99th %
chunk         34.66 K       28.85 μs    ±33.28%       27.90 μs       55.90 μs
iterate       16.52 K       60.52 μs    ±46.68%       58.90 μs       96.90 μs

Comparison:
chunk         34.66 K
iterate       16.52 K - 2.10x slower +31.67 μs

Memory usage statistics:

Name       Memory usage
chunk          20.13 KB
iterate         3.15 KB - 0.16x memory usage -16.98438 KB

**All measurements for memory usage were the same**

##### With input Medium #####
Name              ips        average  deviation         median         99th %
chunk         67.45 K       14.83 μs    ±77.46%       13.90 μs       32.90 μs
iterate       53.71 K       18.62 μs    ±78.24%       17.90 μs       32.90 μs

Comparison:
chunk         67.45 K
iterate       53.71 K - 1.26x slower +3.79 μs

Memory usage statistics:

Name       Memory usage
chunk           9.95 KB
iterate         1.76 KB - 0.18x memory usage -8.18750 KB

**All measurements for memory usage were the same**

##### With input small #####
Name              ips        average  deviation         median         99th %
iterate        1.53 M        0.65 μs  ±6120.45%        0.90 μs        0.90 μs
chunk          0.74 M        1.35 μs  ±3069.60%        0.90 μs        1.90 μs

Comparison:
iterate        1.53 M
chunk          0.74 M - 2.07x slower +0.70 μs

Memory usage statistics:

Name       Memory usage
iterate           184 B
chunk             672 B - 3.65x memory usage +488 B

**All measurements for memory usage were the same**
```

Benchmarking Summary:

|        | small   | medium | large |
| ------ | ------- | ------ | ----- |
| Faster | Iterate | Chunk  | Chunk |
| Memory | Iterate | Chunk  | Chunk |

### Conclusion

So with this comparison in mind, I am in a better place to determine which thought process would be the most beneficial.

Chunk It is both easier to read and for medium/large use cases it is the more efficient option, but for small use cases, there is a trade-off efficiency for readability.

There will always be more than one way to solve a problem, but deciding which is the most efficient may not always be obvious. Benchee makes it simple to compare the solutions' tradeoffs and improves my confidence in which solution I decide to implement. When creating a pull request I enjoy being able to demonstrate why I chose to implement a given solution.A couple of days ago I faced a  straight forward problem:

Given a list of elements
The value of index 2 is a count
Starting with the element after the count
Return a list of lists each with 3 elements (triplet)

Additional note: the count will be >= 48

```elixir
elements = [ "skip", "skip", 4, "list_1_element_1", "list_1_element_2", "list_1_element_3", "list_2_element_1", "list_2_element_2", "list_2_element_3", "list_3_element_1", "list_3_element_2", "list_3_element_3", "list_4_element_1", "list_4_element_2", "list_4_element_3"]

output = [
  [ "list_1_element_1", "list_1_element_2", "list_1_element_3" ],
  [ "list_2_element_1", "list_2_element_2", "list_2_element_3" ],
  [ "list_3_element_1", "list_3_element_2", "list_3_element_3" ],
  [ "list_4_element_1", "list_4_element_2", "list_4_element_3" ]
]
```

2 ways I thought to solve this:

### Iteration
This route would be something like:
1. create a range of integers 0 to count
2. iterate over the range (step)
3. calculate the working index ((step * 3) + count + 1)
4. slice the list from the working index for the next 3 elements

```elixir
  @count_index 2

  def iterate(list) do
    count = Enum.(list, @count_index)
    range = (0..(count - 1))

    Enum.reduce(range, [], fn(iteration, acc) ->
      triplet = find_triplet(list, iteration)
      [triplet|acc]
    end)
    |> Enum.reverse()
  end

  defp find_triplet(list, 0) do
    Enum.slice(list, (@count_index + 1), 3)
  end
  defp find_triplet(list, iteration_number) do
    Enum.slice(list, ((iteration_number * 3) + (@count_index + 1)) , 3)
  end
```

### Chunk It
This route would be something like:
1. Drop all elements of the list upto the count + 1
2. [Enum.chunk_every/2](https://hexdocs.pm/elixir/1.10.4/Enum.html#chunk_every/2)

```elixir
  @count_index 2

  def chunk(list) do
    list
    |> Enum.drop((@count_index + 1))
    |> Enum.chunk_every(3)
  end
```

From a readability standpoint there is a clear winner in my mind: *Chunk it*. It's easy to read, and I can determine what it is going to do at a glance, but is there a tradeoff of memory usage or time?

Method to create some test lists for benchmarking:
```elixir
def create_list(n) do
 elems = (1..n)
 |> Enum.flat_map(fn(_x) -> ["elem1","elem2","elem3"] end)

 ["skip", "skip", n | elems]
end
```

## Benchmarking with Benchee

When looking for the elixir equivalent of ruby's [benchmark](https://ruby-doc.org/stdlib-2.7.1/libdoc/benchmark/rdoc/Benchmark.html) I came across [Benchee](https://github.com/bencheeorg/benchee).  So using that let's see if there are any tradeoffs when using [Chunk it](#chunk-it) vs [Iterate](#iteration). Since benchee can accept and inputs we can do a comparison with sizes.


```elixir
run(
  %{
    "iterate" => fn(input) -> iterate(input) end,
    "chunk" => fn(input) -> chunk(input) end
  },
  inputs: %{
    "small" => create_list(1),
    "Medium" => create_list(24),
    "Biggest" => create_list(48)
  },
  time: 10,
  memory_time: 2
)
```

```
$ mix run lib/triplet_finder/benchmark.2

Benchmark suite executing with the following configuration:
warmup: 2 s
time: 10 s
memory time: 2 s
parallel: 1
inputs: Biggest, Medium, small
Estimated total run time: 1.40 min

Benchmarking chunk with input Biggest...
Benchmarking chunk with input Medium...
Benchmarking chunk with input small...
Benchmarking iterate with input Biggest...
Benchmarking iterate with input Medium...
Benchmarking iterate with input small...

##### With input Biggest #####
Name              ips        average  deviation         median         99th %
chunk         34.66 K       28.85 μs    ±33.28%       27.90 μs       55.90 μs
iterate       16.52 K       60.52 μs    ±46.68%       58.90 μs       96.90 μs

Comparison:
chunk         34.66 K
iterate       16.52 K - 2.10x slower +31.67 μs

Memory usage statistics:

Name       Memory usage
chunk          20.13 KB
iterate         3.15 KB - 0.16x memory usage -16.98438 KB

**All measurements for memory usage were the same**

##### With input Medium #####
Name              ips        average  deviation         median         99th %
chunk         67.45 K       14.83 μs    ±77.46%       13.90 μs       32.90 μs
iterate       53.71 K       18.62 μs    ±78.24%       17.90 μs       32.90 μs

Comparison:
chunk         67.45 K
iterate       53.71 K - 1.26x slower +3.79 μs

Memory usage statistics:

Name       Memory usage
chunk           9.95 KB
iterate         1.76 KB - 0.18x memory usage -8.18750 KB

**All measurements for memory usage were the same**

##### With input small #####
Name              ips        average  deviation         median         99th %
iterate        1.53 M        0.65 μs  ±6120.45%        0.90 μs        0.90 μs
chunk          0.74 M        1.35 μs  ±3069.60%        0.90 μs        1.90 μs

Comparison:
iterate        1.53 M
chunk          0.74 M - 2.07x slower +0.70 μs

Memory usage statistics:

Name       Memory usage
iterate           184 B
chunk             672 B - 3.65x memory usage +488 B

**All measurements for memory usage were the same**
```

Benchmarking Summary:

|        | small   | medium | large |
| ------ | ------- | ------ | ----- |
| Faster | Iterate | Chunk  | Chunk |
| Memory | Iterate | Chunk  | Chunk |

### Conclusion

So with this comparison in mind, I am in a better place to determine which thought process would be the most beneficial.

Chunk It is both easier to read and for medium/large use cases it is the more efficient option, but for small use cases, there is a trade-off efficiency for readability.

There will always be more than one way to solve a problem, but deciding which is the most efficient may not always be obvious. Benchee makes it simple to compare the solutions' tradeoffs and improves my confidence in which solution I decide to implement. When creating a pull request I enjoy being able to demonstrate why I chose to implement a given solution.