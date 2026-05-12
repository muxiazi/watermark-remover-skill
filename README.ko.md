# Watermark Remover Skill (팀 버전 설치 가이드)

[🇨🇳 简体中文](./README.md) | [🇬🇧 English](./README.en.md) | [🇯🇵 日本語](./README.ja.md) | **🇰🇷 한국어**


Claude / Claude Code 등의 AI 에이전트가 「부이(Buyi) 이미지 일괄 워터마크 제거 소프트웨어」를 자동으로 호출하여 이미지를 처리할 수 있도록 합니다.
소프트웨어가 없다면 "布衣图片批量去水印软件"로 검색하여 받으세요.

---

## 작동 원리 (설치 전에 먼저 이해하세요)

```
[에이전트] ──이미지 투입──► [입력 디렉토리] ──감시로 자동 트리거──► [GUI 처리] ──쓰기──► [출력 디렉토리] ──에이전트 읽기
```

에이전트는 GUI 버튼을 직접 조작하지 않고, **GUI가 감시 중인 입력 디렉토리로 이미지를 복사**한 다음 **결과가 나타날 때까지 출력 디렉토리를 폴링**합니다.

따라서 GUI는 **항상 켜져 있어야** 하며, **「📡 디렉토리 감시」 버튼이 켜진 상태**여야 합니다.

---

## 설치 단계

### 멤버십 가입 주소: https://buyitanan.com/bu_yi_tu_pian_pi_liang_qu_shui_yin

### 1. GUI 프로그램 준비

> **참고:** GUI 자체는 현재 중국어만 지원합니다. 아래 한국어 버튼명은 참고용 번역이며, 앱 내 실제 라벨은 중국어(괄호 안 표시)입니다.

- 「부이 이미지 일괄 워터마크 제거 소프트웨어」(布衣图片批量去水印软件) 설치 및 실행
- 계정 로그인 (**무료 회원은 안 됩니다**, 무료 회원은 감시 권한이 없음)
- GUI에서:
  - 「입력 폴더 선택」(选择输入文件夹)을 클릭하여 디렉토리 선택 (전용 폴더를 새로 만들길 권장, 예: `D:\watermark_in`)
  - 「출력 폴더 선택」(选择输出文件夹)을 클릭하여 다른 디렉토리 선택 (예: `D:\watermark_out`)
  - 「📡 디렉토리 감시: 꺼짐」(📡 监控目录:关闭) 버튼을 클릭, 녹색의 「📡 디렉토리 감시: 켜짐」(📡 监控目录:开启)으로 바뀌어야 함
- 프로그램을 계속 실행 상태로 유지 (최소화는 가능하나 닫으면 안 됨)

### 2. 본 skill 설치

```bash
# 이 디렉토리를 팀원의 로컬 임의 위치로 복사
# 예: ~/skills/watermark-remover/
```

Python 3.8+ 만 필요합니다. **서드파티 의존성 없음**.

### 3. 설정 파일 생성

```bash
cp config.example.json config.json
```

`config.json`을 편집하고 `input_dir` / `output_dir`을 1단계에서 GUI에서 실제로 선택한 두 디렉토리로 변경합니다. **양쪽이 완전히 일치해야 합니다**.

### 4. 자가 점검 실행

```bash
python watermark_remover.py check
```

✅이 출력되면 입력/출력 디렉토리가 모두 인식된 것입니다.

테스트 이미지 한 장으로 실시간 검증을 추가로 하려면:

```bash
python watermark_remover.py check --sample test.png --timeout 60
```

60초 이내에 ✅이 나오면 에이전트 → GUI 전체 파이프라인이 연결되어 있음을 의미합니다.

---

## Claude 용 배포 방법

`watermark-remover-skill/` 디렉토리 전체를 Claude 프로젝트의 지식 베이스 / Skill 라이브러리에 업로드하면 됩니다. Claude는 `SKILL.md`의 frontmatter (`name` / `description`)를 읽어 사용자가 워터마크 제거 작업을 언급할 때 자동으로 skill을 호출합니다.

Claude Code 등의 CLI 에이전트라면 디렉토리를 에이전트가 접근 가능한 위치에 두고, 설정 파일 경로를 에이전트에게 알려주거나 환경 변수를 설정합니다:

```bash
export WATERMARK_REMOVER_CONFIG=~/skills/watermark-remover/config.json
```

---

## 자주 쓰는 명령 모음

```bash
# 자가 점검
python watermark_remover.py check
python watermark_remover.py check --sample test.png

# 한 장 처리
python watermark_remover.py process input.jpg

# 여러 장 처리
python watermark_remover.py process a.jpg b.png c.webp

# 디렉토리 전체 처리
python watermark_remover.py process ~/photos_to_clean/

# 대량 배치 + 사용자 정의 타임아웃 + 결과 내보내기
python watermark_remover.py process ~/big_batch/ --timeout 1800 --json-out result.json

# 작업 서브 디렉토리 사용 안 함 (입력 루트에 직접 배치)
python watermark_remover.py process input.jpg --no-subdir

# 복사 대신 이동 (원본 파일이 삭제됩니다!)
python watermark_remover.py process input.jpg --move
```

---

## 팀 협업 시 주의사항

여러 팀원이 동일한 GUI 인스턴스(같은 머신)를 공유할 수 있기 때문에 다음을 약속해 두세요:

1. **항상 작업 서브 디렉토리로 격리**: 기본적으로 매번 UUID 서브 디렉토리가 자동 생성되어 여러 사람의 동시 실행에서도 충돌이 없습니다. **임의로 `--no-subdir`를 사용하지 마세요**.
2. **서로의 영속화 기록을 지우지 않기**: GUI의 "처리 완료 목록 비우기"에서 "Yes to All"을 누르면 `processed_files.json`이 비워져, 다른 사람의 작업이 재처리될 수 있습니다. **사용 전 반드시 팀과 동기화하세요**.
3. **타임아웃은 충분하게**: 한 장은 보통 3~30초이지만, 대용량 이미지 배치 처리는 몇 분이 걸릴 수 있습니다. 에이전트가 pending을 보고하면 먼저 GUI가 아직 실행 중인지 확인한 후 포기할지 timeout을 연장할지 결정하세요.
4. **「완료 후 정리」 절차 수립을 권장**: 처리 완료 후 에이전트가 작업 서브 디렉토리를 입력 디렉토리에서 삭제하도록 하여 환경을 깨끗하게 유지할 수 있습니다.

---

## 파일 목록

```
watermark-remover-skill/
├── SKILL.md              # Claude/에이전트가 읽는 지시 파일 (frontmatter 수정 금지)
├── README.md             # 본 파일, 사람용
├── watermark_remover.py  # 핵심 스크립트 (CLI + 라이브러리 양용)
├── config.example.json   # 설정 파일 템플릿
└── config.json           # 실제 설정 (최초 설치 시 템플릿에서 복사하여 수정)
```

---

## 피드백과 확장

- "GUI 비의존" CLI 버전(소프트웨어 없이 알고리즘을 직접 호출)을 만들려면 원본 GUI 코드에서 `InpaintWorker`의 처리 로직을 추출해야 합니다. 본 skill에서는 아직 다루지 않습니다.
- Webhook / HTTP API 트리거를 추가하려면 `WatermarkRemoverClient` 클래스를 기반으로 직접 확장할 수 있습니다.
