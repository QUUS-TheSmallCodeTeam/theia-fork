# SNS-Core 도입 검토

## 개요

**프로젝트**: [sns-core](https://github.com/EsotericShadow/sns-core)
**목적**: LLM 간 내부 통신을 위한 토큰 효율적 표기 체계

SNS-Core (Shorthand Notation Script)는 다단계 AI 시스템에서 LLM 간 통신을 위해 설계된 표기 체계로, 자연어 대비 60-85%의 토큰 절감을 달성합니다.

## 주요 특징

### 1. 토큰 효율성
- **60-85% 토큰 절감**: 자연어 대비 대폭적인 토큰 사용 감소
- **Zero-shot 이해**: 추가 학습 없이 LLM이 즉시 이해 가능
- **비용 절감**: 프로덕션 환경에서 월 $2K-$10K 절감 사례

### 2. 기술적 특성
- **범용 호환성**: GPT-4, Claude, Llama, Mistral 등 모든 LLM 지원
- **직관적 표기**: 화살표(→), 파이프(|), 조건문(? :) 등 기호 활용
- **성능**: 15-25% 레이턴시 개선, 정확도 저하 없음

### 3. 사용 사례
- RAG 오케스트레이션
- 멀티 에이전트 통신
- 문서 처리 파이프라인
- 챗봇 intent routing
- 데이터 ETL 작업

## EG-DESK 적용 가능성

### 현재 EG-DESK 에이전트 아키텍처
EG-DESK는 다음과 같은 에이전트 스웜 시스템을 운영 중:
- **Main Thread**: 사용자와 직접 통신, 작업 조율
- **PM Agent** (egdesk-pm-agent): 전략적 가이드 제공
- **Framework Analyzers**: theia-analyzer-agent, electron-analyzer-agent
- **Specialized Agents**: ux-flow-simulator, error-recovery, coding-agent 등

### 도입 시나리오

#### 시나리오 1: 에이전트 간 통신 최적화
**현재 문제점**:
- Main Thread → PM Agent 프롬프트가 자연어로 장황함
- 에이전트 간 context 전달 시 토큰 사용량 높음
- 복잡한 워크플로우에서 중간 단계 통신 비용 증가

**SNS-Core 적용**:
```sns
# 현재 (자연어)
"I've completed Phase 1 and found the following issues. Framework agents reported conflicts. My current plan for Phase 2 involves..."

# SNS-Core 적용 후
Phase1:DONE → issues[conflict@framework] | Phase2:plan{...}
```

#### 시나리오 2: 프롬프트 파일 최적화
**현재**: `.claude/prompts/` 디렉토리의 프롬프트 파일이 자연어 기반
**적용 후**: SNS 표기로 간결화하여 토큰 사용 감소

#### 시나리오 3: 에이전트 정의 최적화
**현재**: `.claude/agents/*.md` 파일의 instructions 섹션이 장황
**적용 후**: 핵심 로직을 SNS로 압축 표현

### 예상 효과

#### 긍정적 효과
1. **토큰 비용 절감**: 에이전트 호출당 60-85% 토큰 감소
2. **응답 속도 향상**: 15-25% 레이턴시 개선
3. **확장성**: 더 복잡한 에이전트 워크플로우 구축 가능
4. **프롬프트 관리**: 더 간결하고 유지보수 쉬운 프롬프트

#### 고려 사항
1. **학습 곡선**: 팀 멤버가 SNS 표기법 학습 필요
2. **가독성**: 자연어보다 SNS가 사람에게는 덜 직관적
3. **디버깅**: 에러 발생 시 SNS 표기 해석 필요
4. **초기 구축 비용**: 기존 프롬프트를 SNS로 변환하는 작업

## 도입 로드맵

### Phase 1: 실험 (Proof of Concept)
1. `model.sns` 파일을 프로젝트에 추가
2. 단일 에이전트 통신 경로에 SNS 적용 (예: Main → PM)
3. 토큰 사용량 측정 및 효과 검증

### Phase 2: 점진적 확장
1. 효과가 입증된 경우, 에이전트 간 통신 전반에 확대
2. 프롬프트 파일 일부를 SNS로 변환
3. 가이드 문서 작성 (팀원용 SNS 표기법 가이드)

### Phase 3: 전면 도입
1. 모든 에이전트 instructions에 SNS 적용
2. 자동화 도구 개발 (자연어 ↔ SNS 변환기)
3. 성능 모니터링 대시보드 구축

## 기술 스택 통합

### 필요한 구성 요소
- **model.sns**: SNS 표기법 정의 파일
- **Validation Layer**: SNS 구문 검증 로직 (선택적)
- **Monitoring**: 토큰 사용량 추적 시스템

### 예상 변경 범위
```
.claude/
├── prompts/
│   └── (SNS 변환 필요)
├── agents/
│   └── *.md (instructions 섹션 SNS 적용)
└── sns/
    ├── model.sns (추가)
    ├── guidelines.md (추가)
    └── examples/ (추가)
```

## 결론

### 권장 사항
**결정 보류 → PoC 진행 권장**

SNS-Core는 에이전트 스웜 시스템의 토큰 효율성을 크게 개선할 잠재력이 있으나:
1. EG-DESK 에이전트 시스템의 실제 토큰 사용 패턴을 먼저 분석 필요
2. 소규모 PoC로 효과 검증 후 확대 여부 결정
3. 사용자(개발자) 경험과 디버깅 편의성 고려 필요

### 다음 단계
1. Main Thread → PM Agent 통신 로그 수집 및 토큰 분석
2. SNS 적용 전후 비교 실험 설계
3. PM Agent에게 비전 정렬성 검토 요청
4. PoC 결과 기반 최종 결정

## 참고 자료
- [sns-core GitHub](https://github.com/EsotericShadow/sns-core)
- [EG-DESK Agent Swarm 아키텍처](../../AGENT_SWARM_FLOW.md)
- [Claude Agent SDK](../claude-agent-sdk/)
