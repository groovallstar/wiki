# NVIDIA — Mastering LLM Techniques: Inference Optimization

- **저자**: Shashank Verma, Neal Vaidya (NVIDIA)
- **연도**: 2023 (원문 게시 연도)
- **열람일**: 2026-09-14
- **매체/학회**: NVIDIA Technical Blog
- **링크**: https://developer.nvidia.com/blog/mastering-llm-techniques-inference-optimization/
- **유형**: 1차 문헌 (구현 주체의 기술 해설)

## 핵심 요지

입력의 병렬 prefill과 출력의 순차 decode는 계산 특성이 다르다. 배치·KV 캐시·양자화를 함께 고려하여 추론 비용을 분석한다.

## 주요 내용

- prefill은 입력을 병렬 처리하며, decode는 가중치·KV·활성값의 메모리 이동에 제약받기 쉽다.
- 배치의 여러 요청은 가중치 읽기 비용을 공유하지만, KV 메모리가 배치 확대를 제한한다.
- 양자화는 메모리와 계산 비용을 줄일 수 있으나 정밀도와 하드웨어 지원을 고려해야 한다.
- 위키의 해석: 토큰 수 비율만으로 실행 시간 비율이나 양자화의 속도 배수를 계산할 수 없다.

웹 문서는 수정될 수 있으며, 열람한 설명을 특정 하드웨어의 성능 보장으로 취급하지 않는다.

## 인용하는 위키 페이지

- `concepts/llm-inference-serving.md`
