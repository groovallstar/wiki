# Zaratiana et al. (2025) — GLiNER2

- **저자**: Urchade Zaratiana, Gil Pasternak, Oliver Boyd, George Hurn-Maloney, Ash Lewis
- **연도**: 2025
- **매체/학회**: EMNLP System Demonstrations, 130–140쪽
- **링크**: https://aclanthology.org/2025.emnlp-demos.10/
- **원문**: https://aclanthology.org/2025.emnlp-demos.10.pdf
- **열람일**: 2026-09-14
- **유형**: 1차 문헌

## 핵심 요지

스키마를 입력으로 받는 encoder로 NER·분류·계층 구조 추출을 통합한다. 서지 제목은 “Schema-Driven Multi-Task Learning for Structured Information Extraction”, PDF 제목은 “An Efficient Multi-Task Information Extraction System with Schema-Driven Interface”이다.

## 주요 내용

- §2는 자연어 타입 설명, 중첩 span, 단일·다중 라벨 분류, 작업 조합을 설명한다.
- §3은 zero-shot NER와 분류를 평가하지만 계층 구조 추출은 벤치마크 부재로 평가하지 않았다.
- Table 3에서 CrossNER 평균 F1은 GLiNER2가 비교 모델 모두를 능가하지 않는다. 기능 확장과 성능 우위를 구분한다.
- Table 4의 CPU 지연은 분류 실험이며 모든 NER·언어·장치 조건으로 일반화하지 않는다.

## 인용하는 위키 페이지

- `concepts/schema-driven-extraction.md`
