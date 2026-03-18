---
title: "테스트 작성"
description: "unittest로 코드 검증하기 — 자동화된 품질 보증"
order: 10
part: 2
---

## 왜 테스트를 작성하나요?

코드를 수정할 때마다 직접 실행해서 결과를 확인하는 것은 번거롭고 빠뜨리기 쉽습니다. **테스트 코드**를 작성하면, 명령어 하나로 모든 기능이 올바르게 동작하는지 자동으로 확인할 수 있습니다.

> 비유하면: 테스트는 **자동 검수 장치**입니다. 공장에서 제품이 나올 때마다 사람이 하나씩 검사하는 대신, 자동 검수 라인이 불량품을 잡아냅니다.

## unittest — 파이썬 내장 테스트 프레임워크

```python
import unittest

class TestExample(unittest.TestCase):
    def test_addition(self):
        self.assertEqual(1 + 1, 2)

    def test_string(self):
        self.assertTrue("hello".startswith("he"))
```

실행:

```
$ python -m unittest test_example.py -v
test_addition (test_example.TestExample) ... ok
test_string (test_example.TestExample) ... ok

Ran 2 tests in 0.001s
OK
```

### 구조

- **`unittest.TestCase`** — 테스트 클래스의 부모 클래스 (상속)
- **`test_`로 시작하는 메서드** — 개별 테스트 케이스
- **`self.assert*()`** — 검증 메서드

## 우리 프로젝트의 테스트 구조

```
tests/
├── test_ass_parser.py     # ass_parser.py 테스트
├── test_ass2lrc.py        # ass2lrc.py 테스트
├── test_validate_lrc.py   # validate_lrc.py 테스트
└── test_verify_lrc.py     # verify_lrc.py 테스트
```

모든 테스트 실행:

```
$ python -m unittest discover -s tests -v
```

- `-m unittest` — unittest 모듈을 실행
- `discover` — 테스트 파일을 자동으로 찾기
- `-s tests` — `tests` 폴더에서 검색
- `-v` — 자세한 출력 (verbose)

## 주요 assert 메서드

| 메서드 | 검증 내용 | 예시 |
|--------|-----------|------|
| `assertEqual(a, b)` | `a == b` | `self.assertEqual(result, "expected")` |
| `assertTrue(x)` | `x`가 참 | `self.assertTrue(os.path.exists(path))` |
| `assertFalse(x)` | `x`가 거짓 | `self.assertFalse(has_error)` |
| `assertIn(a, b)` | `a in b` | `self.assertIn("title", output)` |
| `assertNotIn(a, b)` | `a not in b` | `self.assertNotIn("error", output)` |
| `assertRaises(E)` | 예외 `E` 발생 | 아래 예시 참고 |
| `assertGreater(a, b)` | `a > b` | `self.assertGreater(len(result), 0)` |

## 실전 예시: ass_parser 테스트

### 1. 단순 함수 테스트

```python
# tests/test_ass_parser.py

class TestStripAssTags(unittest.TestCase):
    def test_basic_tag_removal(self):
        """기본적인 태그 제거를 테스트"""
        result = strip_ass_tags(r'{\K432}여행을 {\K88}떠나요')
        self.assertEqual(result, '여행을 떠나요')

    def test_no_tags(self):
        """태그가 없는 경우"""
        result = strip_ass_tags('태그 없는 텍스트')
        self.assertEqual(result, '태그 없는 텍스트')

    def test_empty_string(self):
        """빈 문자열"""
        result = strip_ass_tags('')
        self.assertEqual(result, '')
```

테스트의 패턴: **정상 케이스, 경계 케이스, 빈 입력**을 모두 확인합니다.

### 2. 여러 반환값 테스트

```python
class TestParseAssTime(unittest.TestCase):
    def test_basic_time(self):
        tm, sec, cs = parse_ass_time('0:00:11.64')
        self.assertEqual(tm, 0)
        self.assertEqual(sec, 11)
        self.assertEqual(cs, 64)

    def test_with_hours(self):
        tm, sec, cs = parse_ass_time('1:30:05.20')
        self.assertEqual(tm, 90)  # 1시간 30분 = 90분
        self.assertEqual(sec, 5)
        self.assertEqual(cs, 20)
```

### 3. 예외 발생 테스트 — `assertRaises`

```python
class TestParseAssEvents(unittest.TestCase):
    def test_no_events_section(self):
        """[Events] 섹션이 없으면 ValueError가 발생해야 함"""
        path = self._write_ass('[Script Info]\nTitle: test\n')
        try:
            self.assertRaises(ValueError, parse_ass_events, path)
        finally:
            os.unlink(path)
```

`assertRaises(에러타입, 함수, 인자들)` — 이 함수를 호출했을 때 지정한 에러가 발생하면 테스트 통과.

## 헬퍼 메서드 패턴

테스트에서 반복되는 코드를 헬퍼 메서드로 만들어 재사용합니다:

```python
class TestAss2Lrc(unittest.TestCase):
    def _write_ass(self, content):
        """테스트용 임시 ASS 파일 생성"""
        f = tempfile.NamedTemporaryFile(
            mode='w', suffix='.ass', delete=False, encoding='utf-8')
        f.write(content)
        f.close()
        return f.name

    def _read(self, path):
        """파일 내용 읽기"""
        with open(path, encoding='utf-8') as f:
            return f.read()
```

이 헬퍼들을 사용해서 테스트를 간결하게 작성합니다:

```python
def test_conversion(self):
    ass_content = """[Events]
Format: Layer, Start, End, Style, Name, Text
Dialogue: 0,0:00:11.64,0:00:15.00,Default,,{\\K432}여행을 떠나요
"""
    path = self._write_ass(ass_content)
    try:
        output = convert_ass_to_lrc(path)
        lrc = self._read(output)
        self.assertIn('[00:15.96]', lrc)
    finally:
        os.unlink(path)
        if os.path.exists(output):
            os.unlink(output)
```

## 테스트 파일의 import 설정

테스트 파일은 `tests/` 폴더에 있지만, 테스트 대상은 상위 폴더에 있습니다:

```python
# tests/test_ass_parser.py — 파일 상단
import os
import sys
import unittest

# 상위 디렉토리를 모듈 경로에 추가
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..'))

from ass_parser import (
    strip_ass_tags,
    parse_ass_time,
    format_lrc_time,
    parse_ass_events,
    extract_karaoke_preroll,
    add_centiseconds
)
```

## 테스트 건너뛰기 — `skipTest`

특정 조건에서 테스트를 건너뛸 수 있습니다:

```python
def test_with_real_file(self):
    path = os.path.join(PROJECT_ROOT, 'input', 'example.ass')
    if not os.path.exists(path):
        self.skipTest('example.ass 파일 없음')
    # 파일이 있으면 테스트 진행
```

## 좋은 테스트의 원칙

### 1. 하나의 테스트는 하나만 검증

```python
# ✅ 좋음 — 각각 독립적으로 검증
def test_time_parsing(self):
    tm, sec, cs = parse_ass_time('0:00:11.64')
    self.assertEqual((tm, sec, cs), (0, 11, 64))

def test_time_formatting(self):
    result = format_lrc_time(0, 15, 96)
    self.assertEqual(result, '[00:15.96]')
```

### 2. 정리(cleanup)를 잊지 않기

```python
# ✅ finally로 임시 파일 정리
path = self._write_ass(content)
try:
    # 테스트 코드
finally:
    os.unlink(path)  # 항상 삭제
```

### 3. 경계 케이스 포함

```python
def test_zero_time(self):
    result = format_lrc_time(0, 0, 0)
    self.assertEqual(result, '[00:00.00]')

def test_overflow(self):
    # 99초 + 1초 = 1분 40초 (분 올림)
    result = add_centiseconds(0, 59, 99, 1)
    self.assertEqual(result, (1, 0, 0))
```

## 전체 프로젝트 지식 맵

이 챕터까지 배운 내용으로, 프로젝트의 모든 코드를 이해할 수 있습니다:

```
Chapter 1: .py 파일 실행, print, __name__
Chapter 2: 변수, 타입 (int, str, list, dict, tuple)
Chapter 3: 문자열 (strip, split, join, f-string, 슬라이싱)
Chapter 4: 함수 (def, return, 기본값, lambda, 내장 함수)
Chapter 5: 모듈 (import, sys, os, __name__)
Chapter 6: 파일 I/O (open, with, encoding)
Chapter 7: 흐름 제어 (if/for/while, 컴프리헨션)
Chapter 8: 정규표현식 (re 모듈)
Chapter 9: 에러 처리 (try/except, raise)
Chapter 10: 테스트 (unittest) ← 현재!
```

| 프로젝트 파일 | 관련 챕터 |
|---------------|-----------|
| `ass_parser.py` | 3, 4, 6, 7, 8 |
| `ass2lrc.py` | 2, 5, 6, 7, 9 |
| `validate_lrc.py` | 7, 8, 9 |
| `verify_lrc.py` | 3, 7, 8 |
| `tests/*.py` | 5, 10 |

## 핵심 정리

| 개념 | 설명 |
|------|------|
| `unittest.TestCase` | 테스트 클래스의 부모 |
| `test_`로 시작 | 테스트 메서드 이름 규칙 |
| `assertEqual` | 두 값이 같은지 확인 |
| `assertTrue` / `assertFalse` | 참/거짓 확인 |
| `assertRaises` | 예외 발생 확인 |
| `finally` | 정리 코드 (임시 파일 삭제) |
| `skipTest` | 조건부 테스트 건너뛰기 |
| `python -m unittest discover` | 모든 테스트 실행 |

## 직접 해보기

1. `format_lrc_time` 함수를 테스트하는 클래스를 만들어보세요. 정상 케이스와 경계 케이스(0, 큰 숫자)를 포함하세요.
2. `assertRaises`로 `int("abc")`가 `ValueError`를 발생시키는지 확인해보세요.
3. 프로젝트의 전체 테스트를 실행해보세요: `python -m unittest discover -s tests -v`
