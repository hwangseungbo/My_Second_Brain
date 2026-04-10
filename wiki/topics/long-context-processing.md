---
title: "Long-Context Processing in LLMs"
type: topic
created: 2026-04-10
updated: 2026-04-10
sources: ["Doc_to_LoRA.pdf", "Lost_in_the_Middle.pdf"]
tags: [LLM, long-context, efficiency, synthesis]
---

# Long-Context Processing in LLMs

LLM이 긴 입력 컨텍스트를 처리하는 방법과 그 한계에 대한 종합 페이지. 현재 두 논문이 이 주제의 서로 다른 면을 다룬다.

## 핵심 문제

Transformer 기반 LLM의 attention 비용은 시퀀스 길이에 대해 **이차적(quadratic)**으로 증가한다. 이는 두 가지 문제를 낳는다:

1. **효율성 문제**: 긴 컨텍스트 → 높은 메모리/레이턴시 비용
2. **활용도 문제**: 긴 컨텍스트를 넣어도 모델이 그 정보를 제대로 사용하지 못함

## 두 가지 관점

### 진단: 모델이 긴 컨텍스트를 어떻게 사용하는가

[[sources/lost-in-the-middle]]은 LLM이 긴 컨텍스트의 정보를 **위치에 따라 불균등하게** 활용한다는 것을 밝혔다:
- 시작과 끝: 잘 활용 → [[concepts/primacy-bias]], [[concepts/recency-bias]]
- 중간: 크게 하락 (GPT-3.5-Turbo에서 20% 이상 성능 저하)
- 컨텍스트 윈도우를 늘려도 이 문제는 해결되지 않음

이는 [[concepts/retrieval-augmented-generation]] 시스템 설계에 직접적인 영향을 미친다.

### 우회: 컨텍스트를 파라미터로 내재화

[[sources/doc-to-lora]]는 긴 컨텍스트를 입력으로 처리하는 대신, 모델 **파라미터 안에 내재화**하는 접근법을 제안했다:
- [[concepts/hypernetwork]]가 문서를 읽고 [[concepts/lora]] 어댑터를 1초 미만에 생성
- 추론 시 원본 문서 불필요 → KV-cache 메모리 대폭 절감
- 모델의 기본 컨텍스트 윈도우(8K)보다 4배 이상 긴 문서도 처리 가능

## 연결점

두 논문은 상호보완적이다:
- Lost in the Middle이 밝힌 **위치 편향 문제**는, 긴 컨텍스트를 있는 그대로 입력하는 방식의 근본적 한계를 보여준다
- Doc-to-LoRA는 이 한계를 **우회**한다 — 컨텍스트를 입력에 넣지 않으므로, 위치 편향 문제 자체가 발생하지 않음
- 다만 D2L은 아직 ICL(in-context learning) 대비 성능 격차가 있으며, 긴 컨텍스트 처리를 완전히 대체하기보다 보완하는 위치에 있음

## 미해결 질문

- D2L이 Lost in the Middle에서 발견된 위치 편향 문제를 실제로 얼마나 완화하는지에 대한 직접적 실험은 아직 없음
- 컨텍스트 윈도우 확장(Rope scaling 등)과 D2L 같은 내재화 접근법 중 어느 것이 장기적으로 더 유망한가?
- 두 접근을 결합하는 하이브리드 시스템의 가능성

## 관련 페이지

- [[sources/doc-to-lora]] — 컨텍스트 내재화 접근법
- [[sources/lost-in-the-middle]] — 긴 컨텍스트 활용 한계 분석
- [[concepts/context-distillation]] — D2L의 이론적 기반
- [[concepts/positional-bias]] — Lost in the Middle의 핵심 발견
- [[concepts/retrieval-augmented-generation]] — 이 문제의 영향을 직접 받는 시스템
