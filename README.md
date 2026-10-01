# 6.1 Sol Orch

**필요한 작업만 에이전트에게 나누고, 최종 통합과 검증은 Sol이 책임지는 Codex 스킬입니다.**

[English](README.en.md) · [스킬 지침](skills/6-1-sol-orch/SKILL.md) · [Apache-2.0](LICENSE)

![Sol 6.1이 Luna 탐색자·조사자와 Sol 구현자에게 작업을 분담하고 결과를 통합·검증한 뒤, 필요할 때만 Astra 검토를 거쳐 최종 확인하는 흐름](assets/orchestration-flow.png)

분담이 필요한 경우의 권장 흐름입니다. 중간 열은 위에서부터 **탐색·조사·구현**, 결과가 모이는 Sol은 **통합·검증**, 맨 오른쪽 Sol은 **최종 확인**을 맡습니다. 점선은 필요한 경우의 Astra 검토, 아래쪽 우회 화살표는 검토를 생략하는 경로입니다.

*태양·달·별은 모델을 구분하기 위해 직접 만든 설명용 심볼이며, 공식 로고가 아닙니다.*

복잡한 작업에서는 조사·탐색·구현을 분담하고, 작은 작업은 메인 에이전트가 직접 처리합니다. 모든 역할을 항상 실행하지 않으며, 독립 검토는 구체적인 위험이나 검증 공백이 있을 때 추가합니다.

이 저장소는 `SKILL.md`와 UI 메타데이터를 배포합니다. 실행 프로그램이나 모델 설정을 포함하지 않습니다. 모델 사용 가능 여부와 에이전트 호출 방식은 사용하는 Codex 환경에 따라 달라집니다.

## 역할 구조

| 역할 | 목표 모델 ID | 추론 수준 | 맡는 일 |
| --- | --- | --- | --- |
| 메인 / 조정자 | `gpt-6.1-sol` | medium | 목표, 범위, 의사결정, 통합, 최종 검증 |
| 탐색자 | `gpt-6-luna` | high | 코드 경로와 의존성 조사, 근거 보고 |
| 조사자 | `gpt-6-luna` | high | 현재 정보와 1차 출처 조사 |
| 구현자 | `gpt-6.1-sol` | high | 지정된 파일 수정과 관련 테스트 |
| 독립 검토자 | `gpt-6-astra` | low | 필요한 경우 완성된 변경의 중요한 결함 검토 |

위 ID는 이 스킬이 목표로 하는 환경의 값입니다. **실행 전에 자신의 환경에서 사용할 수 있는 모델 ID와 추론 수준을 확인하세요.** 같은 ID가 모든 Codex 클라이언트·계정에서 제공된다고 보장하지 않습니다. 지원되지 않는 모델을 사용한 것처럼 보고하지 않고, 대체 모델을 사용했다면 실제 모델을 명시합니다.

## 설치

### Codex에게 설치 요청하기

Codex의 GitHub 스킬 설치 기능이 있는 환경에서는 다음 내용을 보내세요.

```text
다음 GitHub 경로의 Codex 스킬을 설치해줘.
https://github.com/Kimhyuntae9665/codex-skill-6-1-sol-orch/tree/main/skills/6-1-sol-orch
```

### 직접 설치하기

Git이 필요합니다. 현재 공식 문서의 사용자 스킬 경로인 `~/.agents/skills`에 복사합니다. 명령은 저장소를 아직 복제하지 않은 디렉터리에서 실행하세요.

**Windows PowerShell**

```powershell
git clone https://github.com/Kimhyuntae9665/codex-skill-6-1-sol-orch.git
if ($LASTEXITCODE -ne 0) { throw 'Git clone failed.' }
$skillTarget = Join-Path $HOME '.agents/skills/6-1-sol-orch'
if (Test-Path -LiteralPath $skillTarget) { throw 'Skill already exists. Back it up before updating.' }
New-Item -ItemType Directory -Path (Split-Path $skillTarget) -Force | Out-Null
Copy-Item -LiteralPath './codex-skill-6-1-sol-orch/skills/6-1-sol-orch' -Destination $skillTarget -Recurse
```

**macOS / Linux**

```sh
(
  set -eu
  git clone https://github.com/Kimhyuntae9665/codex-skill-6-1-sol-orch.git
  skill_target="$HOME/.agents/skills/6-1-sol-orch"
  if [ -e "$skill_target" ]; then
    echo 'Skill already exists. Back it up before updating.' >&2
    exit 1
  fi
  mkdir -p "$HOME/.agents/skills"
  cp -R ./codex-skill-6-1-sol-orch/skills/6-1-sol-orch "$skill_target"
)
```

클라이언트의 스킬 목록에서 `6-1-sol-orch`를 확인하세요. 보이지 않으면 새 대화를 열거나 클라이언트를 다시 시작하세요. 일부 환경의 기본 GitHub 설치기는 `$CODEX_HOME/skills` 또는 `~/.codex/skills`를 사용합니다. 해당 환경의 실제 검색 경로를 확인하고, 같은 스킬을 여러 검색 경로에 중복 설치하지 마세요.

프로젝트에서만 쓰려면 사용자 경로 대신 프로젝트의 `.agents/skills/6-1-sol-orch`에 같은 폴더를 복사하세요. 설치는 스킬 폴더 복사로 끝나며, 기존 `config.toml`이나 `AGENTS.md`를 수정할 필요가 없습니다.

## 사용하기

1. Codex에서 메인 모델과 추론 수준을 선택합니다. 목표 구성은 `gpt-6.1-sol` / medium입니다.
2. 하위 에이전트 기능과 원하는 하위 모델이 현재 환경에서 제공되는지 확인합니다.
3. `$6-1-sol-orch`와 함께 수행할 작업을 구체적으로 적습니다.

**스킬 호출은 이미 실행 중인 메인 모델을 바꾸지 않습니다.** 다른 모델로 시작했다면 이 스킬은 그 사실을 밝혀야 합니다. 이 저장소는 모델별 에이전트 설정 파일을 자동 설치하지 않습니다. 모델 고정 방법은 환경의 호출 도구와 [공식 하위 에이전트 문서](https://developers.openai.com/codex/multi-agent)를 확인하세요.

### 코드 변경

```text
$6-1-sol-orch
주문 CSV 내보내기에 날짜 필터를 추가해줘.
탐색자는 현재 API와 테스트 경로를 확인하고,
구현자는 관련 파일만 수정해줘.
메인 에이전트는 결과를 통합하고 테스트로 동작을 확인해줘.
```

### 조사와 문서 작성

```text
$6-1-sol-orch
이 프로젝트의 설치 README를 처음 사용하는 사람도 따라 할 수 있게 고쳐줘.
조사자는 공식 문서에서 필요한 버전과 설치 조건을 확인하고,
메인 에이전트는 로컬 파일과 대조해 설치 절차와 사용 예시를 작성해줘.
```

### 작은 작업

```text
$6-1-sol-orch
README에 있는 잘못된 파일 경로 하나를 고쳐줘.
```

작고 서로 밀접한 작업이면 메인 에이전트가 직접 끝냅니다. 스킬을 호출했다고 하위 에이전트가 반드시 생기지는 않습니다.

## 작업 규칙

- **필요한 역할만 선택:** 독립적으로 나눌 가치가 있는 작업만 맡깁니다.
- **파일마다 작성자 한 명:** 각 에이전트에 목표, 파일 범위, 수정 권한과 완료 조건을 지정합니다.
- **근거를 받은 뒤 통합:** 메인 에이전트가 반환된 결과와 변경을 직접 확인합니다.
- **검증 책임 유지:** 구현자는 관련 테스트를 실행하고, 메인 에이전트는 통합 결과를 확인합니다. 별도의 상시 테스트 역할은 없습니다.
- **독립 검토는 필요할 때:** 보안, 데이터 무결성, 동시성, 호환성, 여러 구성요소에 걸친 위험이나 구체적인 검증 공백이 있으면 Astra 검토를 추가합니다.
- **권한과 동시 실행 한도 준수:** 실제 환경이 제공하는 도구와 제한이 우선합니다.

`agents/openai.yaml`은 암묵적 스킬 선택을 허용하지만, 스킬 본문은 사용자나 적용 지침이 에이전트 위임을 허용했을 때만 위임하도록 제한합니다. 명시적으로 `$6-1-sol-orch`를 호출하면 이 작업 분담 방식의 사용을 요청하는 것입니다. 배포·게시·최종 제출·외부 연락 권한은 별도로 판단합니다.

## 비용과 한계

작은 작업에서 불필요한 위임을 피하려는 설계입니다. **토큰 절감률이나 속도 향상은 측정하지 않았습니다.** 하위 에이전트는 추가 토큰과 대기 시간을 사용할 수 있습니다. 계정의 한도와 모델 비용을 확인하세요.

스킬 지침은 실행 도구의 기능을 새로 만들거나 환경 제한을 우회하지 않습니다. 하위 에이전트가 없는 환경에서는 메인 에이전트가 작업을 수행하고 제약을 알립니다.

## 파일 구성

```text
.
├── README.md
├── README.en.md
├── LICENSE
├── NOTICE
├── assets/                 # README 그림과 편집 원본
└── skills/6-1-sol-orch/
    ├── SKILL.md
    ├── LICENSE
    ├── NOTICE
    └── agents/openai.yaml
```

라이선스와 출처 고지는 스킬 폴더에도 포함돼 폴더만 설치해도 함께 보존됩니다.

그림은 [SVG 원본](assets/orchestration-flow.svg), [PNG](assets/orchestration-flow.png), [노드·연결과 심볼 출처 자료](assets/flow-source/README.md)를 함께 제공합니다. 시각 자료도 이 저장소의 Apache-2.0 라이선스로 수정·재사용할 수 있습니다.

## 출처와 라이선스

[donvito/codex-astra-luna-orchestrator](https://github.com/donvito/codex-astra-luna-orchestrator)의 작업 분담 구조를 바탕으로 만든 로컬 스킬을 공개 배포합니다. 기반으로 삼은 버전은 `f1de1b8729c8ccfb7978321b4764c58bd34c8493`입니다. Sol 6.1 중심 역할 배치, 필요할 때만 위임하는 흐름, 선택적 Astra 검토와 호스트 제약 안내를 담습니다. 변경 내역과 출처는 [NOTICE](NOTICE)에 기록했습니다.

Apache License 2.0으로 사용·수정·재배포할 수 있습니다. OpenAI 공식 프로젝트는 아닙니다.

공식 참고 문서: [Codex 스킬](https://developers.openai.com/codex/skills), [하위 에이전트](https://developers.openai.com/codex/multi-agent), [설정 참고](https://developers.openai.com/codex/config).
