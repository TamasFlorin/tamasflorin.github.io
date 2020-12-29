---
title: Advent of Code 2020 - Day 1
category: blog
layout: post
description: Advent of Code 2020 Day 1
tag:
- advent-of-code-2020
- python
headerImage: false
---

Even though[ Advent of Code 2020 ](https://adventofcode.com/2020) is over, I thought it would be a good opportunity for me to get back into writing Python code and create some blog posts along the way.

My idea for these posts is to explore multiple solutions(*read naive and optimal solutions*) and compare their runtime and space complexities. 

All of the code is going to be written in python3, so if you want to follow along make sure that you have it installed locally.

Every Advent of Code problem consists of two parts, so we're going to split each post into two sections: In the first section, we're going to be talking about the possible solutions for the first part, while in the second section we're going to focus on the second part. If you already solved one of the parts, you can just jump to the one that is of interest to you.

Today we're going to explore the **Day 1: Report Repair** problem.
# Part 1
> Before you leave, the Elves in accounting just need you to fix your expense report (your puzzle input); apparently, something isn't quite adding up.
> Specifically, they need you to find the two entries that sum to 2020 and then multiply those two numbers together.

In shorter terms, given a list of integers, we need to find the two entries that sum to **2020** and return their product.\\
For example, given the list **`[1721, 979, 366, 299, 675, 1456]`**, the two numbers that match the given condition are  **`1721`** and **`299`** so we need to return **`1721 * 299 = 514579`**.

Let's start off by creating the function for reading the input from a file:
```python
from typing import List
import sys

def read_input(filepath: str) -> List[int]:
    with open(filepath) as f:
        numbers = list(map(lambda x: int(x.strip()), f.readlines()))
        return numbers

if __name__ == '__main__':
    numbers = read_input(sys.argv[1])
    print(f'The numbers are: {numbers}')

```

Now if we run our code, we get the integer list as expected:
```bash
$ python3 day_1.py sample_input.txt

The numbers are: [1721, 979, 366, 299, 675, 1456]
```

## Naive Solution
The first solution that comes to mind is to just iterate over all pairs of integers and check if their sum is equal to **2020**:

```python
from typing import List
from typing import Tuple
from typing import Optional

def two_sum(numbers: List[int], target: int) -> Optional[Tuple[int, int]]:
    for i in range(len(numbers)):
        for j in range(i + 1, len(numbers)):
            s = numbers[i] + numbers[j]
            if s == target:
                return numbers[i], numbers[j]

    return None

if __name__ == '__main__':
    numbers = read_input(sys.argv[1])
    first, second = two_sum(numbers, target=2020)
    print(first, second, first * second)
```

Looks good right? Let's run it on the sample input:
```bash
$ python3 day_1.py sample_input.txt 

1721 299 514579
```
Yay, we got the correct output from the first try! This wasn't that hard, was it?

At this point you might have some questions, so let's address some of them.

> Why is the second loop starting from **i + 1**?

There are two main reasons for this:
- We do not want to reuse the same element twice. For example, if our list contained **`[1010, 1721, 299]`** and **`j`** would start from **`i`** when **`i = 0`**, we would use **`1010`** twice, thus returning an invalid solution.
- If we already visited the subset **`[0, i]`**, we no longer need to visit it in the second loop since it would just generate a duplicate solution with the pairs in reverse order.

> Why did you return an Optional?

This is not really required for our problem since we are always going to have a pair that satisfies the given condition but I wanted to follow a more generic approach that does not depend on this assumption.

> Isn't there a more *pythonic* approach to this?

Good catch, we can write this in a much nicer way by making use of python's **itertools.combinations**.  The **combinations** function can be used to return al the successive *2-length* combinations from a given iterable.
First, let's bring the *combinations* functional method into our namespace:
```python
from itertools import combinations
```

Now we can rewrite the **two_sum** method as follows:
```python
def two_sum(numbers: List[int], target: int) -> Optional[Tuple[int, int]]:
    return next(filter(lambda p: sum(p) == target,
                       combinations(numbers, 2)), None)
```

If we run the code on the sample input once again, we still get the expected output:
```bash
$ python3 day_1.py sample_input.txt 

1721 299 514579
```

The **runtime complexity** of this approach is **O(N^2)** where N is equal to the number of integer values that we were given as input. The **space complexity** is **O(1)** since we do not store any auxiliary data structures and we just return a pair containing the matching elements.

## Optimal Solution
How could we improve our naive solution? A good place to start would be the removal of the inner loop but how could we do that? 
Let's first look at why we need the inner loop: for a given element **a** we need to find an element **b** such at **a + b = 2020**, from this we can deduce that **b = 2020 - a**.
Since we know what the value of the element should be, we just need a data structure that can help us check if that value is present in our integer list in **O(1)** time.
A perfect fit for this is a **hash map**(or **set** in python) since it has an amortized runtime complexity of **O(1)** for both adding and checking for the existence of an element.

This problem is commonly known as [**Two Sum**](https://leetcode.com/problems/two-sum/)(thus the function name) and has an O(n) solution since for any given **k**, the **ksum** problem can be solved in **O(N^(k / 2))**.

Thus, our function can be rewritten as follows:
```python
def two_sum(numbers: List[int], target: int) -> Optional[Tuple[int, int]]:
    available_numbers = set()
    for number in numbers:
        remaining = target - number
        if remaining in available_numbers:
            return [remaining, number]
        available_numbers.add(number)

    return None
```

The **runtime complexity** of this approach is **O(N)** due to the usage of a **set** for the task of checking if the remaining target difference exists. The space complexity is also is **O(N)** since we are using a **set** to store the available number at each step.

# Part 2
> The Elves in accounting are thankful for your help; one of them even offers you a starfish coin they had left over from a past vacation. 
> They offer you a second one if you can find three numbers in your  expense report that meet the same criteria.

Compared to **Part 1** we now need to find three numbers such that **a  + b + c = 2020**. For example, given the list **`[1721, 979, 366, 299, 675, 1456]`**, the three numbers that match the given condition are  **`979`**, **`366`** and **`675`** so we need to return **`979 * 366 * 675 = 241861950`**.

## Naive Solution
The naive solution is almost the same as the solution for **Part 1**, the only difference is that we need to change the length of the tuples that need to be generated by **itertools.combinations**.
To make this more generic, let's add a paramter to our initial function that allows us to control the tuple size and let's rename the function to **"k_sum"** to better express it's functionality.

```python
def k_sum(numbers: List[int], target: int, tuple_size: int) -> Optional[Tuple[int, ...]]:
    assert tuple_size > 0 and tuple_size <= len(numbers)
    return next(filter(lambda p: sum(p) == target,
                       combinations(numbers, tuple_size)), None)

if __name__ == '__main__':
    numbers = read_input(sys.argv[1])
    first, second, third = k_sum(numbers, target=2020, tuple_size=3)
    print(first, second, third, first * second * third)
```

Running the above solution prodices the expected output:
```bash
$ python3 day_1.py sample_input.txt 

979 366 675 241861950
```

The **runtime complexity** of this approach is **O(N^3)** where N is equal to the number of integer values that we were given as input. The **space complexity** is **O(1)** for the same reasons as for Part 1.
## Optimal Solution
The optimal solution for this part is based on the observation that for **a + b + c = 2020** we can use our optimal **two_sum** function to find the values of **b** and **c**, since **b + c = 2020 - a**.

```python
def three_sum(numbers: List[int], target: int) -> Optional[Tuple[int, int, int]]:
    for i, number in enumerate(numbers):
        remaining = target - number
        pair = two_sum(numbers[i + 1:], remaining)
        if pair:
            return number, (*pair)

    return None
```

The **runtime complexity** of this approach is **O(N^2)** where N is equal to the number of integer values that we were given as input.
The **space complexity** is **O(N)** since the **two_sum** method stores the available numbers in a **set**.
