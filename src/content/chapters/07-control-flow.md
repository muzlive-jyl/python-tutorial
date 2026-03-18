---
title: "조건문과 반복문"
description: "if/for/while, 컴프리헨션 — 프로그램의 흐름 제어"
order: 7
part: 2
---

## 조건문 — if / elif / else

조건에 따라 다른 코드를 실행합니다.

```python
status = 'FAIL'

if status == 'PASS':
    print("통과!")
elif status == 'FAIL':
    print("실패...")
else:
    print("알 수 없음")
```

파이썬은 **들여쓰기(indentation)**로 코드 블록을 구분합니다. 같은 레벨의 들여쓰기가 같은 블록입니다.

### 비교 연산자

| 연산자 | 의미 | 예시 |
|--------|------|------|
| `==` | 같다 | `status == 'FAIL'` |
| `!=` | 다르다 | `status != 'PASS'` |
| `<`, `>` | 작다, 크다 | `len(lines) > 0` |
| `<=`, `>=` | 이하, 이상 | `i >= 1` |
| `in` | 포함 | `'title' in metadata` |
| `not in` | 미포함 | `key not in time_groups` |
| `is` | 동일 객체 | `output_path is None` |

### 논리 연산자

```python
if status == 'FAIL' and detail:
    print(f"실패: {detail}")

if not clean_text.strip():
    continue  # 빈 텍스트면 건너뛰기
```

우리 프로젝트에서:

```python
# ass2lrc.py — 인자 개수 확인
if len(sys.argv) < 2:
    print(f'Usage: python {sys.argv[0]} input.ass [output.lrc]')
    sys.exit(1)

# 삼항 연산자 (한 줄 if-else)
output_file = sys.argv[2] if len(sys.argv) > 2 else None
```

**삼항 연산자**: `값A if 조건 else 값B` — 조건이 참이면 값A, 거짓이면 값B.

## for 반복문

리스트나 다른 반복 가능한(iterable) 객체의 원소를 하나씩 처리합니다.

```python
lines = ["가사1", "가사2", "가사3"]
for line in lines:
    print(line)
```

### `range()` — 숫자 범위 반복

```python
for i in range(5):
    print(i)  # 0, 1, 2, 3, 4

for i in range(1, 4):
    print(i)  # 1, 2, 3
```

우리 프로젝트에서 순차 비교할 때:

```python
# validate_lrc.py — 타임스탬프 순서 검사
for i in range(1, len(ts_values)):
    if ts_values[i] < ts_values[i - 1]:
        out_of_order.append(i)
```

### `enumerate()` — 인덱스와 함께 반복

```python
for i, line in enumerate(lines):
    print(f"줄 {i}: {line}")
```

### `for`와 `continue`, `break`

```python
for line in lines:
    if not line.strip():
        continue  # 빈 줄은 건너뛰기
    if line.startswith('[Events]'):
        break     # [Events] 찾으면 반복 중단
    process(line)
```

- `continue` — 현재 반복을 건너뛰고 다음으로
- `break` — 반복 전체를 중단

우리 프로젝트에서:

```python
# ass_parser.py — [Events] 섹션을 찾는 루프
for line in lines:
    stripped = line.strip()
    if stripped.startswith('[') and stripped != '[Events]':
        if in_events:
            break  # 다른 섹션 시작 → Events 끝
    if stripped == '[Events]':
        in_events = True
        continue   # [Events] 헤더 자체는 건너뜀
```

## while 반복문

조건이 참인 동안 계속 반복합니다.

```python
count = 0
while count < 3:
    print(count)
    count += 1
# 0, 1, 2
```

우리 프로젝트에서:

```python
# ass_parser.py — 뒤쪽의 빈 줄 제거
while lines and not lines[-1].strip():
    lines.pop()
```

`lines`가 비어있지 않고(`lines`), 마지막 줄이 공백이면(`not lines[-1].strip()`), 마지막 줄을 제거합니다(`.pop()`).

## 리스트 컴프리헨션

`for` 문으로 리스트를 만드는 **간결한 문법**입니다.

### 기본 형태

```python
# 일반 for 문
result = []
for l in lines:
    result.append(l.strip())

# 리스트 컴프리헨션 (같은 결과, 한 줄)
result = [l.strip() for l in lines]
```

### 조건 포함

```python
# if 조건 추가
timestamps = [l for l in lines if l.startswith('[0')]
```

우리 프로젝트에서 많이 사용됩니다:

```python
# ass_parser.py — 줄바꿈 제거
lines = [l.rstrip('\n').rstrip('\r') for l in lines]

# ass_parser.py — Format 필드 추출
format_fields = [f.strip() for f in line[len('Format:'):].split(',')]

# verify_lrc.py — 텍스트만 추출
texts = [text for _, _, text, _ in dialogues]

# validate_lrc.py — 타임스탬프 튜플 추출
ts_tuples = [(mm, ss, cc) for mm, ss, cc, _ in timestamps]
```

### 셋 컴프리헨션

리스트 대신 셋을 만들 수도 있습니다:

```python
# validate_lrc.py — 발견된 헤더 태그를 셋으로
found_tags = {h.split(']')[0] + ']' for h in headers_found}
```

`[]` 대신 `{}`를 사용하면 셋 컴프리헨션이 됩니다.

### 제너레이터 표현식

`()`를 사용하면 메모리를 절약하는 제너레이터가 됩니다. `sum()` 같은 함수와 함께 쓸 때 유용합니다:

```python
# verify_lrc.py — 모든 텍스트의 총 길이
total = sum(len(t) for t in ass_texts)
```

## 딕셔너리와 반복

### `.items()` — 키-값 함께 반복

```python
metadata = {"title": "여행의 노래", "artist": "홍길동"}
for key, value in metadata.items():
    print(f"{key}: {value}")
```

### 딕셔너리 그룹화 패턴

우리 프로젝트에서 가장 중요한 패턴 중 하나입니다:

```python
# ass2lrc.py — 같은 타임스탬프의 대사를 그룹화
time_groups = {}
for time_key, style, text, preroll_cs in sorted(dialogues, key=lambda x: x[0]):
    if time_key not in time_groups:
        time_groups[time_key] = []
    time_groups[time_key].append(text)
```

이 패턴을 단계별로 보면:
1. 빈 딕셔너리 생성
2. 정렬된 데이터를 순회
3. 키가 없으면 빈 리스트 생성
4. 값을 리스트에 추가

### 중복 감지 패턴

```python
# validate_lrc.py — 타임스탬프 중복 검사
seen = {}
for ts_str, i in timestamps:
    if ts_str in seen:
        duplicates.append(f"줄 {i}와 {seen[ts_str]}이 동일")
    else:
        seen[ts_str] = i
```

## 핵심 정리

| 구문 | 용도 | 프로젝트에서 |
|------|------|--------------|
| `if/elif/else` | 조건 분기 | 인자 검사, 에러 처리 |
| `for ... in` | 반복 | 줄 순회, 대사 처리 |
| `while` | 조건 반복 | 빈 줄 제거 |
| `continue` | 건너뛰기 | 빈 텍스트 무시 |
| `break` | 중단 | 섹션 경계에서 멈춤 |
| `[x for x in ...]` | 리스트 컴프리헨션 | 데이터 변환, 필터링 |
| `A if 조건 else B` | 삼항 연산자 | 기본값 처리 |

## 직접 해보기

1. 1부터 10까지의 짝수만 출력하는 리스트 컴프리헨션을 작성해보세요: `[x for x in range(1, 11) if x % 2 == 0]`
2. 딕셔너리 그룹화 패턴을 직접 만들어보세요: 과일 리스트를 첫 글자별로 그룹화
3. `enumerate()`를 사용해서 리스트의 인덱스와 값을 함께 출력해보세요.
