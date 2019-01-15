---
title: "카카오 신입 공채 코딩 테스트: 다트 게임"
published: true
---

국내 회사들도 개발자 채용을 위해 코딩 테스트를 적극 도입하고 있다. 얼마전 화제가 되었던 카카오 신입 공채 코딩 테스트 문제 중에서 하나를 풀어보자.

# 문제

카카오 신입 공채 1차 코딩 테스트 - '다트 게임' 점수 계산

[http://tech.kakao.com/2017/09/27/kakao-blind-recruitment-round-1](http://tech.kakao.com/2017/09/27/kakao-blind-recruitment-round-1)

# 풀이

```py
import re

def char_to_power(ch):
    if ch == 'S':
        return 1
    elif ch == 'D':
        return 2
    elif ch == 'T':
        return 3
    assert False

def str_to_iter_tuples(score_str):
    regex_pattern = r"(\d+)([DST])(#|\*)?"
    conv = lambda groups: (int(groups[0]), char_to_power(groups[1]), groups[2])
    return (conv(m.groups()) for m in re.finditer(regex_pattern, score_str))

def tuples_to_score(iter_tps):
    prev_pt = 0
    total_score = 0
    for pt, pw, opt in iter_tps:
        point = pt ** pw
        if opt == '#':
            point *= -1
            prev_pt = point
            total_score += point
        elif opt == '*':
            point *= 2
            total_score += point + prev_pt
            prev_pt = point
        else:
            prev_pt = point
            total_score += point
    return total_score

def calculate_score(score_str):
    return tuples_to_score(str_to_iter_tuples(score_str))

###

test_cases = [
    ('1S2D*3T',  37, [(1,1,None), (2,2,'*'), (3,3,None)]),
    ('1D2S#10S',  9, [(1,2,None), (2,1,'#'), (10,1,None)]),
    ('1D2S0T',    3, [(1,2,None), (2,1,None), (0,3,None)]),
    ('1S*2T*3S', 23, [(1,1,'*'), (2,3,'*'), (3,1,None)]),
    ('1D#2S*3S',  5, [(1,2,'#'), (2,1,'*'), (3,1,None)]),
    ('1T2D3D#',  -4, [(1,3,None), (2,2,None), (3,2,'#')]),
    ('1D2S3T*',  59, [(1,2,None), (2,1,None), (3,3,'*')])
]

for score_str, _, tuples in test_cases:
    assert list(str_to_iter_tuples(score_str)) == tuples

for _, expected_score, tuples in test_cases:
    assert tuples_to_score(tuples) == expected_score

for score_str, expected_score, _ in test_cases:
    total_score = calculate_score(score_str)
    assert total_score == expected_score
    print(score_str, total_score)
```

직접 실행해보기: [https://tech.io/snippet/goMCpvi](https://tech.io/snippet/goMCpvi){:target="_blank"}

# Takeaways

* 간단한 정규식이라도 테스터 없이 만들기는 쉽지 않다.
* 문자열 파싱과 점수 계산 로직을 별도의 함수로 만들었다.
