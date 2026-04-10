---
title: "Positional Bias in Long Contexts"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources: ["Lost_in_the_Middle.pdf"]
tags: [LLM, attention, long-context, bias]
---

# Positional Bias in Long Contexts

LLM이 긴 입력 컨텍스트에서 정보의 **위치**에 따라 활용도가 크게 달라지는 현상이다.

## U자형 성능 곡선

[[sources/lost-in-the-middle]]이 체계적으로 입증한 핵심 발견:

- **시작 부분**: 높은 성능 ([[concepts/primacy-bias]])
- **중간 부분**: 성능 급락 — 20% 이상 하락하는 경우도 존재
- **끝 부분**: 높은 성능 ([[concepts/recency-bias]])

이 패턴은 instruction fine-tuning 여부와 무관하게, 기본 모델에서도 나타난다.

## 영향

- 확장된 컨텍스트 윈도우(16K, 100K)를 가진 모델도 동일한 문제를 보임
- [[concepts/retrieval-augmented-generation]]에서 검색된 문서의 배치 순서가 성능에 직접 영향
- 더 많은 문서를 넣는다고 반드시 성능이 올라가지 않음 — 오히려 떨어질 수 있음

## 관련 페이지

- [[sources/lost-in-the-middle]] — 이 현상을 발견하고 체계적으로 분석한 논문
- [[concepts/retrieval-augmented-generation]] — 이 편향의 실질적 영향을 받는 시스템
- [[sources/doc-to-lora]] — 긴 컨텍스트 처리 문제를 우회하는 다른 접근법
