---
title: "문자열에서 단어 순서 뒤집기 (공백처리 포함) - Reverse words in a string"
published: true
---

# 문제

주어진 문자열에 나오는 단어들의 순서를 뒤집는 함수를 작성하라.

```
입력: "the sky is blue"
결과: "blue is sky the"

입력: "   the  sky   is blue    "
결과: "blue is sky the"
```

추가조건:

* '단어'는 '공백문자 (space)'가 아닌 문자들의 연속으로 정의한다.
* 입력 문자열은 앞부분 또는 뒷부분에 공백문자들이 있을 수 있다.
* 결과 문자열은 앞부분/뒷부분의 공백문자들이 제거되어 있어야 한다.
* 단어 사이에 있는 두 개 이상의 연속된 공백문자들은 하나의 공백문자로 줄여서 출력해야 한다.

**도전과제: C 프로그래머라면 ~~아재라면~~ in-place로 동작하는 O(1) 공간 복잡도로 풀어 보자**

물론 시간 복잡도는 O(n)

출처: Leetcode 151 - [https://leetcode.com/problems/reverse-words-in-a-string](https://leetcode.com/problems/reverse-words-in-a-string)

# 풀이

결론부터 이야기하면 이 문제를 in-place로 푸는 방법은

'주어진 문자열을 처음부터 끝까지 전부 뒤집고, 다시 각 단어의 시작과 끝을 찾아가면서 단어마다 부분적으로 문자열을 뒤집어 주면 총 2번 스캔을 하므로 O(2n) = O(n) 으로 풀 수 있는 문제다.'

프로그래밍 문제라기보다 '요런 방법은 몰랐지?' 하는 식의 퍼즐에 가까운 이런 문제들을 별로 좋아하지 않는다. 하지만 추가로 붙어있는 중복된 공백 제거 조건 때문에 이 문제가 흥미로워졌다. 인터넷에 있는 풀이들을 찾아보니 문자열을 뒤집는 동작과는 별도로 중복된 공백을 찾아서 문자열을 shift하면서 공백을 제거하는 단계를 추가하는 방식으로 푼 경우가 많았다. 과연 문자열을 shift하는 동작을 하지 않고 문제를 풀 수는 없을까 하는 고민을 해보았다. 하나씩 풀어나가보자.

## 공백문자 제거 없이 풀기

우선 쉬운 경우를 먼저 풀어보자. 어떻게 하더라도 문자열에서 두 개의 문자를 교환하는 동작은 반드시 필요하다. 두 개의 문자를 바꾸는 함수를 만들어보자.

```c
void swap(char* pa, char* pb) {
  char ch = *pa;
  *pa = *pb;
  *pb = ch;
}
```

어디서 많이 본듯한 이 함수는 C언어 문법책에서 포인터를 배우면서 지겹도록 봤던 swap 함수다. (모르나? --;)

'XOR을 이용하면 더 빠르게 swap을 할 수 있어요.'
이런 주장을 하는 경우가 있는데, 실전에서는 웬만하면 그러지 말자.
* <https://betterexplained.com/articles/swap-two-variables-using-xor>
* <https://en.wikipedia.org/wiki/XOR_swap_algorithm#Reasons_for_avoidance_in_practice>

그럼, swap 함수를 이용해서 문자열을 뒤집는 함수를 만들어보자.

```c
void reverse(char* begin, char* end) {
  while (begin + 1 < end) {
    swap(begin++, --end);
  }
}
```

정확하게는 문자열 뿐 아니라 [begin, end) 메모리 구간의 연속적인 (contiguous) char 데이터들을 모두 뒤집는 함수다. C언어의 문자열은 다름아닌 '\0' 문자 (null문자) 로 끝나는 연속된 char형 (또는 적절한 문자형 - unsigned char, wchar, or 뭐든지) 의 배열이다.

문자열을 다룰 때는 매번 포인터로 다루는 것 보다는 문자열 내의 인덱스로 다루는 것이 더 편하다. 어떤 문자열에서 [ibegin, iend) 의 구간을 뒤집는 함수를 만들자.

```c
void reverse_by_index(char* str, size_t ibegin, size_t iend) {
  reverse(str + ibegin, str + iend);
}
```

또는,

```c
void reverse_by_index_easy(char* str, size_t ibegin, size_t iend) {
  size_t to = (iend - ibegin) / 2;
  for (size_t i = 0; i < to; i++) {
    swap(str + ibegin + i, str + iend - 1 - i);
  }
}
```
함수도 가능하다.

이 함수들을 이용해서 문자열에서 단어별로 뒤집는 함수를 만들어보자.

```c
void reverse_by_words(char* str, size_t length)
{
  size_t ibegin = 0;
  for (size_t i = 0; i < length + 1; i++) {
    if (*(str + i) == ' ' || *(str + i) == '\0') {
      reverse_by_index(str, ibegin, i);
      ibegin = i + 1;
    }
  }
}
```

지금까지 만든 함수들을 이용해서 중복된 공백문자가 없는 문자열의 단어들을 뒤집는 함수를 만들어보자.

```c
void reverse_words(char* str, size_t length) {
  reverse_by_index(str, 0, length);
  reverse_by_words(str, length);
}
```

직접 실행해보기: [https://tech.io/snippet/BxZcOlN](https://tech.io/snippet/BxZcOlN)

## 공백문자 처리 포함

문자열의 시작부분에 공백문자들을 제거하는 trim_left 함수는 문자열의 시작 위치를 바꿔주지 않는 한, 문자열의 내용을 왼쪽으로 shift 시키는 동작을 해야만 한다. 반면에 trim_right는 맨 오른쪽에 있는 null 문자의 위치만 바꿔주면 되기 때문에 훨씬 효과적이다.

```c
size_t trim_right(char* str, size_t length) {
  size_t i = length - 1;
  for (; i >= 0 && *(str + i) == ' '; i--) {}
  if (i != length - 1) {
    *(str + i + 1) = '\0';
  }
  return i + 1;
}
```

따라서 trim_right 만으로 중복된 공백들을 전부 없앨 수 있다면 효과적인 구현이 가능할 것이다. 우리가 문자열을 어차피 두번이나 뒤집을 것이기 충분히 가능하다.

원하는 동작은
```
"  the   sky   is  blue   "

"   eulb  si   yks   eht  "
"blue     si   yks   eht  "
"blue is       yks   eht  "
"blue is sky         eht  "

"blue is sky the          "

"blue is sky the"
```
이런 단계를 거쳐서 답을 얻고 싶다. 매번 단어들을 뒤집을 때마다 공백문자들을 오른쪽으로 몰아가는 동작이 된다는 것을 볼 수 있다.

중복된 공백을 고려해서 단어 부분들을 뒤집는 함수는 아래와 같은 방식으로 가능하다.
```c
void reverse_by_words_with_trim(char* str, size_t length)
{
  size_t redundant_spaces = 0;
  size_t chars = 0;
  size_t ibegin = 0;
  for (size_t i = 0; i < length + 1; i++) {
    if (*(str + i) == ' ' || *(str + i) == '\0') {
      if (chars > 0) {
        reverse_by_index(str, ibegin, i);
        ibegin = i - redundant_spaces + 1;
        chars = 0;
      }
      else {
        redundant_spaces++;
      }
    }
    else {
      chars++;
    }
  }
}
```

완성된 함수는,

1. 먼저 전체 문자열을 뒤집는다.
2. 각 부분을 뒤집을 때 단어의 왼쪽에 있는 중복된 공백문자들을 포함해서 뒤집는다.
3. 맨 마지막으로 trim_right를 한번만 적용한다.

```c
size_t reverse_words_with_trim(char* str, size_t length) {
  reverse_by_index(str, 0, length);
  reverse_by_words_with_trim(str, length);
  return trim_right(str, length);
}
```

직접 실행해보기: [https://tech.io/snippet/8Bspy4P](https://tech.io/snippet/8Bspy4P)

# Takeaway

* C언어 문자열 처리는 위험하다
* 실제로 그림을 그려보면서 구현하지 않으면 인덱스와 포인터 계산 맞추기가 쉽지 않다.
    * 그림 보면서 해도 어렵다.
* 단순한 코딩 문제라도 완결된 동작으로 분리 가능한 부분들을 완결된 함수로 분할해서 구현하는 것이 좋다.
    * Divede & conquer는 알고리즘 전략 뿐 아니라 구현 전략으로써도 중요하다.
* C언어 계열이 아닌 immutable string을 사용하는 언어들 (Python, Java 등) 에서는 in-place로 하려고 괜히 땀 빼지 말자.
