# Ramshaw & Marcus (1995) — Text Chunking using Transformation-Based Learning

- **저자**: Lance A. Ramshaw, Mitchell P. Marcus
- **연도**: 1995
- **학회**: Third Workshop on Very Large Corpora (VLC), ACL
- **링크**: https://aclanthology.org/W95-0107/
- **유형**: 1차 문헌 (방법론 원조)

## 핵심 요지

텍스트 청킹(명사구·동사구 등 단순 구조 인식) 작업에 **변환 기반 학습(Transformation-Based Learning)**을 적용한 연구. 본 위키의 관점에서 더 중요한 기여는 청킹 경계를 표현하기 위한 **IOB(=BIO) 태깅 스킴**을 제안·정착시킨 것이다.

## 원문의 IOB 스킴과 IOB2의 구분

원문 §4.1의 baseNP 실험은 다음 태그를 사용한다. 이 방식은 이후 IOB1으로 불린다:
- `B` — 다른 baseNP 바로 뒤에서 시작하는 baseNP의 첫 토큰
- `I` — baseNP 내부의 토큰. 앞에 다른 baseNP가 없는 청크의 시작에도 사용한다.
- `O` — 청크 외부

모든 청크의 시작을 `B-<TYPE>`으로 표시하는 IOB2와 구분해야 한다. `concepts/bio-tagging.md`의 변환 예시는 IOB2를 전제로 한다. 원문은 IOB2의 정의나 공백에 의한 토큰 단위 판별법의 근거가 아니다.

후속 명칭 구분은 Tjong Kim Sang (2002), §3의 설명을 참고한다: https://jmlr.org/papers/volume2/tks02a/html/node9.html

## 인용하는 위키 페이지

- `concepts/bio-tagging.md` — BIO ↔ Span 변환 알고리즘의 전제
