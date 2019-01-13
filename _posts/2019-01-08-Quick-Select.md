---
title: "Quick select"
published: true
---

# 문제

정렬되어 있지 않은 중복없는 숫자들의 배열에서 k번째로 작은 숫자를 찾아라. (k는 언제나 유효한 값 이라고 가정한다. 1 <= k  <= length of array)

```py
find_kth([7, 10, 4, 3, 20, 15], 3)
7

find_kth([7, 10, 4, 3, 20, 15], 4)
10
```

* 출처

GeeksforGeeks [https://www.geeksforgeeks.org/kth-smallestlargest-element-unsorted-array](https://www.geeksforgeeks.org/kth-smallestlargest-element-unsorted-array)


# 풀이

간단하게 생각하면 정렬하고 k번째 (인덱스 k - 1) 위치의 값을 꺼내면 되겠다.

```py
def find_kth_naive(nums, k):
    return sorted(nums)[k - 1]
```
시간복잡도는 O(NlogN)

그런데, 이렇게 간단하게 끝날 문제일리가 없다. 평균 시간복잡도 O(N)으로 동작하게 만들어보자.

단순한 정렬은 일반적인 방법으로는 NlogN이 최선이다. 하지만, 우리가 필요한 값은 상위 k까지의 값이므로 필요한 부분까지만 정렬을 하면 성능을 개선할 수 있지 않을까? 부분으로 나눠서 각 부분을 정렬하는 알고리즘 하면 Quicksort가 가장 먼저 떠오른다.

## Quicksort

분할과 정복 (Divede & conquer) 방법으로 정렬하는 대표적인 알고리즘 Quicksort를 직접 만들어 보자.

```py
def qsort_rec(arr):
    if not arr:
        return []
    pivot = arr[-1]
    lesser = [n for n in arr[:-1] if n < pivot]
    greater = [n for n in arr[:-1] if pivot < n]
    return qsort_rec(lesser) + [pivot] + qsort_rec(greater)
```

알고리즘의 재귀적인 성질 그대로 재귀 호출로 구현한 Quicksort는 이해하기 쉽고 간결하다. 그러나 매번 새로운 배열을 생성하는 것은 메모리 낭비다. 순수 함수형 프로그래밍 언어가 아니라면 in-place Quicksort를 구현할 필요가 있다.

## Lomuto's partition scheme

in-place Quicksort를 만들기 위해서 이해해야 할 것은 분할 (partition) 함수의 동작 방식이다.

여러가지 방법들이 오래 전부터 연구되어 있는데 그 가운데 Lomuto 방법이 이해하기 쉬운 편이다.
[https://en.wikipedia.org/wiki/Quicksort#Lomuto_partition_scheme](https://en.wikipedia.org/wiki/Quicksort#Lomuto_partition_scheme)

![](https://upload.wikimedia.org/wikipedia/commons/8/84/Lomuto_animated.gif)

Lomuto 방법으로 partition 함수를 만들어 보자.

```py
def partition(arr, begin, end):
    i = j = begin
    pivot = arr[end - 1]
    while j < end - 1:
        if arr[j] < pivot:
            if i != j:
                arr[i], arr[j] = arr[j], arr[i]
            i += 1
        j += 1
    arr[i], arr[end - 1] = arr[end - 1], arr[i]
    return i
```
직접 실행해보기 [https://tech.io/snippet/I3xUYbT](https://tech.io/snippet/I3xUYbT){:target="_blank"}

* 맨 마지막 요소를 pivot으로 정하고
* 두개의 포인터를 전진시키면서 pivot을 제외한 값들을 교환해 가면서 lesser그룹과 greater group으로 분할하고
* 끝으로 pivot과 greater의 첫번째 요소를 교환하고
* pivot의 index를 반환한다

또 다른 분할 방법 중에 유명한 Hoare 방법으로도 partition 함수를 만들어 보자.
[https://en.wikipedia.org/wiki/Quicksort#Hoare_partition_scheme](https://en.wikipedia.org/wiki/Quicksort#Hoare_partition_scheme)

## in-place Quicksort

partition 함수를 이용하면 in-place Quicksort 를 쉽게 만들수 있다.

```py
def qsort_in_place(arr, begin, end):
    if begin < end:
        pivot = partition(arr, begin, end)
        qsort_in_place(arr, begin, pivot)
        qsort_in_place(arr, pivot + 1, end)
```

* 반닫힌구간을 사용하는 것을 눈여겨 보자
* 오른쪽 분할 영역으로 재귀 호출을 할 때 pivot + 1 부터 시작해야 하는 것을 잊지말자

[https://tech.io/snippet/35ka5T6](https://tech.io/snippet/35ka5T6){:target="_blank"}


## k번째 작은 값 찾기

분할을 반복하면서 정렬하는 Quicksort의 성질을 잘 살펴보면 k번째로 작은 값을 찾기 위해서 모든 영역을 정렬할 필요는 없다는 것을 알 수 있다.

k번째 값을 찾는 과정에서 최선의 경우를 생각해보면

* 전체 배열에 대해서 pivot을 잡고 분할을 했더니 lesser 그룹의 갯수가 우연히 (k - 1) 개로 딱 떨어지는 경우가 되겠다. 즉, 처음 잡은 pivot이 그냥 k번째 값인 경우다.
    * 이때는 그냥 pivot의 값을 반환하면 문제해결!

최선의 경우가 아니라면 재귀 호출을 반복하게 되는데, 다음 두 가지 경우가 있다.

* pivot의 순위가 k보다 작은 경우 (pivot 보다 작은 값들이 k - 1개 보다 적은 경우)
    * 이제 pivot 왼쪽 부분은 굳이 정렬할 필요가 없다.
    * 오른쪽 분할 부분에서 (k - pivot의 순위) 번째 작은 값을 찾도록 재귀 호출

* pivot의 순위가 k보다 큰 경우 (pivot 보다 작은 값들이 k - 1개 보다 많은 경우)
    * pivot 오른쪽은 신경 쓸 필요가 없다. 왼쪽 분할 영역에서 k번째 값을 찾도록 재귀 호출

```py
def find_kth_smallest(arr, begin, end, k):
    if begin < end:
        i_pivot = partition(arr, begin, end)
        pivot_rank = i_pivot - begin + 1  # rank is 1 base
        if pivot_rank == k:
            return arr[i_pivot]
        if pivot_rank < k:
            return find_kth_smallest(arr, i_pivot + 1, end, k - pivot_rank)
        return find_kth_smallest(arr, begin, i_pivot, k)
    raise ValueError('end should be greater than begin')
```

[https://tech.io/snippet/xh1ITaO](https://tech.io/snippet/xh1ITaO){:target="_blank"}

### Time complexity

* 최악의 경우는 O(N^2)이지만 평균적으로 O(N)의 성능을 얻을 수 있다.
    * ~~여백이 좁아서 증명은 생략한다...~~

# Takeaways

* 불필요한 작업은 아예 하지 않도록 알고리즘을 만들면 아주 좋다.
* 복잡한 경계값 판단을 쉽게 하기 위해서 변수의 의미를 조금 다르게 정의했더니 이해가 쉬워졌다.
    * elems_in_left를 기준으로 재귀 호출 영역을 찾으려고 했을때 보다 pivot_rank로 생각을 했더니 더 이해하기 쉽고 직관적으로 코드가 나왔다.
