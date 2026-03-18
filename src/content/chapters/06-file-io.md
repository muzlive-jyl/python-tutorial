---
title: "파일 읽기와 쓰기"
description: "open, with, encoding — 파이썬으로 파일 다루기"
order: 6
part: 2
---

## 왜 파일 I/O가 중요한가요?

우리 프로젝트의 핵심은 **파일을 읽고 쓰는 것**입니다:
- ASS 파일을 **읽어서** 파싱
- LRC 파일을 **생성하여** 저장
- 검증/비교를 위해 두 파일을 **동시에 읽기**

파일 입출력(I/O)은 이 프로젝트를 이해하기 위한 필수 지식입니다.

## 파일 읽기

### 기본 방법 — `open()` + `with`

```python
with open('input.ass', encoding='utf-8') as f:
    content = f.read()  # 파일 전체를 하나의 문자열로
```

- `open()` — 파일을 여는 내장 함수
- `encoding='utf-8'` — 파일의 문자 인코딩 지정
- `with ... as f:` — 파일을 안전하게 열고 자동으로 닫아주는 구문
- `f.read()` — 파일 전체 내용을 문자열로 반환

### `with` 문이 필요한 이유

파일을 열면 반드시 닫아야 합니다. `with` 문을 사용하면 블록이 끝날 때 **자동으로 닫힙니다**.

```python
# ❌ 위험한 방법 — 에러 발생 시 파일이 닫히지 않을 수 있음
f = open('input.ass')
content = f.read()
f.close()

# ✅ 안전한 방법 — with 문 사용
with open('input.ass') as f:
    content = f.read()
# 여기서 f는 자동으로 닫힘
```

> 비유하면: `with`는 **자동문**과 같습니다. 들어가면(열면) 나올 때(블록이 끝나면) 자동으로 닫힙니다.

### `readlines()` — 줄 단위로 읽기

```python
with open('input.ass', encoding='utf-8') as f:
    lines = f.readlines()  # 각 줄을 리스트의 원소로
```

우리 프로젝트에서:

```python
# ass_parser.py — ASS 파일을 줄 단위로 읽기
with open(filepath, encoding='utf-8-sig') as f:
    lines = f.readlines()
lines = [l.rstrip('\n').rstrip('\r') for l in lines]
```

**`utf-8-sig`**는 BOM(Byte Order Mark)이 있는 UTF-8 파일도 올바르게 읽어주는 인코딩입니다. 일부 윈도우 프로그램이 파일 앞에 BOM을 추가하기 때문에 이를 처리합니다.

## 파일 쓰기

### `'w'` 모드로 쓰기

```python
with open('output.lrc', 'w', encoding='utf-8') as f:
    f.write('[title:여행의 노래]\n')
    f.write('[00:15.96]여행을 떠나요\n')
```

- `'w'` — 쓰기 모드 (파일이 있으면 덮어씀)

우리 프로젝트에서:

```python
# ass2lrc.py — LRC 파일 저장
with open(output_path, 'w', encoding='utf-8') as f:
    f.write('\n'.join(lrc_lines))
```

리스트의 모든 줄을 `'\n'`(줄바꿈)으로 합쳐서 한 번에 씁니다.

### 파일 모드 정리

| 모드 | 의미 | 설명 |
|------|------|------|
| `'r'` | read | 읽기 (기본값) |
| `'w'` | write | 쓰기 (덮어씀) |
| `'a'` | append | 이어쓰기 |
| `'x'` | exclusive | 새 파일만 (파일 있으면 에러) |

## 인코딩(Encoding)

텍스트 파일은 바이트로 저장됩니다. **인코딩**은 "문자를 바이트로 변환하는 규칙"입니다.

```python
# UTF-8: 가장 일반적인 인코딩
with open('file.txt', encoding='utf-8') as f:
    content = f.read()

# UTF-8-sig: BOM이 있는 UTF-8 (윈도우 호환)
with open('file.ass', encoding='utf-8-sig') as f:
    content = f.read()
```

> 인코딩을 지정하지 않으면 OS 기본 인코딩이 사용됩니다. 한글이 깨지는 문제를 방지하려면 항상 `encoding='utf-8'`을 명시하는 것이 좋습니다.

## 경로 처리 — `os.path`

파일을 다룰 때 경로 처리도 함께 알아야 합니다.

```python
import os

# 확장자 변경: input.ass → input.lrc
base, ext = os.path.splitext('input/song.ass')
# base = 'input/song', ext = '.ass'
output = base + '.lrc'  # 'input/song.lrc'

# 경로 합치기
path = os.path.join('input', 'song.ass')  # 'input/song.ass'

# 파일 존재 확인
os.path.exists('input/song.ass')  # True or False
```

## 임시 파일 — `tempfile`

테스트에서 임시 파일을 만들어 사용합니다:

```python
import tempfile
import os

# 임시 파일 생성
f = tempfile.NamedTemporaryFile(
    mode='w',           # 쓰기 모드
    suffix='.ass',      # 확장자
    delete=False,       # 자동 삭제 안 함
    encoding='utf-8'
)
f.write('테스트 내용')
f.close()

path = f.name  # 임시 파일의 경로

# 작업 후 삭제
os.unlink(path)
```

우리 프로젝트의 테스트 파일에서:

```python
# tests/test_ass2lrc.py
def _write_ass(self, content):
    """테스트용 임시 ASS 파일을 생성하는 헬퍼 메서드"""
    f = tempfile.NamedTemporaryFile(
        mode='w', suffix='.ass', delete=False, encoding='utf-8')
    f.write(content)
    f.close()
    return f.name
```

## 실전 예시: ASS 파일 읽기 흐름

우리 프로젝트에서 ASS 파일을 읽는 전체 흐름을 따라가 봅시다:

```python
# ass_parser.py — parse_ass_events 함수의 핵심 흐름

# 1. 파일 열기
with open(filepath, encoding='utf-8-sig') as f:
    lines = f.readlines()

# 2. 줄바꿈 제거
lines = [l.rstrip('\n').rstrip('\r') for l in lines]

# 3. [Events] 섹션 찾기
in_events = False
for line in lines:
    if line.strip() == '[Events]':
        in_events = True
        continue
    if in_events and line.startswith('Format:'):
        # Format 줄에서 필드 이름 추출
        format_fields = [f.strip() for f in line[len('Format:'):].split(',')]
    if in_events and line.startswith('Dialogue:'):
        # Dialogue 줄에서 데이터 추출
        parts = line[len('Dialogue:'):].split(',', len(format_fields) - 1)
        field_map = dict(zip(format_fields, [p.strip() for p in parts]))
```

이 코드에서 지금까지 배운 것들이 모두 사용됩니다:
- `open()` + `with` (파일 I/O)
- `.readlines()`, `.rstrip()`, `.strip()` (문자열 메서드)
- 리스트 컴프리헨션 (Chapter 7에서 자세히)
- `.startswith()` (문자열 확인)
- `.split()`, `dict()`, `zip()` (데이터 변환)

## 핵심 정리

| 패턴 | 용도 | 예시 |
|------|------|------|
| `with open() as f:` | 안전하게 파일 열기 | 자동으로 닫힘 |
| `f.read()` | 전체 읽기 | 하나의 문자열 |
| `f.readlines()` | 줄 단위 읽기 | 문자열 리스트 |
| `f.write()` | 파일 쓰기 | `'w'` 모드 필요 |
| `encoding='utf-8'` | 인코딩 지정 | 한글 깨짐 방지 |
| `encoding='utf-8-sig'` | BOM 처리 | 윈도우 호환 |
| `os.path.splitext()` | 확장자 분리 | `.ass` → `.lrc` |
| `tempfile` | 임시 파일 | 테스트용 |

## 직접 해보기

1. 아무 텍스트 파일을 만들고, 파이썬으로 읽어서 출력해보세요.
2. 리스트를 `'\n'.join()`으로 합쳐서 파일에 저장해보세요.
3. `os.path.splitext()`으로 파일의 확장자를 바꿔보세요.
