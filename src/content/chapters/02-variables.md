---
title: "변수와 데이터 타입"
description: "파이썬의 기본 데이터 타입 — int, str, list, dict, tuple, bool"
order: 2
part: 1
---

## 변수란?

변수는 값을 담아두는 **이름표**입니다. 파이썬에서는 `=` 기호로 값을 저장합니다.

```python
name = "ass2lrc"
version = 1
```

> 비유하면: 변수는 **라벨이 붙은 상자**입니다. `name`이라는 상자에 `"ass2lrc"`를 넣어두면, 나중에 `name`이라고 부르면 그 값이 나옵니다.

파이썬은 **동적 타입 언어**입니다. 변수를 만들 때 타입을 명시하지 않아도 됩니다. 파이썬이 알아서 판단합니다.

## 기본 데이터 타입

### 숫자 — int, float

```python
preroll_cs = 432          # int: 정수
seconds = 11.64           # float: 소수점
total = preroll_cs + 100  # 532
```

우리 프로젝트에서 `preroll_cs`(카라오케 프리롤 시간)는 **센티초(centisecond)** 단위의 정수입니다.

산술 연산자:

| 연산자 | 의미 | 예시 | 결과 |
|--------|------|------|------|
| `+` | 더하기 | `432 + 100` | `532` |
| `-` | 빼기 | `432 - 100` | `332` |
| `*` | 곱하기 | `4 * 100` | `400` |
| `/` | 나누기 | `432 / 100` | `4.32` |
| `//` | 몫 | `432 // 100` | `4` |
| `%` | 나머지 | `432 % 100` | `32` |

`//`(몫)과 `%`(나머지)는 우리 프로젝트에서 시간 계산에 사용됩니다:

```python
# ass_parser.py에서 — 센티초를 초와 나머지로 분리
total_cs = centiseconds + delta_cs
extra_sec, new_cs = divmod(total_cs, 100)  # divmod는 몫과 나머지를 동시에 반환
```

`divmod(432, 100)`은 `(4, 32)`를 반환합니다 — 몫 4, 나머지 32.

### 문자열 — str

문자열은 텍스트 데이터입니다. 따옴표(`'` 또는 `"`)로 감싸서 만듭니다.

```python
title = "여행의 노래"
artist = '홍길동'
tag = "{\K432}"
```

문자열은 다음 챕터에서 자세히 다룹니다.

### 불리언 — bool

참(`True`) 또는 거짓(`False`) 두 가지 값만 가집니다.

```python
all_pass = True
has_error = False
```

우리 프로젝트의 `validate_lrc.py`에서:

```python
all_pass = True
# ... 검증 중 실패를 발견하면
if status == 'FAIL':
    all_pass = False
```

### None

"값이 없음"을 나타내는 특별한 값입니다.

```python
output_path = None  # 아직 출력 경로가 정해지지 않음
```

우리 프로젝트의 `ass2lrc.py`에서:

```python
def convert_ass_to_lrc(filepath, output_path=None):
    # output_path가 None이면 자동으로 경로를 생성
```

## 컬렉션 타입

### 리스트 — list

순서가 있는 값들의 모음입니다. `[]`로 만듭니다.

```python
lines = ["첫 번째 줄", "두 번째 줄", "세 번째 줄"]
numbers = [1, 2, 3, 4, 5]
empty = []
```

리스트의 기본 조작:

```python
lines = ["가", "나", "다"]

lines[0]          # "가" — 인덱스는 0부터 시작!
lines[-1]         # "다" — 음수 인덱스는 뒤에서부터
lines.append("라") # ["가", "나", "다", "라"] — 끝에 추가
len(lines)        # 4 — 길이
```

우리 프로젝트에서:

```python
# ass2lrc.py — LRC 줄들을 리스트에 모은 다음, 나중에 파일로 저장
lrc_lines = []
lrc_lines.append("[title:여행의 노래]")
lrc_lines.append("[00:15.96]여행을 떠나요")
```

### 딕셔너리 — dict

**키-값 쌍**으로 이루어진 데이터입니다. `{}`로 만듭니다.

```python
metadata = {
    "title": "여행의 노래",
    "artist": "홍길동",
    "font": "맑은 고딕"
}

metadata["title"]           # "여행의 노래"
metadata.get("album", "")   # "" — 키가 없으면 기본값 반환
```

우리 프로젝트에서 같은 타임스탬프의 대사를 모아주는 데 사용됩니다:

```python
# ass2lrc.py — 타임스탬프별로 대사를 그룹화
time_groups = {}
for time_key, style, text, preroll_cs in dialogues:
    if time_key not in time_groups:
        time_groups[time_key] = []
    time_groups[time_key].append(text)
```

`time_key not in time_groups`는 "이 키가 딕셔너리에 아직 없으면"이라는 뜻입니다.

### 튜플 — tuple

리스트와 비슷하지만 **변경할 수 없는(immutable)** 데이터입니다. `()`로 만듭니다.

```python
time_key = (0, 11, 64)  # (분, 초, 센티초)
```

우리 프로젝트에서 타임스탬프를 튜플로 표현합니다:

```python
# ass_parser.py
return total_minutes, int(sec), cs  # 튜플을 반환 (괄호 생략 가능)
```

튜플은 **언패킹(unpacking)**할 수 있습니다:

```python
tm, sec, cs = (0, 11, 64)  # tm=0, sec=11, cs=64
```

> 리스트 vs 튜플: 리스트(`[]`)는 내용을 바꿀 수 있고, 튜플(`()`)은 바꿀 수 없습니다. 타임스탬프처럼 "한번 정해지면 변하지 않는 값"에 튜플을 사용합니다. 또한 튜플은 딕셔너리의 키로 사용할 수 있지만, 리스트는 불가능합니다.

### 셋 — set

**중복이 없는** 값들의 모음입니다. `{}`로 만들지만, 빈 셋은 `set()`으로 만듭니다.

```python
expected = {"[font]", "[title]", "[artist]"}
found = {"[title]", "[artist]"}
missing = expected - found  # {"[font]"} — 차집합
```

우리 프로젝트의 `validate_lrc.py`에서 필수 헤더가 모두 있는지 확인할 때 사용됩니다.

## 타입 변환

값의 타입을 바꿀 수 있습니다:

```python
int("432")    # 432 — 문자열을 정수로
str(432)      # "432" — 정수를 문자열로
list("abc")   # ["a", "b", "c"] — 문자열을 리스트로
```

우리 프로젝트에서:

```python
# ass_parser.py — ASS 시간 문자열 "0:00:11.64"를 숫자로 변환
h, m, s = time_str.strip().split(':')
# h="0", m="00", s="11.64" — 이 시점에서는 전부 문자열!
hours = int(h)    # 0
minutes = int(m)  # 0
```

## Truthiness (참/거짓 판단)

파이썬에서는 `bool` 타입이 아닌 값도 참/거짓으로 판단합니다:

| 거짓(Falsy) | 참(Truthy) |
|-------------|------------|
| `False` | `True` |
| `0`, `0.0` | 0이 아닌 숫자 |
| `""` (빈 문자열) | `"abc"` |
| `[]` (빈 리스트) | `[1, 2]` |
| `{}` (빈 딕셔너리) | `{"a": 1}` |
| `None` | 대부분의 객체 |

우리 프로젝트에서:

```python
# ass2lrc.py — metadata가 비어있지 않으면 헤더를 추가
if metadata:  # 딕셔너리가 비어있으면 False
    lrc_lines.append(f"[title:{metadata['title']}]")
```

## 핵심 정리

| 타입 | 예시 | 프로젝트에서의 용도 |
|------|------|---------------------|
| `int` | `432` | 센티초, 줄 번호 |
| `float` | `11.64` | ASS 시간 값 |
| `str` | `"여행의 노래"` | 텍스트, 파일 경로 |
| `bool` | `True` / `False` | 검증 결과 |
| `None` | `None` | 기본 매개변수 |
| `list` | `["a", "b"]` | LRC 줄들, 대사 목록 |
| `dict` | `{"key": "val"}` | 타임스탬프 그룹, 메타데이터 |
| `tuple` | `(0, 11, 64)` | 타임스탬프, 함수 반환값 |
| `set` | `{"[font]"}` | 중복 검사, 헤더 검증 |

## 직접 해보기

1. 변수를 만들어 `type()` 함수로 타입을 확인해보세요: `print(type(432))`
2. 리스트를 만들고 `.append()`로 값을 추가한 뒤 `len()`으로 길이를 확인해보세요.
3. 딕셔너리에 `in` 연산자를 사용해보세요: `print("title" in {"title": "test"})`
