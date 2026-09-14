# vLLM 프로젝트 — 추론 서빙 공식 문서

- **저자**: vLLM 프로젝트
- **연도**: 2026
- **열람일**: 2026-09-14
- **매체/학회**: 공식 문서
- **검증 기준 커밋**: `73d2a8cf86c46e21af723ffd34ce0d56668fdd14` (개발 브랜치이며 안정 릴리스와 구분한다.)
- **링크**:
  - https://github.com/vllm-project/vllm/blob/73d2a8cf86c46e21af723ffd34ce0d56668fdd14/docs/design/prefix_caching.md
  - https://github.com/vllm-project/vllm/blob/73d2a8cf86c46e21af723ffd34ce0d56668fdd14/docs/features/speculative_decoding/README.md
  - https://github.com/vllm-project/vllm/blob/73d2a8cf86c46e21af723ffd34ce0d56668fdd14/docs/features/speculative_decoding/mtp.md
- **유형**: 1차 문헌

## 핵심 요지

prefix caching은 입력의 공통 계산을 재사용하고, speculative decoding은 초안을 병렬 검증한다. 두 기능의 이득은 워크로드에 따라 측정해야 한다.

## 주요 내용

### Prefix caching

블록 해시에는 앞선 prefix와 현재 토큰 등이 포함된다. 완전한 블록을 캐시하며, 캐시 적중 토큰 비율이 전체 시간 절감률을 뜻하지는 않는다.

### Speculative decoding

target 분포를 보존하는 샘플링과 greedy 동일성 검증을 설명한다. 동시에 부동소수점 정밀도와 배치 크기로 출력이 달라질 수 있으며 logprob 안정성도 보장하지 않는다고 명시한다. 따라서 문자열 차이만으로 알고리즘 결함을 단정하지 않는다.

방법 선택 표는 모델·트래픽·하드웨어·샘플링 조건에 의존하는 정성적 안내다. 중저부하 메모리 병목뿐 아니라 고부하에서도 이득이 가능한 방법을 제시하므로, 고부하에 일괄 부적합하다고 해석하지 않는다.

### MTP

지원 모델 계열의 다중 토큰 예측 기능을 사용하며, 일부 계열은 assistant 체크포인트를 지정한다. 추측 깊이 1은 시작값이다. 보편적인 채택률 90% 또는 깊이 증가에 따른 처리량 감소를 보장하지 않는다. 추가 연산·메모리 비용과 실제 지연·처리량을 함께 측정한다.

## 인용하는 위키 페이지

- `concepts/llm-inference-serving.md`
