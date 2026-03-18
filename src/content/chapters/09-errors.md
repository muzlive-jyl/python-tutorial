---
title: "에러 처리"
description: "try/except, raise, sys.exit — 예외 상황을 다루는 방법"
order: 9
part: 2
---

## 에러가 발생하면?

프로그램 실행 중 문제가 생기면 파이썬은 **예외(Exception)**를 발생시키고 프로그램이 멈춥니다.

```python
int("abc")  # ValueError: invalid literal for int()
open("없는파일.txt")  # FileNotFoundError
```

우리 프로젝트는 파일을 다루므로, "파일이 없다", "형식이 잘못됐다" 같은 에러가 발생할 수 있습니다. 이런 상황을 **우아하게(gracefully)** 처리해야 합니다.

## try / except — 에러 잡기

```python
try:
    result = int("abc")
except ValueError:
    print("숫자로 변환할 수 없습니다")
```

- `try` 블록: 에러가 발생할 수 있는 코드
- `except` 블록: 에러가 발생했을 때 실행되는 코드

### 에러 정보 가져오기 — `as e`

```python
try:
    open("없는파일.txt")
except FileNotFoundError as e:
    print(f"에러: {e}")
    # 에러: [Errno 2] No such file or directory: '없는파일.txt'
```

`as e`로 에러 객체를 변수에 저장하면 에러 메시지를 확인할 수 있습니다.

### 여러 에러 처리

```python
try:
    convert_ass_to_lrc(input_file, output_file)
except FileNotFoundError as e:
    print(f'파일을 찾을 수 없습니다: {e}')
    sys.exit(1)
except ValueError as e:
    print(f'형식 오류: {e}')
    sys.exit(1)
```

우리 프로젝트의 `ass2lrc.py`에서:

```python
# ass2lrc.py — 메인 실행부
if __name__ == '__main__':
    input_file = sys.argv[1]
    output_file = sys.argv[2] if len(sys.argv) > 2 else None
    try:
        convert_ass_to_lrc(input_file, output_file)
    except FileNotFoundError as e:
        print(f'Error: {e}')
        sys.exit(1)
    except ValueError as e:
        print(f'Error: {e}')
        sys.exit(1)
```

### try / except / finally

`finally` 블록은 에러 발생 여부와 관계없이 **항상** 실행됩니다.

```python
try:
    f = open("data.txt")
    data = f.read()
except FileNotFoundError:
    print("파일 없음")
finally:
    f.close()  # 에러가 나든 안 나든 파일을 닫음
```

우리 프로젝트의 테스트에서:

```python
# tests/test_ass2lrc.py — 임시 파일을 반드시 삭제
def test_basic_conversion(self):
    path = self._write_ass(content)
    try:
        result = convert_ass_to_lrc(path)
        self.assertTrue(os.path.exists(result))
    finally:
        os.unlink(path)  # 테스트 성공/실패 관계없이 임시 파일 삭제
```

## raise — 에러 발생시키기

직접 에러를 발생시킬 수 있습니다.

```python
raise ValueError("잘못된 값입니다")
raise FileNotFoundError("파일을 찾을 수 없습니다")
```

우리 프로젝트에서:

```python
# ass_parser.py — [Events] 섹션이 없으면 에러
if not in_events:
    raise ValueError(f'No [Events] section found in {filepath}')

# ass_parser.py — Format 필드가 없으면 에러
if format_fields is None:
    raise ValueError(f'No Format line found in [Events] section of {filepath}')
```

왜 `raise`를 사용할까요? 잘못된 입력을 받았을 때, **문제를 감추지 않고 명확히 알려주기** 위해서입니다. 에러 메시지에 구체적인 정보(파일명 등)를 포함하면 디버깅이 쉬워집니다.

## `sys.exit()` — 프로그램 종료

```python
import sys

sys.exit(0)   # 정상 종료 (exit code 0)
sys.exit(1)   # 에러 종료 (exit code 1)
```

- **exit code 0**: "문제없이 끝났습니다"
- **exit code 1 (또는 0이 아닌 수)**: "문제가 있었습니다"

우리 프로젝트에서의 패턴:

```python
# 인자 부족 → 사용법 출력 후 종료
if len(sys.argv) < 2:
    print(f'Usage: python {sys.argv[0]} input.ass [output.lrc]')
    sys.exit(1)

# 변환 성공
sys.exit(0)
```

이 exit code는 다른 프로그램(셸 스크립트, CI/CD 등)에서 성공/실패를 판단하는 데 사용됩니다.

## 자주 만나는 에러 타입

| 에러 | 원인 | 프로젝트에서 |
|------|------|--------------|
| `FileNotFoundError` | 파일이 없음 | 잘못된 입력 파일 경로 |
| `ValueError` | 값이 올바르지 않음 | [Events] 섹션 없음 |
| `TypeError` | 타입이 맞지 않음 | 함수에 잘못된 인자 |
| `IndexError` | 인덱스 범위 초과 | 리스트가 비어있을 때 `[0]` 접근 |
| `KeyError` | 딕셔너리에 키 없음 | 없는 필드 접근 |

## 가드 패턴 (Guard Clause)

에러를 잡는 것보다 **미리 방지**하는 것이 더 좋습니다.

```python
# ❌ 나중에 에러 발생
def process(text):
    result = text.strip()  # text가 None이면 에러!
    return result

# ✅ 미리 확인 (가드 절)
def process(text):
    if not text:
        return ""
    return text.strip()
```

우리 프로젝트에서:

```python
# ass_parser.py — 빈 텍스트는 건너뛰기
if not clean_text.strip():
    continue

# ass2lrc.py — format_fields가 None이면 에러
if format_fields is None:
    raise ValueError(...)
```

## 에러 처리 흐름 전체 그림

우리 프로젝트의 에러 처리 흐름:

```
사용자 입력 (터미널)
    │
    ▼
인자 검사 (len(sys.argv) < 2?)
    │ 실패 → print(사용법) + sys.exit(1)
    ▼
파일 읽기 (open)
    │ 실패 → FileNotFoundError → except에서 잡음 → sys.exit(1)
    ▼
형식 검사 ([Events] 있나?)
    │ 실패 → raise ValueError → except에서 잡음 → sys.exit(1)
    ▼
변환 수행
    │
    ▼
결과 저장 + sys.exit(0)
```

## `.get()` — KeyError 방지

딕셔너리에서 키가 없을 때 에러 대신 기본값을 반환하게 할 수 있습니다:

```python
metadata = {"title": "여행의 노래"}

metadata["artist"]          # KeyError!
metadata.get("artist", "")  # "" — 에러 없이 기본값 반환
```

## 핵심 정리

| 패턴 | 용도 | 예시 |
|------|------|------|
| `try/except` | 에러 잡기 | 파일 열기 실패 처리 |
| `except 에러 as e` | 에러 정보 접근 | `print(f'Error: {e}')` |
| `finally` | 항상 실행 | 임시 파일 정리 |
| `raise` | 에러 발생 | 잘못된 형식 알림 |
| `sys.exit(0/1)` | 프로그램 종료 | 성공/실패 코드 |
| `.get(key, default)` | 안전한 딕셔너리 접근 | `KeyError` 방지 |

## 직접 해보기

1. `try/except`로 `int("abc")`의 `ValueError`를 잡아보세요.
2. `raise ValueError("테스트 에러")`를 실행하고, `try/except`로 잡아보세요.
3. `sys.argv`의 길이를 확인하고 인자가 부족하면 사용법을 출력하는 스크립트를 만들어보세요.
