# vLLM 프로젝트 — 서버 운영 문서·엔진 로그 (튜닝 · 지표 · 장애 대응)

- **저자**: vLLM 프로젝트
- **연도**: 2026 (열람 시점 기준 최신판)
- **매체/학회**: 공식 문서 + 엔진 소스의 로그 포맷 문자열
- **링크**:
  - https://docs.vllm.ai/en/latest/configuration/optimization.html
  - https://docs.vllm.ai/en/latest/usage/metrics.html
  - https://docs.vllm.ai/en/latest/usage/troubleshooting.html
  - https://github.com/vllm-project/vllm — 로그·오류 문자열은 엔진 소스에서 직접 확인
- **유형**: 1차 문헌 (구현 주체가 쓴 운영 문서와 그 구현체)

## 핵심 요지

서버를 띄우는 쪽에서 실제로 마주치는 것은 개념이 아니라 **로그 몇 줄과 옵션 몇 개**다. 이 문서들과 엔진 소스를 함께 보면 그 줄들이 임의의 진단 메시지가 아니라 **하나의 나눗셈을 단계마다 찍은 중간값**임이 드러난다 — 카드 메모리에서 무엇을 빼고 남은 자리가 KV 캐시가 되며, 그 자리가 동시 처리 수를 정한다. 기동 실패 메시지도 같은 나눗셈이 한 요청 몫에 못 미쳤다는 보고다.

## 주요 내용

### 기동 로그의 원문 포맷

엔진 소스가 찍는 형식 문자열은 아래와 같다. 값 자리(`%s`·`%.2f`)만 실행마다 바뀐다.

| 단계 | 형식 문자열 |
|---|---|
| 적재 시작 | `Starting to load model %s...` |
| 적재 완료 | `Model loading took %s GiB memory and %.6f seconds` |
| 프로파일링 결과 | `Available KV cache memory: %s GiB` |
| 캐시 확정 | `GPU KV cache size: %s tokens, Maximum concurrency for %s tokens per request: %.2fx` |
| 그래프 캡처 진행 | `Capturing CUDA graphs ({}, {})` |
| 그래프 캡처 완료 | `Graph capturing finished in %.0f secs, took %.2f GiB` |
| 그래프 몫 대조 | `CUDA graph pool memory: %s GiB (actual), %s GiB (estimated), difference: %s GiB (%.1f%%).` |

캐시 확정 줄의 두 값은 독립이 아니다. 소스에서 토큰 수는 `int(max_concurrency * max_model_len)` 으로 계산되므로, **동시성 배수가 먼저 정해지고 토큰 수가 거기서 나온다.** 배수의 뜻은 "모든 요청이 최대 문맥 길이를 끝까지 쓴다고 가정했을 때 동시에 안을 수 있는 요청 수"다.

### 메모리 예산의 구성

프로파일링 단계는 `requested_memory`(총 메모리 × 사용률)에서 가중치·프레임워크 밖 메모리·활성값 최대치를 뺀 나머지를 KV 캐시로 잡는다. 여기에 CUDA graph 예상 몫이 추가로 빠지는데, 그 동작은 환경 변수로 갈린다.

- 기본값(`VLLM_MEMORY_PROFILER_ESTIMATE_CUDAGRAPHS=1`): 그래프 몫을 미리 빼둔다. 문서의 안내는 "The current `--gpu-memory-utilization=%.4f` is equivalent to `--gpu-memory-utilization=%.4f` without CUDA graph memory profiling" — 즉 같은 사용률이라도 예전보다 KV 캐시가 작게 잡힌다.
- 끄면: "Without it, CUDA graph memory is not accounted for during KV cache allocation, which may require lowering `--gpu-memory-utilization` to avoid OOM."

### 설정값 사이의 상충

- `gpu_memory_utilization`: KV 캐시로 미리 잡는 비율. 올리면 "provide more KV cache memory", 너무 높으면 기동 중 메모리 부족.
- `max_num_seqs`: 내리면 "reduces the number of concurrent requests in a batch, thereby requiring less KV cache space".
- `max_num_batched_tokens`: 한 iteration 의 토큰 예산. 작은 값(문서 예시 2048)은 토큰 간 지연에 유리하고, 큰 값(8192 초과, 특히 큰 GPU 의 작은 모델)은 첫 토큰 시간에 유리하다.
- 제약: "When chunked prefill is disabled, `max_num_batched_tokens` must be greater than `max_model_len`".

### 자리가 모자랄 때 — preemption

문서가 인용한 경고 원문:

> `Sequence group 0 is preempted by PreemptionMode.RECOMPUTE mode because there is not enough KV cache space. This can affect the end-to-end performance. Increase gpu_memory_utilization or tensor_parallel_size to provide more KV cache memory. total_cumulative_preemption_cnt=1`

V1 아키텍처의 기본 모드는 `SWAP` 이 아니라 `RECOMPUTE` 다 — 되돌린 요청을 다시 계산하는 편이 오버헤드가 낮기 때문이다. 대처 순서는 문서가 제시한 우선순위대로 사용률 상향 → 동시 요청 수·배치 토큰 예산 하향 → 병렬도 상향이다.

### 운영 중 정기 통계 줄

엔진이 주기적으로 찍는 한 줄은 고정 항목 넷에 조건부 항목이 붙는 형태다.

- 항상: `Avg prompt throughput: %.1f tokens/s`, `Avg generation throughput: %.1f tokens/s`, `Running: %d reqs`, `Waiting: %d reqs`, `GPU KV cache usage: %.1f%%`, `Prefix cache hit rate: %.1f%%`
- 조건부: `Preemptions: %d`(한 번이라도 발생), `Deferred: %d reqs`(대기 중 건너뛴 요청이 있을 때), `External prefix cache hit rate`·`MM cache hit rate`(해당 기능 사용 시)

유휴 상태에서는 `logger.info` 가 아니라 `logger.debug` 로 내려간다 — 놀고 있는 서버가 로그를 채우지 않게 하기 위해서다. 즉 **이 줄이 안 보이면 대개 요청이 없는 것**이다.

### 지표 엔드포인트

| 지표 | 유형 | 재는 것 |
|---|---|---|
| `vllm:time_to_first_token_seconds` | Histogram | 첫 토큰까지 걸린 시간 |
| `vllm:inter_token_latency_seconds` | Histogram | 토큰 사이 간격 |
| `vllm:request_time_per_output_token_seconds` | Histogram | 출력 토큰 1개당 소요 |
| `vllm:e2e_request_latency_seconds` | Histogram | 요청 전체 소요 |
| `vllm:request_queue_time_seconds` | Histogram | 처리 전 대기 시간 |
| `vllm:request_prefill_time_seconds` | Histogram | prefill 구간 소요 |
| `vllm:request_decode_time_seconds` | Histogram | decode 구간 소요 |
| `vllm:prompt_tokens` | Counter | 처리한 prefill 토큰 누계 |
| `vllm:generation_tokens` | Counter | 생성한 토큰 누계 |
| `vllm:num_requests_running` | Gauge | 실행 배치에 들어 있는 요청 수 |
| `vllm:num_requests_waiting` | Gauge | 대기 중인 요청 수 |
| `vllm:kv_cache_usage_perc` | Gauge | KV 캐시 점유율(1.0 이 만석) |
| `vllm:prefix_cache_hits` / `vllm:prefix_cache_queries` | Counter | 적중 수 / 조회 수 — 비율은 나눠서 구한다 |
| `vllm:num_preemptions` | Counter | 누적 preemption 횟수 |

### 기동·운영 장애

엔진이 던지는 오류 원문과 문서의 대처는 아래와 같다.

- 한 요청분도 못 담을 때: `To serve at least one request with the model's max seq len (...), (... GiB KV cache is needed, which is larger than the available KV cache memory (... GiB).` 여기에 `Based on the available memory, the estimated maximum model length is N.` 이 붙어 **줄여야 할 목표치를 엔진이 직접 알려준다.** 대처는 `gpu_memory_utilization` 상향 또는 `max_model_len` 하향.
- 예산이 아예 남지 않을 때: `No available memory for the cache blocks. Try increasing 'gpu_memory_utilization' when initializing the engine`
- 프로파일링 중 외부 간섭: `Error in memory profiling. Initial free memory ... current free memory ...` — "This happens when other processes sharing the same container release GPU memory while vLLM is profiling during initialization." 대처는 카드를 독점시키는 것.
- CUDA graph 관련 크래시: 문서는 `self.graph.replay()` 부근에서 죽는 경우를 들고 `--enforce-eager` 로 캡처를 꺼 원인을 가르라고 안내한다. 대가는 정상 상태의 decode 성능이다.
- 적재가 안 끝날 때: 네트워크 파일시스템 대신 로컬 디스크에 두고, 스왑 스래싱을 확인한다. `--load-format dummy` 로 적재가 병목인지 가른다.
- hybrid 모델 경고: `Hybrid KV cache manager is disabled for this hybrid model, ... we do not enable any optimizations for saving KV cache memory (e.g., dropping the KV cache outside the sliding window).` sliding window 레이어를 full attention 으로 취급해 캐시를 잡으므로 **메모리가 예상보다 크게 잡힌다.** 다만 계산량 절감은 유지된다.

## 인용하는 위키 페이지

- `concepts/vllm-serving-operations.md`
