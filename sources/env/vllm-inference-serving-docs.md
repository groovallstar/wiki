# vLLM 프로젝트 — 추론 서빙 공식 문서 (prefix caching · speculative decoding · MTP)

- **저자**: vLLM 프로젝트
- **연도**: 2026 (열람 시점 기준 최신판)
- **매체/학회**: 공식 문서
- **링크**:
  - https://docs.vllm.ai/en/latest/design/prefix_caching.html
  - https://docs.vllm.ai/en/latest/features/speculative_decoding/
  - https://docs.vllm.ai/en/latest/features/speculative_decoding/mtp/
- **유형**: 1차 문헌 (구현 주체가 쓴 설계·기능 문서)

## 핵심 요지

같은 모델·같은 하드웨어라도 **서버를 어떻게 띄웠느냐가 처리 속도를 몇 배 바꾼다.** 이 문서들이 다루는 두 기능이 그 대표다 — 이미 계산한 프롬프트 앞부분을 재사용하는 prefix caching, 그리고 다음 토큰을 미리 추측해 한 번에 검산하는 speculative decoding 이다. 둘 다 출력을 바꾸지 않으면서 시간을 줄이지만, **이득이 나는 조건이 정반대**라 워크로드를 보고 골라야 한다.

## 주요 내용

### prefix caching — 계산해둔 앞부분을 건너뛴다

캐시의 단위는 토큰 하나가 아니라 **블록**이다. 문서의 표현으로 "we hash each kv-cache block by the tokens in the block and the tokens in the prefix before the block" — 블록 안의 토큰뿐 아니라 그 앞의 토큰들까지 함께 해시한다. 앞이 다르면 뒤가 같아도 다른 블록이 되므로, 재사용은 **문장 앞에서부터 연속으로 일치하는 구간**에만 일어난다.

제약이 하나 있다. "We only cache full blocks" — 꽉 찬 블록만 캐시된다. 문서 예시에서 14토큰 중 10토큰이 일치했는데도 실제 재사용은 8토큰(온전한 두 블록)뿐이었다. 자투리는 다시 계산한다.

이득의 크기는 적중률에 정비례한다. 맞은 블록은 순전파를 통째로 건너뛰기 때문이다.

### speculative decoding — 추측하고 검산한다

출력이 달라지지 않는다는 것이 이 기법의 전제다. 문서는 "theoretically lossless up to the precision limits of hardware numerics" 라 적고, 검증 항목으로 "greedy sampling with speculative decoding matches greedy sampling without it" 를 명시한다. 즉 **greedy(온도 0) 조건에서는 켜든 끄든 같은 문장이 나와야 정상**이다. 다만 logprobs 의 안정성은 보장하지 않는다.

이득이 나는 자리는 분명하다 — "low QPS (latency focused)" 워크로드에서 "high gain" 이다. 요청이 드물어 GPU 가 놀 때, 노는 연산으로 추측분을 검산해 지연을 줄이는 구조이기 때문이다.

### MTP — 별도 draft 모델이 필요 없는 변형

"MTP is a speculative decoding method where the target model includes native multi-token prediction capability. Unlike draft-model-based methods, you do not need to provide a separate draft model." 추측을 담당하는 작은 모듈이 본 모델 안에 함께 배포된다.

권장값은 추측 토큰 1개다 — "A small value like `1` is a good default to start with". 더 늘리면 한 번에 확보하는 토큰은 늘지만 채택률이 떨어져 전체 처리량이 되레 낮아진다. 1개 설정의 채택률은 통상 90%를 넘는다.

### 두 기능이 부딪히는 지점

MTP 는 **동시 처리량을 깎는다.** 추측 토큰이 KV 캐시 자리를 차지해 한 번에 처리할 수 있는 요청 수가 줄기 때문이다. 그래서 지연이 중요한 저부하에서는 이득이고, 처리량이 중요한 고부하에서는 손해가 될 수 있다. 이득이 큰 쪽은 생성 시간이 지배적인(decode-heavy) 워크로드다.

## 인용하는 위키 페이지

- `concepts/llm-inference-serving.md`
