---
title: "Context Distillation"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources: ["Doc_to_LoRA.pdf"]
tags: [LLM, knowledge-distillation, efficiency, fine-tuning]
---

# Context Distillation

**Context distillation (CD)**은 긴 입력 컨텍스트에 담긴 정보를 모델의 파라미터 안으로 내재화하는 기법이다. 추론 시 원본 컨텍스트를 매번 처리하지 않아도 되도록, 컨텍스트가 있을 때의 모델 출력을 컨텍스트 없이도 재현하도록 모델을 미세조정한다.

## 핵심 아이디어

- 교사(teacher): 컨텍스트가 포함된 프롬프트로 생성한 출력
- 학생(student): 컨텍스트 없이 동일한 출력을 내도록 학습된 모델
- 목적: 추론 시 KV-cache 메모리와 레이턴시 절감

## 한계

전통적 CD는 **각 컨텍스트마다** 별도의 미세조정이 필요하여 실용적이지 않다. 학습에 수십 초~수 분이 걸리며, 새 문서가 올 때마다 반복해야 한다.

## 발전

[[sources/doc-to-lora]]는 이 한계를 극복하기 위해 [[concepts/hypernetwork]]를 사용하여 단일 forward pass로 근사 CD를 수행하는 방법을 제안했다. 문서별 최적화 루프를 메타 학습 단계로 대체하여, 새 컨텍스트에 대해 1초 미만의 내재화를 달성했다.

## 관련 페이지

- [[sources/doc-to-lora]] — CD를 단일 forward pass로 근사하는 D2L 제안
- [[concepts/lora]] — CD의 대상이 되는 경량 어댑터
- [[concepts/hypernetwork]] — D2L에서 CD를 수행하는 메타 학습 네트워크
