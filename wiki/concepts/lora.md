---
title: "LoRA (Low-Rank Adaptation)"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources: ["Doc_to_LoRA.pdf"]
tags: [LLM, fine-tuning, parameter-efficient, adapter]
---

# LoRA (Low-Rank Adaptation)

**LoRA**는 대규모 언어 모델의 파라미터 효율적 미세조정 기법이다. 원본 가중치를 고정한 채, 각 레이어에 저랭크(low-rank) 행렬 쌍(A, B)을 추가하여 학습한다.

## 핵심 원리

- 가중치 변화 ΔW를 저랭크 분해 ΔW = BA로 표현 (B: d×r, A: r×d, r << d)
- 전체 모델 대비 극소수의 파라미터만 학습
- 추론 시 원본 가중치에 병합 가능 → 추가 레이턴시 없음

## 위키 내 등장

- [[sources/doc-to-lora]] — [[concepts/hypernetwork]]가 LoRA 어댑터를 즉석 생성하여 컨텍스트를 모델에 내재화. Rank 16을 사용하며, Rank가 높을수록 정보 저장량이 증가하지만 비용도 증가.

## 관련 페이지

- [[concepts/context-distillation]] — LoRA를 통해 컨텍스트를 파라미터에 주입
- [[concepts/hypernetwork]] — LoRA 가중치를 생성하는 메타 네트워크
