---
layout: post
title: "The Infamous Interview Question Kth Largest"
description: "Kth Largest or Kth Smallest is a very commonly asked problem during interviews. Here I'll explore all the possible answers including Priority Queue (Min Heap/Max Heap) and Quick Sort."
---

The question is simple: how do you find the kth largest element in an unsorted array? They love asking it, and you sure as hell should know it. Here I'll just list the possible solutions:

* Just sort it, easy peasy `O(n*log(n))`.
* Min heap, `O(k+(n-k)*log(k))`.
* Max heap, `O(n+k*log(n))`.
* Some gimmick quick sort, `O(n^2)` worst, `O(n)` average.

For min heap:
- build a min heap using k elements.
- for the rest elements:
  - if it's smaller than heap.min ignore it
  - otherwise pop the heap and put the new one in.
- in the end heap.min is the kth largest.

For max heap:
- build a max heap using all n elements.
- pop k elements and the last pop is the kth largest.

Here are some questions you should be able to answer with ease:

* Q: What about an almost sorted array?

  A: use insertion sort.
* Q: What's the difference of min heap and max heap.

  A: use min heap if you have a data stream.
* Q: How do we guarantee that we always get the average time using quick sort.

  A: shuffle the array first.

Now since this problem is so simple, interviewer often wants you to implement the binary heap, it is crucial that you know how to construct the heap in `O(n)` time. Since ruby doesn't have build-in heap, it's good practice to write one yourself, (watch [this](https://www.coursera.org/lecture/algorithms-part1/binary-heaps-Uzwy6)):

```ruby
class MinHeap
  def initialize(arr=[])
    @arr = [nil]
    @arr = @arr.concat(arr)
    (arr.size/2).downto(1).each { |i| sink(i) }
  end

  def insert(x)
    @arr << x
    swim(size)
  end

  def pop
    x = @arr[1]
    @arr[1] = @arr[-1]
    @arr.pop
    sink(1)
    x
  end

  def size
    @arr.size - 1
  end

  def min
    @arr[1]
  end

  private

  def swim(i)
    while i > 1 && @arr[i/2] > @arr[i]
      swap(i, i/2)
      i = i/2
    end
  end

  def sink(i)
    while i*2 <= size
      j = i*2
      j += 1 if j < size && @arr[j+1] < @arr[j]
      break if @arr[j] > @arr[i]
      swap(i, j)
      i = j
    end
  end

  def swap(i, j)
    @arr[i], @arr[j] = @arr[j], @arr[i]
  end
end
```

Then to solve the kth largest using `MinHeap`:

```ruby
def kth_largest(arr, k)
  h = MinHeap.new(arr[0..(k-1)])
  arr[k..].each do |x|
    if x > h.min
      h.pop
      h.insert(x)
    end
  end
  h.min
end
```

Max heap is basically the same, I'll leave it to you.

The best answer is the quick sort method, and it doesn't require additional space. However I can only hope the interviewer doesn't want you to analyze the time complexity because I myself have trouble understanding it. Anyway, the idea is simple, we simply abandon 'half' of the input on every recursion and stop when we find it. The key is the `three_way_partition` method, it handles duplicates and it's easy to implement, for example:

```
k = 3, nums = [5, 4, 1, 1, 4, 3, 5, 7]
three way partition, v = 5

[5, 4, 1, 1, 4, 3, 5, 7]
 |  |                 |
 lt i                 gt
4<5, swap(lt,i) then lt++, i++

[4, 5, 1, 1, 4, 3, 5, 7]
    |  |              |
    lt i              gt
1<5, swap(lt,i) then lt++, i++

[4, 1, 5, 1, 4, 3, 5, 7]
       |  |           |
       lt i           gt
1<5, swap(lt,i) then lt++, i++

[4, 1, 1, 5, 4, 3, 5, 7]
	  |  |        |
	  lt i        gt
4<5, swap(lt,i) then lt++, i++

[4, 1, 1, 4, 5, 3, 5, 7]
	     |  |     |
	     lt i     gt
3<5, swap(lt,i) then lt++, i++

[4, 1, 1, 4, 3, 5, 5, 7]
		|  |  |
		lt i  gt
5==5, i++

[4, 1, 1, 4, 3, 5, 5, 7]
		|    /\
		lt  i  gt
7>5, swap(gt,i), then gt--

[4, 1, 1, 4, 3, 5, 5, 7]
		|  |  |
		lt gt i
i > gt, break, return [5, 6]
```

This is the first iteration of three way partition, we found that since k==3, length-k==5, it's between [5,6] so we found the kth largest item. if k>3, we just repeat above step within [0,4], aka the left side else we search the right side.

```ruby
def find_kth_largest(nums, k)
  nums.shuffle!
  helper(nums, 0, nums.length-1, k)
end

def helper(nums, lo, hi, k)
  left, right = three_way_partition(nums, lo, hi)
  pos = nums.length-k
  if pos < left
    helper(nums, lo, left-1, k)
  elsif pos > right
    helper(nums, right+1, hi, k)
  else
    nums[left]
  end
end

def three_way_partition(nums, lo, hi)
  i, lt, gt, v = lo+1, lo, hi, nums[lo]
  while i <= gt
    if nums[i] > v
      swap(nums, i, gt)
      gt -= 1
    elsif nums[i] < v
      swap(nums, i, lt)
      i += 1
      lt += 1
    else
      i += 1
    end
  end
  [lt, gt]
end

def swap(nums, i, j)
  nums[i], nums[j] = nums[j], nums[i]
end
```

There are some other gimmick questions, like what if we can't load all the numbers into memory at once. Min heap is a good answer since it only has `O(k)` space requirement. Anyway knowing all this should help you regardless.