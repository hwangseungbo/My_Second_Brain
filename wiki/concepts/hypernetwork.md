---
title: "Hypernetwork"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources: ["Doc_to_LoRA.pdf"]
tags: [LLM, meta-learning, neural-architecture]
---

# Hypernetwork

**Hypernetwork**은 다른 신경망의 가중치를 생성하는 신경망이다. 입력 데이터에 따라 타겟 네트워크의 파라미터를 동적으로 생성할 수 있다.

## 위키 내 등장

[[sources/doc-to-lora]]에서 D2L의 핵심 구조가 하이퍼네트워크다:
- **입력**: 텍스트 컨텍스트 (문서)
- **출력**: 타겟 LLM을 위한 [[concepts/lora]] 어댑터 가중치
- **아키텍처**: [[concepts/perceiver]] 기반, 309M 파라미터, 8개 cross-attention 블록
- 컨텍스트를 청크 단위로 처리하여 고정 크기 잠재 표현으로 압축한 뒤, LoRA 가중치로 디코딩

## 관련 페이지

- [[sources/doc-to-lora]] — 하이퍼네트워크를 이용한 즉시 컨텍스트 내재화
- [[concepts/lora]] — 생성되는 어댑터의 형태
- [[concepts/context-distillation]] — 하이퍼네트워크가 수행하는 작업
