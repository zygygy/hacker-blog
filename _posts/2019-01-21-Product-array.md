---
title: "곱하기 배열 퍼즐 - Product array puzzle"
published: true
categories: [problem]
tags: [array, puzzle, easy, range]
---

# 문제

Given an array of integers, return a new array such that each element at index i of the new array is the product of all the numbers in the original array except the one at i.

주어진 정수 배열에서, 원래 배열의 인덱스 i의 값을 제외한 모든 숫자들을 곱한 값이 새로운 배열의 인덱스 i번째 값이 되도록 하는 새로운 배열을 반환하라.
> 이게 무슨 말이냐. --;

For example, if our input was [1, 2, 3, 4, 5], the expected output would be [120, 60, 40, 30, 24]. If our input was [3, 2, 1], the expected output would be [2, 3, 6].

예를 들어, [1, 2, 3, 4, 5]가 입력으로 주어지면, 출력은 [120, 60, 40 ,30, 24], 입력이 [3, 2, 1] 이면, 출력은 [2, 3, 6] 이 된다.

Follow-up: what if you can't use division?
나누기 연산을 하지 말고 풀어야 한다.

추가 조건: 공간 복잡도 O(1) 으로 풀어라.

* 출처 <https://www.geeksforgeeks.org/a-product-array-puzzle>{:target="_blank"}

# 풀이

이제 아예 대놓고 퍼즐이란다.

공간 복잡도 O(1) 즉 문제 배열안에서 스캔하면서 동시에 임시 계산값을 저장하는 트릭을 사용하면 메모리 사용을 줄일 수 있다는 것을 알려주는 문제.

```py

def product_of_all_others(nums):
    def product_all(output, ns, rng):
        so_far = 1
        for i in rng:
            output[i] = so_far * output[i]
            so_far *= ns[i]
    size = len(nums)
    prod = [1] * size
    product_all(prod, nums, range(size))
    product_all(prod, nums, range(size - 1, -1, -1))
    return prod

```

직접 실행해보기: <https://tech.io/snippet/9ttNwnC>{:target="_blank"}

## Lazy range in Python 3

Python 3는 range가 lazy 하다는 것을 주목하자. Python 2에는 xrange가 있지만 Python 3의 range가 여러가지로 더 좋아졌다고 한다.
<https://treyhunner.com/2018/02/python-3-s-range-better-than-python-2-s-xrange/>{:target="_blank"}

# Takeaway

* 이런 문제는 좀 내지말자.
* 나중에 DP 문제들을 풀 때 비슷한 트릭을 써먹을 수 있다는 것은 기억해두자.
