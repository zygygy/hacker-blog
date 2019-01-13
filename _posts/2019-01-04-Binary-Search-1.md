---
title: "Binary Search #1"
published: true
---

# 문제

정렬된 배열에서 특정 숫자가 몇 번 나오는지 반환하는 함수를 작성하라.

```py
count_numbers([1, 1, 2, 2, 2, 2, 3], 1)
2

count_numbers([1, 1, 2, 2, 2, 2, 3], 2)
4

count_numbers([1, 1, 2, 2, 2, 2, 3], 3)
1

count_numbers([1, 1, 2, 2, 2, 2, 3], 4)
0
```

* 출처

GeeksforGeeks [https://www.geeksforgeeks.org/count-number-of-occurrences-or-frequency-in-a-sorted-array](https://www.geeksforgeeks.org/count-number-of-occurrences-or-frequency-in-a-sorted-array)


# 풀이

정렬된 데이터에서 특정 값을 찾는 문제를 만나면 조건반사적으로 이진탐색(Binary Search)을 떠올려야 한다. 이진탐색은 O(logN)의 좋은 성능을 얻을 수 있는 분할과 정복 (Divide & conquer) 방식의 대표적인 알고리즘이다.

이 문제를 풀기 위해서는 값 하나를 찾아내는 것보다는 찾으려는 숫자들의 맨 왼쪽 인덱스와 맨 오른쪽 인덱스를 알아내는 것이 더 편리하다. python 표준 라이브러리에서 제공하는 bisect_left와 bisect_right 함수를 사용하면 아주 간단하게 최적의 해답을 작성할 수 있다. 시간 복잡도 O(logN)

```py
import bisect

def count_numbers(nums, n):
    return bisect.bisect_right(numbers, n) - bisect.bisect_left(numbers, n)

```

직접 실행해보기 [https://tech.io/snippet/MQ8AGxH](https://tech.io/snippet/MQ8AGxH){:target="_blank"}

이렇게 쉽게 끝나면 조금 아쉽다. 당연히 "bsearch 또는 bisect를 직접 구현하셔야 합니다." 라고 하지 않겠는가?

# 이진탐색 Binary search

이진탐색을 구현하는 것은 언뜻 생각하면 쉬울 것 같다. 하지만, 막상 버그 없이 구현하려면 처음 생각처럼 만만하지가 않다.

구현을 하기 전에 몇 가지를 정해야 한다.

* 검색의 범위를 닫힌구간으로 할지 반닫힌구간으로 할지?
    * 반단힌구간이 여러가지면에서 구현이 깔끔하다. 실제로 표준 라이브러리들은 반닫힌구간으로 구현된 것들이 많다.
* 구현을 재귀적으로 할지 반복적으로 할지?
    * 알고리즘의 특징상 재귀적으로 구현하는 것이 깔끔하지만 반복적인 구현도 나쁘지 않다.

우선, 단순한 binary_search 함수를 만들어 보자.

```py
def bsearch(arr, value, begin, end):
    while begin < end:
        mid = begin + (end - begin) // 2
        if arr[mid] == value:
            return mid
        if arr[mid] < value:
            begin = mid + 1
        else:
            end = mid
    return -1
```
[https://tech.io/snippet/QskgUgD](https://tech.io/snippet/QskgUgD){:target="_blank"}

지금 만든 bsearch 함수는 같은 값이 두 개 이상 있는 배열에서도 값을 찾자마자 함수를 반환한다. 우리는 중복된 값이 있는 배열을 다루고 싶기 때문에 찾으려는 값의 맨 첫번째 위치와 맨 마지막 위치를 알고싶다. C++의 [std::lower_bound](https://en.cppreference.com/w/cpp/algorithm/lower_bound), [std::upper_bound](https://en.cppreference.com/w/cpp/algorithm/upper_bound), 또는 python의 [bisect_left](https://docs.python.org/2/library/bisect.html#bisect.bisect_left), [bisect_right](https://docs.python.org/2/library/bisect.html#bisect.bisect_right) 같은 함수가 필요하다.

그럼, bisect_left를 만들어 보자.

```py
def bisect_left(arr, value, begin, end):
    if begin >= end:
        return begin
    mid = begin + (end - begin) // 2
    if arr[mid] < value:
        return bisect_left(arr, value, mid + 1, end)
    else:
        return bisect_left(arr, value, begin, mid)
```

아주 유용한 함수이기 때문에 **꼭 외워두자!** 재귀적인 구현이 좀 더 직관적이고 외우기 쉽다.

cpython의 실제 구현은 iterative 하게 되어있다. [https://github.com/python/cpython/blob/master/Lib/bisect.py#L63](https://github.com/python/cpython/blob/master/Lib/bisect.py#L63)

몇가지 특징을 살펴보자.

* 반환되는 값의 범위는 [0, len(arr)]
* 반환된 index에 찾고자 하는 value가 있을 수도 있고 아닐 수도 있다. (bsearch와 다른점)
* bisect_left는 arr이 정렬된 상태를 유지한채로 value를 insert할 수 있는 왼쪽편의 index를 반환한다.
* begin과 end가 같은 값이면 빈공간

대칭적인 함수로 bisect_right를 만들어 보자.

```py
def bisect_right(arr, value, begin, end):
    if begin >= end:
        return begin
    mid = begin + (end - begin) // 2
    if value < arr[mid]:
        return bisect_right(arr, value, begin, mid)
    else:
        return bisect_right(arr, value, mid + 1, end)
```

코드의 흐름을 기억하기 위한 요점은,

* bisect는 lessthan 연산만으로 탐색을 한다. equal은 사용하지 않는다.
* 분할지점 (pivot) 위치의 값과 찾으려는 **값이 같을 때**
    * bisect_left는 분할 배열의 왼쪽을
    * bisect_right는 오른쪽 부분 배열을 선택하는 과정을 눈여겨 보자.
* 값을 찾았거나 찾지 못했거나, 마지막으로 분할한 부분 배열의 begin 을 반환한다.

# bsearch in terms of bisect_left

bisect_left로 bsearch를 쉽게 구현할 수 있다.

```py
def bsearch(arr, value, begin, end):
    found = bisect_left(arr, value, begin, end)
    return found if found < len(arr) and arr[found] == value else -1
```

# Takeaways

* 이진탐색은 쉬워 보이지만 꼭 직접 만들어 보면서 익혀둬야 하는 알고리즘이다.
* 어떤 문제든 정렬되어 있는 데이터가 언급되면 일단 이진탐색으로 해결할 수 있는지 생각해보자.
