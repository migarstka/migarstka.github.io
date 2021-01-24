---
layout: page 
title: Automatic bug discovery with git bisect run  
shorttitle: Bad commits and git bisect
permalink: /gitbisect/
tags: git julia
---

Working on a software project I often realize that I introduced a bug in one of my previous commits.
Especially if the number of candidate commits for the error is large it can be quite tedious to find the problematic commit.
`git bisect` is a helpful tool to find the problematic commit using a binary search over the set of candidate commits.
If your commit history looks like this:

```
A -- B -- C -- D -- E -- F -- G -- H -- I -- HEAD
          ^                                    ^
         works                       doesn't work
```
and you know that your code worked at commit `C` but not at the `HEAD`, Git will checkout the repository at various commits and *ask* you whether that commit causes the problems. Consequently, you'll find the bug in `O(log(n))` steps where `n` is the number of candidate commits that could cause the error. The cool thing about `git bisect` is that you can provide a script to tell Git whether a certain commit is still working or not and automate the whole thing.
Moreover, this also works for finding changes that introduce performance problems.


## A bug in mergesort
I have been working on a project that implements mergesort in Julia. The project folder looks like this:

```
MergeSortProject/
|-- merge_sort.jl
|-- unit_test.jl
```
The 
<span class="sidenote">
<input
aria-label="Show sidenote"
type="checkbox"
id="sidenote__checkbox--1"
class="sidenote__checkbox">
<label
tabindex="0"
title="{{ include.content }}"
aria-describedby="sidenote-1"
for="sidenote__checkbox--1"
class="sidenote__button sidenote__button--number-1
">files</label>
<small
id="sidenote-4"
class="sidenote__content sidenote__content--number-1">
<span class="sidenote__content-parenthesis
"> (sidenote: </span>
The details of the script don't really matter here. If you want to try and find the bug look closer at the first function `merge_sort!`.
<span class="sidenote__content-parenthesis">)</span>
</small>
</span> look like this:

**merge_sort.jl**
{% highlight julia %}
"Sort array `a` using merge sort algorithm. Stable. Runtime complexity: O(n log(n)), Space complexity: O(n)."
function merge_sort!(arr::AbstractArray{T}) where {T <: Real}
    length(arr) <= 1 && return arr
    
    helper = similar(arr)
    _merge_sort!(arr, 1, length(arr), helper)
    tmp = arr[end]
    arr[end] = arr[1]
    arr[1] = tmp
end

"Split array in half and recursively call this function."
function _merge_sort!(arr::AbstractArray{T}, l::Int64, r::Int64, helper::AbstractArray{T}) where {T <: Real}
    # single element: already sorted
    if r - l <= 0
        return nothing
    end

    # split list and sort individually
    m = div(r - l + 1, 2) + l - 1

    _merge_sort!(arr, l, m, helper)
    _merge_sort!(arr, m + 1, r, helper)

    # merge them back together
    merge!(arr, l, m, r, helper)

end

"Merge two sorted arrays in sorted fashion."
function merge!(arr::AbstractArray{T}, l::Int64, m::Int64, r::Int64, helper::AbstractArray{T}) where {T <: Real}    
    for k = l:r
        helper[k] = arr[k]
    end
    helper_l = l
    helper_r = m + 1
    current = l

    # compare elements in left and right partition and copy smaller one 
    while helper_l <= m && helper_r <= r
        if helper[helper_l] <= helper[helper_r]
            arr[current] = helper[helper_l]
            helper_l += 1            
        else
            arr[current] = helper[helper_r]
            helper_r += 1
        end
        current += 1 
    end
    
    # transfer any left overs from left partition (greater than all elem in right partition)
    if helper_l <= m 
         for k = helper_l:m
            arr[current] = helper[k]
            current += 1
        end
    end
end
{% endhighlight %}

Unfortunately, the corresponding unit test **unit_test.jl** for the `merge_sort!` function fails:


{% highlight julia %}
# import code
include("./merge_sort.jl")

# test merge sort code
using Test

@testset "Test Merge Sort" begin
    for k = 1:3
        a = 100. * randn(10000) .- 500.
        merge_sort!(a)
        @test issorted(a)
    end
end
{% endhighlight %}

Let's take a look at the previous commits to see if we can figure out which commit might cause the problem.

```
MergeSortProject (master)🚀 $ git log --pretty=oneline
```

```
cb67f4062dfc16f35cbd91a53582c1655705818d (HEAD -> master) Change deepcopy back to similar
1327949703a06918d62198ea34b5a47d36f3a90f Use deepcopy instead of similar
2c84ffd5916809ba1ca922b749ecede7c5f0139c Remove loop annotation
2659d69af012edefc2c444f9ae4556711f7dcb65 [*] Swap first and last element
2783ba6cc0f5cd5db33b5879eb54547743e6fd8c Make if-statement one-liner
86f3851d0ff4838567dbcb265ed07c6b8ef143fd Add function documentation
0d1c8b0fa1aed40ee7fc2f60f2f1d006b849f59e Add unit test
c5b68639402ff014264faf82e7b61beaf446a1ae Initial commit
```
It
<span class="sidenote">
<input
aria-label="Show sidenote"
type="checkbox"
id="sidenote__checkbox--2"
class="sidenote__checkbox">
<label
tabindex="0"
title="{{ include.content }}"
aria-describedby="sidenote-2"
for="sidenote__checkbox--2"
class="sidenote__button sidenote__button--number-2
">does not seem obvious.</label>
<small
id="sidenote-2"
class="sidenote__content sidenote__content--number-2">
<span class="sidenote__content-parenthesis
"> (sidenote: </span>
Okay, it is fairly obvious. The problematic commit is the one marked `[*]`.
<span class="sidenote__content-parenthesis">)</span>
</small>
</span>I only remember that the code in the first commit `c5b6863` still worked and in the current commit `HEAD` it does not.

## Manual git bisect
We use `git bisect` to start the search for the broken commit: 

```
MergeSortProject (master)🚀 $ git bisect start
```
Next, we give Git the boundary of the search range, i.e. one commit that is bad (the current one):

```
MergeSortProject (master)🚀 $ git bisect bad HEAD
```

and one commit that worked:

```
MergeSortProject (master)🚀 $ git bisect good c5b6863
```

```
Bisecting: 3 revisions left to test after this (roughly 2 steps)
[2783ba6cc0f5cd5db33b5879eb54547743e6fd8c] Make if-statement one-liner
```
Git will now go through the range, checkout different commits and ask us if the commit is good or bad. As you can see it first checks out the commit in the middle of the interval `2783ba6`. I run `unit_test.jl` and the code still works. I tell Git this by using:

```
MergeSortProject ((no branch, bisect started on master))🚀 $ git bisect good
```

```
Bisecting: 1 revision left to test after this (roughly 1 step)
[2c84ffd5916809ba1ca922b749ecede7c5f0139c] Remove loop annotation
```
Next, Git checked out the middle commit `2c84ff` in the remaining interval. After running the unit test I realize that the test fails, so I tell Git:

```
MergeSortProject ((no branch, bisect started on master))🚀 $ git bisect bad
```

```
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[2659d69af012edefc2c444f9ae4556711f7dcb65] [*] Swap first and last element
```
It checks out another commit. I try the unit tests and they fail again, so I again use:

```
MergeSortProject ((no branch, bisect started on master))🚀 $ git bisect bad
```

```
2659d69af012edefc2c444f9ae4556711f7dcb65 is the first bad commit
commit 2659d69af012edefc2c444f9ae4556711f7dcb65
Author: Michael 
Date:   Sat Jan 9 14:46:32 2021 +0000

    [*] Swap first and last element

 merge_sort.jl | 3 +++
 1 file changed, 3 insertions(+)
```
The broken commit has been determined. Looking closer at what I did in that commit,

```
MergeSortProject ((no branch, bisect started on master))🚀 $ git show
```

```
diff --git a/merge_sort.jl b/merge_sort.jl
index 0fc13fd..1b41564 100644
--- a/merge_sort.jl
+++ b/merge_sort.jl
@@ -4,6 +4,9 @@ function merge_sort!(a::AbstractArray{T}) where {T <: Real}

     helper = similar(a)
     _merge_sort!(a, 1, length(a), helper)
+    tmp = a[end]
+    a[end] = a[1]
+    a[1] = tmp
 end
```

I can see that I accidentally swapped the first and last element of the array after sorting it. If at any point during the search process I had misclassified a commit, I could have stopped the search with `git bisect reset`.

## Automatic git bisect

We can make our lifes easier by providing a script that tests each commit during the bisection process and tells Git if the commit is good or bad.
The syntax for this is `git bisect run my_script arguments`.
In this example we can simply use our unit test as the script:

```
MergeSortProject (master)🚀 $ git bisect start
MergeSortProject (master)🚀 $ git bisect bad HEAD
MergeSortProject (master)🚀 $ git bisect good c5b6863
MergeSortProject (master)🚀 $ git bisect run julia unit_test.jl
```

{% highlight bash %}
running julia unit_test.jl
Test Summary: | Pass  Total
Merge Sort    |    3      3
Bisecting: 1 revision left to test after this (roughly 1 step)
[2c84ffd5916809ba1ca922b749ecede7c5f0139c] Remove loop annotation
running julia unit_test.jl
Test Merge Sort: Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11
  Expression: issorted(a)
Stacktrace:
 [1] macro expansion at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11 [inlined]
 [2] macro expansion at /Users/julia/buildbot/worker/package_macos64/build/usr/share/julia/stdlib/v1.5/Test/src/Test.jl:1115 [inlined]
 [3] top-level scope at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:8
Test Merge Sort: Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11
  Expression: issorted(a)
Stacktrace:
 [1] macro expansion at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11 [inlined]
 [2] macro expansion at /Users/julia/buildbot/worker/package_macos64/build/usr/share/julia/stdlib/v1.5/Test/src/Test.jl:1115 [inlined]
 [3] top-level scope at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:8
Test Merge Sort: Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11
  Expression: issorted(a)
Stacktrace:
 [1] macro expansion at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11 [inlined]
 [2] macro expansion at /Users/julia/buildbot/worker/package_macos64/build/usr/share/julia/stdlib/v1.5/Test/src/Test.jl:1115 [inlined]
 [3] top-level scope at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:8
Test Summary:   | Fail  Total
Test Merge Sort |    3      3
ERROR: LoadError: Some tests did not pass: 0 passed, 3 failed, 0 errored, 0 broken.
in expression starting at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:7
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[2659d69af012edefc2c444f9ae4556711f7dcb65] [*] Swap first and last element
running julia unit_test.jl
Merge Sort: Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11
  Expression: issorted(a)
Stacktrace:
 [1] macro expansion at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11 [inlined]
 [2] macro expansion at /Users/julia/buildbot/worker/package_macos64/build/usr/share/julia/stdlib/v1.5/Test/src/Test.jl:1115 [inlined]
 [3] top-level scope at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:8
Merge Sort: Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11
  Expression: issorted(a)
Stacktrace:
 [1] macro expansion at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11 [inlined]
 [2] macro expansion at /Users/julia/buildbot/worker/package_macos64/build/usr/share/julia/stdlib/v1.5/Test/src/Test.jl:1115 [inlined]
 [3] top-level scope at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:8
Merge Sort: Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11
  Expression: issorted(a)
Stacktrace:
 [1] macro expansion at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:11 [inlined]
 [2] macro expansion at /Users/julia/buildbot/worker/package_macos64/build/usr/share/julia/stdlib/v1.5/Test/src/Test.jl:1115 [inlined]
 [3] top-level scope at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:8
Test Summary: | Fail  Total
Merge Sort    |    3      3
ERROR: LoadError: Some tests did not pass: 0 passed, 3 failed, 0 errored, 0 broken.
in expression starting at /Users/Micha/Dropbox/Research/GitBisectExp/unit_test.jl:7
2659d69af012edefc2c444f9ae4556711f7dcb65 is the first bad commit
commit 2659d69af012edefc2c444f9ae4556711f7dcb65
Author: Michael 
Date:   Sat Jan 9 14:46:32 2021 +0000

    [*] Swap first and last element

 merge_sort.jl | 3 +++
 1 file changed, 3 insertions(+)
bisect run success
{% endhighlight %}
The output shows that the unit test passes on the first suggested commit and then fails twice, which correctly identifies the broken commit `[*] Swap first and last element` as before.

In this example the unit test file `unit_test.jl` is also tracked in the repository. Only use a tracked script if you are sure that the script itself does not introduce the problem or was changed severely by one of the commits.

## Finding performance digressions
Above workflow is not only useful to find bugs.
It can also be used to find commits that reduce the performance of our code.

After finding the bug in the previous section, I committed a fix (`a53c36`). I also benchmarked my code and found that it takes `108.5μs` to sort an array of length `5000`. After working a bit longer on the code I realize that the code has noticeably slowed down. Sorting the same array now takes `122.2ms`. All the unit tests still pass, so one of my changes must cause the slowdown.
Let's look again at the commit log:

```
MergeSortProject (master)🚀 $ git log --pretty=oneline
```

```
4fdadbc9304bea5e9db90267fbc36ce12b7fffca (HEAD -> master) Add comment on function call
4c47b70d0cc90630460bc5f744338767972d475f [**] Don't reuse helper
be3c1deb31b1f1595338f09f495dae72937abf7e Rename `merge!` to `_merge!`
1d96d60a3927f7eaaca13a282137bca87b785818 Make if-stament oneliner
a53c361b69efc45780ac90cacf8d51ab00a0f0fc Fix bug found using git bisect
cb67f4062dfc16f35cbd91a53582c1655705818d Change deepcopy back to similar
1327949703a06918d62198ea34b5a47d36f3a90f Use deepcopy instead of similar
2c84ffd5916809ba1ca922b749ecede7c5f0139c Remove loop annotation
2659d69af012edefc2c444f9ae4556711f7dcb65 [*] Swap first and last element
2783ba6cc0f5cd5db33b5879eb54547743e6fd8c Make if-statement one-liner
86f3851d0ff4838567dbcb265ed07c6b8ef143fd Add function documentation
0d1c8b0fa1aed40ee7fc2f60f2f1d006b849f59e Add unit test
c5b68639402ff014264faf82e7b61beaf446a1ae Initial commit
```

I can use `git bisect` to find the commit that is causing the performance issues by writing a script that benchmarks the sorting algorithm and compares it to a target time plus some tolerance. If the test passes the commit is classified as good and as bad otherwise. We can use the following script:

**time_code.jl**

{% highlight julia %}
using BenchmarkTools, Test, Random
include("./merge_sort.jl")

# run benchmark
rng = Random.MersenneTwister(1)
a = 100. * randn(rng, 5000) .- 500.
min_time = @belapsed merge_sort!($a)
tol = 1.1
target_time = parse(Float64, ARGS[1])

println("Benchmark time: $(min_time) vs. target time (+tol): $(tol * target_time)")
@test min_time < tol * target_time
{% endhighlight %}

Next, we run the bisection and pass our target time (`108.5μs`) to the script.

```
MergeSortProject (master)🚀 $ git bisect start
MergeSortProject (master)🚀 $ git bisect bad HEAD
MergeSortProject (master)🚀 $ git bisect good a53c361
MergeSortProject (master)🚀 $ git bisect run julia time_code.jl 0.000108514
```

```
MergeSortProject (master)🚀 $ git bisect run julia time_code.jl  0.000108514
running julia time_code.jl 0.000108514
Benchmark time: 0.000105667 vs. target time (+tol): 0.00011936540000000001
Bisecting: 0 revisions left to test after this (roughly 0 steps)
[4c47b70d0cc90630460bc5f744338767972d475f] [**] Don't reuse helper
running julia time_code.jl 0.000108514
Benchmark time: 0.1221385 vs. target time (+tol): 0.00011936540000000001
Test Failed at /Users/Micha/Dropbox/Research/GitBisectExp/time_code.jl:12
  Expression: min_time < tol * target_time
   Evaluated: 0.1221385 < 0.00011936540000000001
ERROR: LoadError: There was an error during testing
in expression starting at /Users/Micha/Dropbox/Research/GitBisectExp/time_code.jl:12
4c47b70d0cc90630460bc5f744338767972d475f is the first bad commit
commit 4c47b70d0cc90630460bc5f744338767972d475f
Author: Michael Garstka <michael@garstka.org>
Date:   Tue Jan 12 22:40:24 2021 +0000

    [**] Don't reuse helper

 merge_sort.jl | 19 +++++++++----------
 1 file changed, 9 insertions(+), 10 deletions(-)
bisect run success
```
We can take a closer look at the identified commit:

```
MergeSortProject ((no branch, bisect started on master))🚀 $ git show 4c47b70d0
```

```
diff --git a/merge_sort.jl b/merge_sort.jl
index e6f4439..4ae0234 100644
--- a/merge_sort.jl
+++ b/merge_sort.jl
@@ -2,13 +2,13 @@
 function merge_sort!(arr::AbstractArray{T}) where {T <: Real}
     length(arr) <= 1 && return arr

-    helper = similar(arr)
-    _merge_sort!(arr, 1, length(arr), helper)
+
+    _merge_sort!(arr, 1, length(arr))

 end

 "Split array and half and recursively call this function."
-function _merge_sort!(arr::AbstractArray{T}, l::Int64, r::Int64, helper::AbstractArray{T}) where {T <: Real}
+function _merge_sort!(arr::AbstractArray{T}, l::Int64, r::Int64) where {T <: Real}
     # single element: already sorted
     r - l <= 0 && return nothing

@@ -16,19 +16,18 @@ function _merge_sort!(arr::AbstractArray{T}, l::Int64, r::Int64, helper::Abstrac
     # split list and sort individually
     m = div(r - l + 1, 2) + l - 1

-    _merge_sort!(arr, l, m, helper)
-    _merge_sort!(arr, m + 1, r, helper)
+    _merge_sort!(arr, l, m)
+    _merge_sort!(arr, m + 1, r)

     # merge them back together
-    _merge!(arr, l, m, r, helper)
+    _merge!(arr, l, m, r)

 end

 "Merge two sorted arrays in sorted fashion."
-function _merge!(arr::AbstractArray{T}, l::Int64, m::Int64, r::Int64, helper::AbstractArray{T}) where {T <: Real}
-    for k = l:r
-        helper[k] = arr[k]
-    end
+function _merge!(arr::AbstractArray{T}, l::Int64, m::Int64, r::Int64) where {T <: Real}
+    helper = deepcopy(arr)
+
     helper_l = l
     helper_r = m + 1
     current = l
```
And indeed, that commit introduced repeated allocation of the `helper` array in `_merge!` which causes the slowdown of the algorithm.

