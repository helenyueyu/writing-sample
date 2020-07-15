# Week 3, Day 3

* [Curriculum][curriculum-link]
* [Recursion Exercises][recursion-exercises]
* [Project (Word Chains)][project-link]
* [Link to Make Change explanation][change-link]


## Deep Dup 


This is **NOT** the same as `Array#flatten`, so students who are used to the code for that, explain that this is different and we are not trying to flatten a nested array, but really create a true copy. 

Why do we even write `.deep_dup`? Why not just use Ruby's built-in `.dup` method? The reason is because `.dup` is a shallow clone. Ask them to try this out: 

```rb

x = [1, 2, 3, 4, 5]
y = x.dup 

y[0] = "hello"

x # [1, 2, 3, 4, 5]
y # ["hello", 2, 3, 4, 5] (expected behavior - it SEEMS like y is a true copy of x) 

```
**However**, in pry, if we type out: 

```rb
x = [1, [2], 3, 4, 5]
y = x.dup 

y[1] << "hello"

x # [1, [2, "hello"], 3, 4, 5]  
y # [1, [2, "hello"], 3, 4, 5]  Oh no! We meant to only change `y`, how come `x` is changed too?
```

Notice that even though `y` is supposed to be a true copy of `x`, that it isn't. Reminder: when we monkey-patch on the Array class, **self** is referring to the instance of the array that `deep_dup` is called on. 

The code: 

```rb
class Array 
    def deep_dup 
        res = [] 
        self.each do |el|
            if el.is_a?(Array)
                res << el.deep_dup 
            else
                res << el 
            end
        end
        res 
    end
end
```


## BSearch

General implementation: 

* We want to return `nil` if the array is empty. 
* Else, we will recursively search the left and right halves. 

The code: 

```rb
def bsearch(array, target)
    return nil if array.empty? 
    mid = array.length / 2
    l = array[0..mid-1]
    r = array[mid+1..-1] 
    if target < array[mid]
        bsearch(l, target) 
    elsif target > array[mid]
        bsearch(r, target) ? 1 + mid + bsearch(r, target) : nil 
    else 
        return mid 
    end
end

```

A few tricky things to note: 

1. Students might instantiate the left half of the array to be `l = array[0..mid]`, where the `mid` is **included**. When this happens, it is possible for them to get a `stack overflow` error, because when we go down to to an array of size 1, we get that `mid = 0`, so when we key into `array[0..0]`, we get the array back itself. So a 1-element array will never reduce down to the base case (an empty array). Walk through a simple array to show them that this is the case. 

2. It is a little bit tricky to understand why we need to do `1 + mid + bsearch(rightHalf, target)`, instead of just `bsearch(rightHalf, target)`. This is because while the element is correctly indexed in the **right half subarray**, it is wrongly indexed in the **main array**. We need to bubble up the fact that the element was found after the midpoint, and our right subarray's **0th element** is the main array's **n + 1'th element**. 

3. Along point (2), with with respect to the right half of the array students might get the `nil cannot be coerced into an Integer` error. This is because, if the right side returns nil, then when we attempt to add `nil` to `mid + 1`, are adding an integer value with a `nil` value, and Ruby isn't as forgiving as Javascript when it comes to type coercion! 


Note that the reason why bSearch is `O(logn)` and other sorting algorithms like mergeSort `O(nlogn)`? Because the array is **sorted**. 


## Merge Sort 

Some students might be confused about what **merge sort**. Show them that the goal of **merge sort** is to essentially break the array in halves, then break each left and right halve into halves, and so on and so forth until we reach our base case (arrays of length 0 or 1). Arrays of length 0 or 1 are trivially sorted. Then we will recursively merge the **sorted** left and right halves until larger **sorted** halves, until the entire array is sorted. 

Tricky points to note: 

1. Sometimes students will fail implementing their `merge_sort` method, not because their logic on the `merge_sort` method is wrong, but because their helper method `merge` is incorrect. So ask them to test their `merge` method on a bunch of different arrays. 

```
merge([], [1, 2, 3, 4])  # their merge should work on empty arrays!
merge([1, 2, 3], [-10, -9]) # their merge should work on this case!
merge([1, 3, 5], [2, 4, 6]) # should work on this case too!
```

2. Ask them to explain back the abstract logic of **merge sort** back to you! Or if one partner understands it better, ask them to explain to their partner. 

Note: Time complexity of **merge sort** is **O(n log n)**. Why? Because it takes us **O(log n)** time to split the array into left and right halves recursively, and another **O(log n)** to recursively merge back the left and right halves. For each step of merging back the left and right halves, we have to iterate through each element in both sorted subarrays, which is an **O(n)** procedure. So the time complexity is **O(log n) + n O(log n)** which simplifies to **O(n log n)**. 

3. Implementing the **helper merge method** is tricky. Ask them intuitively, if two arrays were sorted, how would you combine them? Their gut reaction might be to check the smallest elements from both arrays. But since the arrays are sorted, wouldn't the smallest element be at index `0`? 
  * Ask them what is the terminating condition for the while loop (how do we know when to stop comparing elements from each array? 
  * Finally, ask them what we should return. Must it be true that both arrays **HAVE** to be empty when the while loop terminates? Or is it possible for one of the arrays to still have remaining elements? And if that array has remaining elements, **what is the relation of those remaining elements to the elements we shoveled into the results array**? 

Implementation: 

```rb
def merge_sort(arr)
    return arr if arr.length <= 1 
    mid = arr.length / 2 
    l = arr[0..mid-1]
    r = arr[mid..-1]
    return merge(merge_sort(l), merge_sort(r))
end

def merge(arr1, arr2)
    res = []
    while arr1.length > 0 && arr2.length > 0
        if arr1[0] <= arr2[0]
            res.push(arr1.shift)
        else
            res.push(arr2.shift)
        end
    end
    return res + arr1 + arr2 
end
```

## Subsets 

Seeing the pattern is tricky, especially from the very base case of an empty set. Ask them, **given that they've already hypothetically solved subsets([1, 2])**, how would they compute the answer for **subsets([1, 3])**? In other words, how do we go from: 

`[[], [1], [2], [1, 2]]` 

to: 

`[[], [1], [2], [1, 2], [3], [1, 3], [2, 3], [1, 2, 3]]`

The key observation to make is that out of the 8 elements in **subsets([1, 3])**, **4 of them are identical** to the elements returned from the previous recursive call **subsets([1, 2])**. What about the **4 new elements**? If we take a look, these new elements are **basically each previous element, but with the newest addition (3) shoveled inside!**. And that is true for the previous cases as well. 

```rb
subsets([]) # [[]]
subsets([1]) #[[], [1]]
subsets([1, 2]) #[[], [1], [2], [1, 2]]
subsets([1, 2, 3]) #[[], [1], [2], [1, 2], [3], [1, 3], [2, 3], [1, 2, 3]]
```


Implementation: 

```rb
def subsets(arr)
    res = [] 
    return [[]] if arr.empty? 
    r = subsets(arr[0..-2])
    r.each do |el|
        res << el 
        res << el + [arr.last]
    end
    res 
end
```

## Permutations

Seeing the pattern is tricky for this one as well, but for each new element, we have iterate through each previous subarray. For each previous subarray, we can insert the new element in `n+1` new places. 

So for example: 

```rb
[[1, 2], [2, 1]]
```

For `[1, 2]`, we can insert the 3 in 3 different places, before the `1`, between the `1` and `2`, and after the `2`. So the three possibilities are: 

* `[3, 1, 2]`
* `[1, 3, 2]`
* `[1, 2, 3]`

Similarly, we can do the exact same thing for the subarray `[2, 1]`. 



Implementation: 

```rb
def permutations(arr)
    res = [] 
    return [[]] if arr.empty? 
    return [[1]] if arr.length == 1
    r = permutations(arr[0..-2])
    r.each do |el|
        (0..el.length).each do |idx|
            res << el[0...idx] + [arr.last] + el[idx..-1]
        end
    end
    res 
end
```


## Make Greedy Change 

We basically keep trying the largest denomination, until we can no longer. Then, we remove the largest denomination from our set of coins, and solve the remainder with the same coins, only with the largest coin removed. 

Note: The implementation below **does not work** if there doesn't exist a combination that adds up to the sum. 

Implementation: 
```rb

def make_greedy_change(sum, coins)
    res = [] 
    return res if sum == 0; 

    while sum >= coins[0]
        res << coins[0]
        sum -= coins[0]
    end

    mcg = make_greedy_change(sum, coins[1..-1])
    res + mcg 
end

p make_greedy_change(16, [10, 7])   # will not work
```


Note: This is the edited version that **works** if there doesn't exist a combination that adds up to the sum (returns `nil`). 

Implementation: 

```rb
def make_greedy_change(sum, coins)
    res = [] 
    return res if sum == 0; 

    return -1 if coins.empty? 

    while sum >= coins[0]
        res << coins[0]
        sum -= coins[0]
    end

    mcg = make_greedy_change(sum, coins[1..-1])
    mcg.is_a?(Integer) ? -1 : res + mcg 
end

p make_greedy_change(24, [10, 7, 1])
p make_greedy_change(16, [10, 7])

```

## Common Issues

* `Array#concat` vs. `Array#+` vs. `Array#<<` people shoveling arrays into arrays and then calling `Array#flatten`
* How the splitting and merging arrays works in merge_sort
* Base cases — having two base cases when you only need one
  + What to return in the base case (array vs int)
* Destroying `bsearch`'s O(log(n)) speed by sorting or using `Array#includes`?
* How to get the index in the right side array in `bsearch` (esp. remember to add 1)
* Understanding why `deep_dup` is necessary
* Wondering what does "don't generate every permutation" mean? (for `make_change`)
* Misinterpreting the inductive step for subarrays
* Not writing `make_greedy_change` recursively
* `+` when they should be doing `+=`

## Code Review Topics

* Check over their syntax and understanding of recursion

## Change Ideas

* Link to [this animation][merge] for `merge_sort`
* More explicit note not to brute force `make_change`
* Clarify what "range" means (does it include start and end of array?) in range problem

[merge]: http://www.algomation.com/player?algorithm=551321f6e1b6fa0300aae4d0
[curriculum-link]: https://github.com/appacademy/curriculum/tree/master/ruby#w1d3
[recursion-exercises]: https://github.com/appacademy/curriculum/tree/master/ruby/projects/recursion
[project-link]: https://github.com/appacademy/curriculum/tree/master/ruby/projects/word_chains
[change-link]: https://vimeo.com/groups/appacademy/videos/91207646
