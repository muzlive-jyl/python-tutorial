---
title: "문자열 다루기"
description: "strip, split, join, f-string, 슬라이싱 — 텍스트 처리의 핵심"
order: 3
part: 1
---

## 왜 문자열이 중요한가요?

우리 프로젝트는 **텍스트 파일을 읽고 변환하는** 프로그램입니다. ASS 파일의 텍스트를 파싱하고, LRC 형식으로 변환하고, 검증하는 모든 과정이 문자열 처리입니다. 문자열을 잘 다루는 것이 곧 이 프로젝트를 이해하는 열쇠입니다.

## 문자열 만들기

```python
single = '작은따옴표'
double = "큰따옴표"
multi = """여러 줄에 걸친
문자열도 가능합니다"""
raw = r'\K432'  # 원시 문자열: 역슬래시를 그대로 유지
```

**원시 문자열(raw string)** `r'...'`은 우리 프로젝트에서 정규표현식 패턴에 사용됩니다:

```python
# ass_parser.py
TAG_PATTERN = re.compile(r'\{[^}]*\}')  # r 없으면 \{가 오류 발생
```

## 문자열 메서드

### `.strip()` — 양쪽 공백 제거

```python
line = "  Hello World  \n"
line.strip()    # "Hello World"
line.rstrip()   # "  Hello World" — 오른쪽만 제거
line.lstrip()   # "Hello World  \n" — 왼쪽만 제거
```

우리 프로젝트에서 파일을 읽을 때 줄바꿈 문자를 제거합니다:

```python
# ass_parser.py
lines = [l.rstrip('\n').rstrip('\r') for l in lines]
```

### `.split()` — 문자열 분리

```python
"a,b,c".split(',')         # ["a", "b", "c"]
"Hello World".split()       # ["Hello", "World"] — 공백 기준
"0:00:11.64".split(':')     # ["0", "00", "11.64"]
```

우리 프로젝트에서 ASS 시간을 파싱합니다:

```python
# ass_parser.py — "0:00:11.64"를 시, 분, 초로 분리
h, m, s = time_str.strip().split(':')
# h="0", m="00", s="11.64"
```

### `.join()` — 문자열 합치기

`.split()`의 반대입니다. 리스트의 요소를 하나의 문자열로 합칩니다.

```python
words = ["여행을", "떠나요"]
" ".join(words)     # "여행을 떠나요"
"\n".join(words)    # "여행을\n떠나요" (줄바꿈으로 연결)
```

우리 프로젝트에서 같은 타임스탬프의 대사를 합칠 때:

```python
# ass2lrc.py — 여러 대사를 공백으로 합침
merged_text = ' '.join(time_groups[time_key])
```

그리고 LRC 파일을 저장할 때:

```python
# ass2lrc.py — 전체 LRC 내용을 줄바꿈으로 합쳐 파일에 쓰기
f.write('\n'.join(lrc_lines))
```

### `.startswith()` / `.endswith()` — 접두/접미 확인

```python
"[00:15.96]여행을 떠나요".startswith('[')   # True
"output.lrc".endswith('.lrc')               # True
```

우리 프로젝트에서 LRC 라인인지 판별할 때:

```python
# validate_lrc.py — 타임스탬프 라인 찾기
if line.strip().startswith('['):
    # 이 줄은 LRC 타임스탬프나 헤더
```

### `.replace()` — 문자열 치환

```python
text = "Hello World"
text.replace("World", "Python")  # "Hello Python"
```

## 문자열 슬라이싱

문자열의 일부분을 잘라낼 수 있습니다. `문자열[시작:끝]` 형태입니다.

```python
s = "[00:15.96]여행을 떠나요"

s[0]      # "["
s[1:3]    # "00"       — 인덱스 1부터 2까지 (끝은 미포함)
s[:9]     # "[00:15.96" — 처음부터 8까지
s[10:]    # "여행을 떠나요" — 10부터 끝까지
s[-3:]    # "떠나요"    — 뒤에서 3글자
```

> 핵심: `[시작:끝]`에서 **끝 인덱스는 포함되지 않습니다**. `s[1:3]`은 인덱스 1, 2만 포함합니다.

우리 프로젝트에서:

```python
# ass_parser.py — 소수점 이하를 센티초로 변환
sec_parts = s.split('.')
if len(sec_parts) == 2:
    cs_str = sec_parts[1]
    cs = int(cs_str[:2])  # 앞 2자리만 사용 (예: "64" → 64)
```

## f-string (포맷 문자열)

`f"..."` 형태로 문자열 안에 변수를 삽입할 수 있습니다. 파이썬 3.6+에서 사용 가능합니다.

```python
name = "Python"
version = 3
print(f"Hello, {name} {version}!")  # "Hello, Python 3!"
```

우리 프로젝트에서 LRC 타임스탬프를 만들 때:

```python
# ass_parser.py
def format_lrc_time(tm, sec, cs):
    return f'[{tm:02d}:{sec:02d}.{cs:02d}]'
```

**`:02d`**는 "최소 2자리, 0으로 채움"이라는 포맷 지정자입니다:
- `f'{5:02d}'` → `"05"`
- `f'{15:02d}'` → `"15"`

이렇게 하면 `[00:15.96]` 같은 정확한 LRC 타임스탬프 형식이 만들어집니다.

에러 메시지에서도 많이 사용됩니다:

```python
# ass2lrc.py
print(f'Converted: {filepath} -> {output_path}')
print(f'Error: {e}')
```

## 문자열은 불변(Immutable)

문자열은 한번 만들어지면 **변경할 수 없습니다**. `.replace()`나 `.strip()` 같은 메서드는 원본을 바꾸지 않고, **새 문자열을 반환**합니다.

```python
text = "Hello World"
text.replace("World", "Python")
print(text)  # "Hello World" — 원본은 그대로!

text = text.replace("World", "Python")  # 새 값을 다시 저장해야 함
print(text)  # "Hello Python"
```

## 메서드 체이닝

메서드를 연달아 호출할 수 있습니다:

```python
line = "  [Events]  \n"
line.strip().startswith('[')  # True
# strip() 결과: "[Events]" → startswith('[') 확인
```

우리 프로젝트에서:

```python
# ass_parser.py — 줄바꿈 제거를 연달아 수행
lines = [l.rstrip('\n').rstrip('\r') for l in lines]
```

## 핵심 정리

| 메서드 | 용도 | 프로젝트에서 |
|--------|------|--------------|
| `.strip()` | 공백/줄바꿈 제거 | 파일 읽기 후 정리 |
| `.split()` | 구분자로 분리 | 시간 파싱, CSV 분리 |
| `.join()` | 리스트를 합침 | 대사 병합, 파일 저장 |
| `.startswith()` | 접두 확인 | LRC 라인 판별 |
| `.replace()` | 문자열 치환 | 태그 제거 |
| `f"..."` | 변수 삽입 | 타임스탬프, 메시지 |
| `[:]` 슬라이싱 | 부분 문자열 | 센티초 추출 |
| `r"..."` | 원시 문자열 | 정규표현식 패턴 |

## 직접 해보기

1. `"0:00:11.64".split(':')`의 결과를 확인해보세요.
2. f-string으로 `[MM:SS.cc]` 형식의 타임스탬프를 만들어보세요: `f'[{0:02d}:{15:02d}.{96:02d}]'`
3. `"  Hello  \n".strip()`과 `"  Hello  \n".rstrip()`의 차이를 확인해보세요.
