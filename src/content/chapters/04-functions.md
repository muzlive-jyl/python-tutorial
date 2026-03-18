---
title: "함수 만들기"
description: "def, return, 매개변수, 기본값 — 코드를 재사용하는 방법"
order: 4
part: 1
---

## 함수란?

함수는 **특정 작업을 수행하는 코드 묶음**에 이름을 붙인 것입니다. 한 번 만들어두면 여러 곳에서 재사용할 수 있습니다.

> 비유하면: 함수는 **자판기**와 같습니다. 동전(입력)을 넣으면 음료(출력)가 나옵니다. 자판기 내부가 어떻게 동작하는지 몰라도, 동전만 넣으면 됩니다.

## 함수 정의하기

```python
def greet(name):
    return f"안녕하세요, {name}님!"

# 함수 호출
message = greet("파이썬")
print(message)  # "안녕하세요, 파이썬님!"
```

- `def` — 함수를 정의하는 키워드
- `greet` — 함수 이름
- `name` — **매개변수(parameter)** — 함수에 전달되는 입력
- `return` — 결과를 돌려주는 키워드

## 우리 프로젝트의 함수들

### `strip_ass_tags()` — 단순한 입력-출력 함수

```python
# ass_parser.py
def strip_ass_tags(text):
    """ASS 태그를 모두 제거한 순수 텍스트를 반환한다."""
    return TAG_PATTERN.sub('', text)
```

- 입력: ASS 태그가 포함된 텍스트 `"{\K432}여행을 떠나요"`
- 출력: 태그가 제거된 텍스트 `"여행을 떠나요"`

### `parse_ass_time()` — 여러 값을 반환하는 함수

```python
# ass_parser.py
def parse_ass_time(time_str):
    """ASS 시간 문자열을 (분, 초, 센티초) 튜플로 변환한다."""
    h, m, s = time_str.strip().split(':')
    total_minutes = int(h) * 60 + int(m)
    sec_parts = s.split('.')
    seconds = int(sec_parts[0])
    cs = int(sec_parts[1][:2]) if len(sec_parts) == 2 else 0
    return total_minutes, seconds, cs
```

파이썬에서는 **쉼표로 구분**하면 여러 값을 한 번에 반환할 수 있습니다. 실제로는 **튜플**이 반환됩니다.

```python
result = parse_ass_time("0:00:11.64")
# result = (0, 11, 64) — 튜플

# 언패킹으로 바로 변수에 저장
tm, sec, cs = parse_ass_time("0:00:11.64")
# tm=0, sec=11, cs=64
```

### `format_lrc_time()` — 여러 매개변수 받기

```python
# ass_parser.py
def format_lrc_time(tm, sec, cs):
    """(분, 초, 센티초)를 LRC 타임스탬프 문자열로 변환한다."""
    return f'[{tm:02d}:{sec:02d}.{cs:02d}]'
```

```python
format_lrc_time(0, 15, 96)  # "[00:15.96]"
```

## 기본값이 있는 매개변수

매개변수에 기본값을 지정할 수 있습니다. 호출할 때 생략하면 기본값이 사용됩니다.

```python
# ass2lrc.py
def convert_ass_to_lrc(filepath, output_path=None):
    # output_path를 지정하지 않으면 None이 됨
    if output_path is None:
        base, _ = os.path.splitext(filepath)
        output_path = base + '.lrc'
```

호출 방법:

```python
convert_ass_to_lrc("input.ass")                    # output_path=None
convert_ass_to_lrc("input.ass", "output.lrc")      # output_path="output.lrc"
convert_ass_to_lrc("input.ass", output_path="o.lrc") # 이름으로 지정도 가능
```

## 독스트링(Docstring)

함수 바로 아래에 `"""..."""`로 설명을 달 수 있습니다. 이것을 **독스트링**이라고 합니다.

```python
def add_centiseconds(tm, sec, cs, delta_cs):
    """주어진 시간에 delta_cs 센티초를 더한 결과를 (분, 초, 센티초) 튜플로 반환한다."""
    total_cs = cs + delta_cs
    extra_sec, new_cs = divmod(total_cs, 100)
    total_sec = sec + extra_sec
    extra_min, new_sec = divmod(total_sec, 60)
    return tm + extra_min, new_sec, new_cs
```

독스트링은 `help()` 함수로 확인할 수 있습니다:

```python
help(add_centiseconds)
# "주어진 시간에 delta_cs 센티초를 더한 결과를..."
```

## `*` 언패킹 연산자

함수를 호출할 때 `*`를 사용하면 튜플이나 리스트를 **펼쳐서** 전달할 수 있습니다.

```python
adjusted = (0, 15, 96)  # 튜플
format_lrc_time(*adjusted)  # format_lrc_time(0, 15, 96)과 동일
```

우리 프로젝트에서:

```python
# ass2lrc.py — 프리롤이 적용된 시간을 LRC 포맷으로 변환
adjusted = add_centiseconds(tm, sec, cs, preroll_cs)
timestamp = format_lrc_time(*adjusted)
```

`*adjusted`는 `(0, 15, 96)`을 `0, 15, 96`으로 풀어서 전달합니다.

## 내장 함수 vs 사용자 정의 함수

파이썬에는 미리 만들어진 **내장 함수(built-in function)**가 있습니다.

우리 프로젝트에서 사용하는 내장 함수들:

| 함수 | 용도 | 예시 |
|------|------|------|
| `print()` | 출력 | `print("Hello")` |
| `len()` | 길이 | `len([1,2,3])` → `3` |
| `int()` | 정수 변환 | `int("432")` → `432` |
| `range()` | 범위 생성 | `range(5)` → 0,1,2,3,4 |
| `sorted()` | 정렬 | `sorted([3,1,2])` → `[1,2,3]` |
| `enumerate()` | 인덱스 부여 | 아래 예시 참고 |
| `zip()` | 묶기 | 아래 예시 참고 |
| `divmod()` | 몫+나머지 | `divmod(432,100)` → `(4,32)` |
| `sum()` | 합계 | `sum([1,2,3])` → `6` |
| `max()` | 최대값 | `max(3,7,1)` → `7` |
| `open()` | 파일 열기 | Chapter 6에서 자세히 |

### `enumerate()` — 인덱스와 함께 반복

```python
lines = ["가사1", "가사2", "가사3"]
for i, line in enumerate(lines):
    print(f"{i}: {line}")
# 0: 가사1
# 1: 가사2
# 2: 가사3
```

우리 프로젝트에서:

```python
# validate_lrc.py — 줄 번호를 추적하며 검사
for i, line in enumerate(content_lines):
    # i는 줄 번호, line은 내용
```

### `zip()` — 두 리스트를 묶기

```python
keys = ["Format", "Layer", "Start"]
values = ["0", "0", "0:00:11.64"]
pairs = dict(zip(keys, values))
# {"Format": "0", "Layer": "0", "Start": "0:00:11.64"}
```

우리 프로젝트에서 ASS의 Format 행과 실제 데이터를 매핑합니다:

```python
# ass_parser.py
field_map = dict(zip(format_fields, parts))
```

### `lambda` — 한 줄 함수

간단한 함수를 한 줄로 만들 수 있습니다.

```python
# 일반 함수
def get_time(item):
    return item[0]

# lambda로 쓰면
get_time = lambda item: item[0]
```

우리 프로젝트에서 정렬 기준으로 사용됩니다:

```python
# ass2lrc.py — 타임스탬프 기준으로 정렬
sorted(dialogues, key=lambda x: x[0])
```

## 핵심 정리

| 개념 | 문법 | 예시 |
|------|------|------|
| 함수 정의 | `def 이름(매개변수):` | `def greet(name):` |
| 반환 | `return 값` | `return tm, sec, cs` |
| 기본값 | `매개변수=기본값` | `output_path=None` |
| 독스트링 | `"""설명"""` | 함수 바로 아래 |
| 언패킹 호출 | `함수(*튜플)` | `format_lrc_time(*adjusted)` |
| lambda | `lambda 매개변수: 표현식` | `lambda x: x[0]` |

## 직접 해보기

1. `add_centiseconds(0, 11, 64, 432)`를 호출하고 결과를 확인해보세요. (힌트: 4.32초를 더하면?)
2. `format_lrc_time`과 `add_centiseconds`를 조합해서 `"[00:15.96]"`을 만들어보세요.
3. `sorted([(3,"c"), (1,"a"), (2,"b")], key=lambda x: x[0])`의 결과를 확인해보세요.
