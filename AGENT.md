# Agent Entry Point

이 파일을 가장 먼저 읽고 아래 규칙에 따라 필요한 프롬프트 모듈을 로드하세요.

## 1. 자기 식별

Agent는 먼저 스스로 자신을 식별하고 공개 스펙을 조회합니다. 아래 룩업 테이블을 사용합니다.

| 모델 | 최대 컨텍스트 | 실용 임계값 (80%) | 경고 임계값 (70%) |
|------|-------------|-----------------|-----------------|
| Claude Opus 4.6 / Sonnet 4.6 | 200,000 | 160,000 | 140,000 |
| Claude Haiku 4.5 | 200,000 | 160,000 | 140,000 |
| GPT-4o / GPT-4o mini | 128,000 | 102,400 | 89,600 |
| o1 / o3 / o4-mini | 128,000 | 102,400 | 89,600 |
| DeepSeek-V3 / R1 | 128,000 | 102,400 | 89,600 |
| Gemini 1.5 Pro | 1,000,000 | 800,000 | 700,000 |
| Gemini 2.0 Flash | 1,000,000 | 800,000 | 700,000 |
| GLM-4 | 128,000 | 102,400 | 89,600 |
| Llama 3.1 / 3.3 70B+ | 128,000 | 102,400 | 89,600 |
| Qwen2.5 72B+ | 128,000 | 102,400 | 89,600 |
| 기타 / 알 수 없음 | 보수적 기본값 적용 | 80,000 | 70,000 |

자신의 모델명을 특정할 수 없거나, 운영자가 컨텍스트를 별도 제한했을 수 있는 경우에만 아래 질문을 합니다.

```
[환경 설정] 최초 1회 확인이 필요합니다.

① 모델명: (예: Claude Sonnet 4.6 / GPT-4o / DeepSeek-V3)
   → 모르시면 "모름" — 보수적 기본값(80K)을 적용합니다.

② 운영자 컨텍스트 제한 여부:
   - max_tokens를 별도로 제한했다면 그 값을 알려주세요.
   - 없으면 "기본값"이라고 하세요.

이 정보는 .ai.dev/AGENT_CONFIG.md에 저장되며 다시 묻지 않습니다.
```

식별 완료 후 `.ai.dev/AGENT_CONFIG.md`를 생성합니다.

```
# Agent Configuration

- model_name: (감지 또는 사용자 입력값)
- self_identified: true/false
- user_confirmed: true/false
- max_tokens: (공개 스펙값)
- operator_limit: null
- effective_max: (min(max_tokens, operator_limit))
- compression_threshold: (effective_max × 0.80)
- warning_threshold: (effective_max × 0.70)
- environment: cli
- file_access: true
- context_compression: enabled
- avg_tokens_per_exchange: 800
- system_prompt_tokens: 20000
- calibration_count: 0
```

## 2. 프로젝트 상태 확인

`.ai.dev/DECISIONS.md`를 읽어 Q0~Q16 답변을 확인합니다.
답변이 없으면 사용자에게 Q0부터 질문을 시작합니다.
답변이 일부만 있으면 이어서 진행합니다.

## 3. 프롬프트 모듈 로드 매트릭스

### Core (항상 로드 — 순서대로)

| 순서 | 파일 | 내용 |
|:----:|------|------|
| 1 | prompts/core/base-rules.prompt | Agent 행동 원칙, 위험 방지, 보안, 승인, 충돌 감지 |
| 2 | prompts/core/file-management.prompt | .ai.dev 구조, 문서화 규칙 |
| 3 | prompts/core/git-conventions.prompt | Conventional Commits, 브랜치 전략, 버전 관리 |
| 4 | prompts/core/questions-base.prompt | Q0~Q16 공통 질문 템플릿 |
| 5 | prompts/core/codebase-profile.prompt | 코드베이스 프로파일 캐시 |
| 6 | prompts/core/modification-protocol.prompt | 코드 수정 실행 프로토콜, 버그 영향 분석 |
| 7 | prompts/core/testing.prompt | 테스트 자동화, 커버리지 |
| 8 | prompts/core/logging.prompt | 로깅 표준 |
| 9 | prompts/core/errors.prompt | 에러 처리 표준화 |
| 10 | prompts/core/context-lifecycle.prompt | 컨텍스트 관리, 압축, 아카이브 |
| 11 | prompts/core/commands.prompt | AI 명령어 |
| 12 | prompts/core/initialization.prompt | 초기화 절차 |

### Profile (DECISIONS.Q0 기반 — 1개 로드)

| Q0 선택 | 로드 파일 |
|---------|----------|
| A. 신규 개발 | prompts/profile/greenfield.prompt |
| B/C/D. 마이그레이션 | prompts/profile/migration.prompt |

### Scale (DECISIONS.Q3 기반 — 1개 로드)

| Q3 선택 | 로드 파일 |
|---------|----------|
| A. 소형 | prompts/scale/small.prompt |
| B. 중형 | prompts/scale/medium.prompt |
| C. 대형 | prompts/scale/large.prompt |

### Stack (DECISIONS.Q5~Q8 기반 — 다중 로드)

| 선택 | 로드 패턴 |
|------|----------|
| 언어 | prompts/stack/lang-{선택값}.prompt |
| DB | prompts/stack/db-{선택값}.prompt |
| 백엔드 프레임워크 | prompts/stack/framework-{선택값}.prompt |
| 프론트엔드 프레임워크 | prompts/stack/framework-{선택값}.prompt |
| 도구 | prompts/stack/tools.prompt |

### Optional (DECISIONS.Q14~Q16 기반 — 선택 항목만)

| 질문 | 선택 | 로드 파일 |
|:----:|------|----------|
| Q14 | A/B/C (관측 가능성) | prompts/optional/observability.prompt |
| Q15-1 | CI/CD | prompts/optional/cicd.prompt |
| Q15-2 | IaC | prompts/optional/iac.prompt |
| Q15-3 | API Gateway | prompts/optional/api-gateway.prompt |
| Q15-4 | Service Mesh | prompts/optional/service-mesh.prompt |
| Q15-5 | Secrets | prompts/optional/secrets.prompt |
| Q15-6 | GitOps | prompts/optional/gitops.prompt |
| Q15-7 | IDP | prompts/optional/idp.prompt |
| Q16-1 | 부하/성능 테스트 | prompts/optional/performance-testing.prompt |
| Q16-2 | 보안/정적 분석 심화 | prompts/optional/security-advanced.prompt |
| Q16-3 | 소프트웨어 공급망 보안 | prompts/optional/security-advanced.prompt |
| Q16-4 | 카오스 엔지니어링 | prompts/optional/chaos-engineering.prompt |
| | MLOps | prompts/optional/mlops.prompt |

### Patterns (작업 중 필요 시 동적 로드)

| 파일 | 트리거 |
|------|--------|
| prompts/patterns/async-patterns.prompt | 비동기 패턴 필요 시 |
| prompts/patterns/distributed-patterns.prompt | MSA, 분산 시스템 패턴 필요 시 |
| prompts/patterns/data-processing.prompt | 대용량 데이터 처리 필요 시 |

## 4. 로드 규칙

1. Core 모듈은 순서대로 모두 로드합니다.
2. Profile과 Scale은 DECISIONS.md 기반으로 각 1개만 선택하여 로드합니다.
3. Stack은 DECISIONS.md에 명시된 선택값에 해당하는 파일만 로드합니다.
4. Optional은 사용자가 선택한 항목만 로드합니다.
5. Patterns는 작업 중 필요 시 동적으로 로드합니다.

## 5. 경량 모드 (소형 프로젝트)

DECISIONS.Q3 = A(소형) 인 경우:

- Core 모듈 중 1~6까지만 로드합니다 (base-rules, file-management, git-conventions, questions-base, codebase-profile, modification-protocol).
- Profile과 Scale은 정상 로드합니다.
- Stack은 언어와 DB만 로드합니다.
- Optional 모듈은 로드하지 않습니다.
- 질문은 Q0~Q4만 진행하고, Q5~Q8은 기본값을 자동 적용합니다.

## 6. 충돌 해결

- 모듈 간 규칙 충돌 시: Core > Profile > Scale > Stack 순으로 우선합니다.
- 기존 코드베이스 컨벤션(CODEBASE_PROFILE.md)과 충돌 시 사용자에게 확인합니다.
- 중요도 태그: `[근본]` > `[전술]` > `[가역]` 순으로 압축 시 보존 우선순위를 결정합니다.

## 7. 환경 설정 명령어

| 명령어 | 동작 |
|--------|------|
| `reconfigure agent` | AGENT_CONFIG.md 재설정 (모델 변경 시 사용) |
| `show agent config` | 현재 설정값 출력 |
| `show token estimate` | 현재 토큰 사용량 추정값 출력 |
| `calibrate tokens` | 보정값 수동 업데이트 |
| `initialize project` | 초기화 절차 실행 |