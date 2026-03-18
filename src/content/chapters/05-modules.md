---
title: "모듈과 import"
description: "파일 간의 의존관계 — import, from...import, 표준 라이브러리"
order: 5
part: 1
---

## 모듈이란?

하나의 `.py` 파일이 곧 하나의 **모듈**입니다. 다른 파일에서 `import`로 불러와 사용할 수 있습니다.

> 비유하면: 모듈은 **도구 상자**입니다. `ass_parser.py`는 "ASS 파싱 도구 상자"이고, 여기에 `strip_ass_tags`, `parse_ass_time` 같은 도구(함수)가 들어있습니다. 필요한 도구만 꺼내 쓸 수 있습니다.

## 우리 프로젝트의 파일 간 관계

```
ass_parser.py (공유 모듈)
    ↑ import        ↑ import
ass2lrc.py       verify_lrc.py
(변환기)          (검증기)
```

`ass_parser.py`에 공통 함수들을 모아두고, `ass2lrc.py`와 `verify_lrc.py`에서 가져다 씁니다. 이렇게 하면 같은 코드를 중복으로 작성하지 않아도 됩니다.

## import 방법

### 1. 모듈 전체 가져오기

```python
import re
import sys
import os
```

사용할 때 `모듈이름.함수이름()`으로 호출합니다:

```python
import re
pattern = re.compile(r'\{[^}]*\}')  # re 모듈의 compile 함수
```

### 2. 특정 함수만 가져오기 — `from ... import`

```python
from ass_parser import strip_ass_tags, parse_ass_time, format_lrc_time
```

이렇게 하면 모듈 이름 없이 바로 함수를 호출할 수 있습니다:

```python
# "ass_parser." 없이 바로 사용
clean = strip_ass_tags("{\K432}여행을 떠나요")
```

### 3. 여러 함수를 괄호로 묶어서 가져오기

```python
# ass_parser.py의 test 파일에서
from ass_parser import (
    strip_ass_tags,
    parse_ass_time,
    format_lrc_time,
    parse_ass_events,
    extract_karaoke_preroll,
    add_centiseconds
)
```

괄호를 사용하면 여러 줄에 걸쳐 깔끔하게 나열할 수 있습니다.

## 우리 프로젝트의 실제 import 구조

### ass2lrc.py

```python
import sys
import os
from ass_parser import format_lrc_time, parse_ass_events, add_centiseconds
```

- `sys` — 명령줄 인자(`sys.argv`), 프로그램 종료(`sys.exit`)
- `os` — 파일 경로 처리(`os.path.splitext`)
- `ass_parser` — 우리가 만든 공유 모듈의 함수들

### verify_lrc.py

```python
import re
import sys
import difflib
from ass_parser import strip_ass_tags, parse_ass_events
```

- `re` — 정규표현식
- `difflib` — 텍스트 비교 (diff 생성)
- `ass_parser` — 공유 모듈

### validate_lrc.py

```python
import re
import sys
```

이 파일은 `ass_parser`를 사용하지 않습니다. LRC 파일만 검증하므로 ASS 파싱이 필요 없기 때문입니다.

## 표준 라이브러리

파이썬에는 설치 없이 바로 사용할 수 있는 **표준 라이브러리**가 포함되어 있습니다. 우리 프로젝트에서 사용하는 표준 라이브러리:

| 모듈 | 용도 | 사용처 |
|------|------|--------|
| `re` | 정규표현식 | 태그 제거, 패턴 매칭 |
| `sys` | 시스템 기능 | 명령줄 인자, 종료 코드 |
| `os` | 파일 시스템 | 경로 조작 |
| `difflib` | 텍스트 비교 | ASS-LRC 내용 검증 |
| `tempfile` | 임시 파일 | 테스트에서 임시 파일 생성 |
| `unittest` | 테스트 프레임워크 | 단위 테스트 |

이 프로젝트의 특징 중 하나는 **외부 패키지를 전혀 사용하지 않는다**는 것입니다. 표준 라이브러리만으로 모든 기능을 구현했습니다. `pip install` 없이 바로 실행할 수 있습니다.

## `sys.argv` — 명령줄 인자

`sys` 모듈의 `argv`는 터미널에서 프로그램에 전달된 인자를 담고 있습니다.

```
$ python ass2lrc.py input.ass output.lrc
```

이 경우:
- `sys.argv[0]` = `"ass2lrc.py"` (스크립트 이름)
- `sys.argv[1]` = `"input.ass"` (첫 번째 인자)
- `sys.argv[2]` = `"output.lrc"` (두 번째 인자)
- `len(sys.argv)` = `3`

우리 프로젝트에서:

```python
# ass2lrc.py
if len(sys.argv) < 2:
    print(f'Usage: python {sys.argv[0]} input.ass [output.lrc]')
    sys.exit(1)

input_file = sys.argv[1]
output_file = sys.argv[2] if len(sys.argv) > 2 else None
```

## `os.path` — 경로 처리

```python
import os

# 파일명과 확장자 분리
os.path.splitext("input.ass")      # ("input", ".ass")

# 경로 합치기
os.path.join("input", "song.ass")  # "input/song.ass"

# 현재 파일의 디렉토리
os.path.dirname(__file__)          # 현재 파일이 있는 폴더 경로
```

우리 프로젝트에서 출력 파일 경로를 자동 생성할 때:

```python
# ass2lrc.py
base, _ = os.path.splitext(filepath)  # "input/song"
output_path = base + '.lrc'            # "input/song.lrc"
```

`_`(밑줄)은 "이 값은 사용하지 않겠다"는 관례적 표현입니다.

## `sys.path`와 테스트 파일의 import

테스트 파일은 `tests/` 폴더에 있지만, 테스트 대상 모듈은 상위 폴더에 있습니다. 이 경우 파이썬이 모듈을 찾을 수 있도록 경로를 추가해야 합니다:

```python
# tests/test_ass_parser.py
import sys
import os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), '..'))
```

이 코드는 "현재 파일의 상위 디렉토리를 모듈 검색 경로에 추가"합니다.

```
tests/test_ass_parser.py  ← 여기서
    ↑ sys.path.insert(0, '..')
ass_parser.py             ← 이걸 import 가능하게 됨
```

## `__name__`과 `'__main__'` 다시 보기

Chapter 1에서 잠깐 본 이 패턴을 이제 완전히 이해할 수 있습니다:

```python
# ass2lrc.py
def convert_ass_to_lrc(filepath, output_path=None):
    # 변환 로직...
    pass

if __name__ == '__main__':
    input_file = sys.argv[1]
    convert_ass_to_lrc(input_file)
```

- `python ass2lrc.py input.ass` → `__name__`이 `'__main__'` → 변환 실행
- `from ass2lrc import convert_ass_to_lrc` → `__name__`이 `'ass2lrc'` → 함수만 가져오고, 실행 코드는 무시

이 덕분에 `ass2lrc.py`는:
1. **터미널에서 직접 실행**하는 스크립트로도 쓸 수 있고
2. **다른 파일에서 import**하는 모듈로도 쓸 수 있습니다

## 핵심 정리

| 패턴 | 용도 | 예시 |
|------|------|------|
| `import 모듈` | 모듈 전체 가져오기 | `import re` |
| `from 모듈 import 함수` | 특정 함수만 가져오기 | `from ass_parser import strip_ass_tags` |
| `sys.argv` | 명령줄 인자 | `sys.argv[1]` → 첫 번째 인자 |
| `os.path` | 경로 처리 | `os.path.splitext("a.lrc")` |
| `sys.path.insert()` | 모듈 검색 경로 추가 | 테스트 파일에서 상위 모듈 접근 |
| `__name__ == '__main__'` | 직접 실행 시에만 동작 | 스크립트/모듈 겸용 |

## 직접 해보기

1. 두 개의 파일을 만들어보세요:
   - `utils.py`: `def hello(name): return f"Hi, {name}!"`
   - `main.py`: `from utils import hello` 후 `print(hello("Python"))`
2. `main.py`를 실행해서 `utils.py`의 함수가 잘 불려지는지 확인하세요.
3. `sys.argv`를 출력하는 스크립트를 만들어보세요: `import sys; print(sys.argv)`
