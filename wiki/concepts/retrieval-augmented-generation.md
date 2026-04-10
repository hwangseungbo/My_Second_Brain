---
title: "Retrieval-Augmented Generation (RAG)"
type: concept
created: 2026-04-10
updated: 2026-04-10
sources: ["Lost_in_the_Middle.pdf"]
tags: [LLM, retrieval, information-retrieval, long-context]
---

# Retrieval-Augmented Generation (RAG)

**RAG**는 외부 문서를 검색하여 LLM의 입력 컨텍스트에 추가하는 방식으로, 모델이 학습 데이터에 없는 정보도 활용할 수 있게 하는 기법이다.

## 기본 구조

1. **검색(Retrieval)**: 사용자 질의와 관련된 문서를 검색 엔진에서 가져옴
2. **증강(Augmentation)**: 검색된 문서를 프롬프트에 삽입
3. **생성(Generation)**: LLM이 증강된 컨텍스트를 기반으로 답변 생성

## 한계 — Lost in the Middle 문제

[[sources/lost-in-the-middle]]에 따르면 RAG에 중요한 실질적 한계가 있다:

- 검색된 문서의 **배치 순서**가 성능에 큰 영향을 미침 ([[concepts/positional-bias]])
- 문서 수를 늘리면 retriever recall은 올라가지만, reader 성능은 **포화**하거나 오히려 하락
- 관련 문서를 처음이나 끝에 배치하면 성능이 개선되지만, 이는 어떤 문서가 관련 있는지 이미 아는 경우에만 가능

## 대안적 접근

- [[sources/doc-to-lora]] — 컨텍스트를 매번 입력에 넣는 대신 모델 파라미터에 내재화하여, RAG의 긴 컨텍스트 의존성 자체를 우회

## 관련 페이지

- [[sources/lost-in-the-middle]] — RAG 시스템의 위치 편향 문제 분석
- [[concepts/positional-bias]] — RAG 성능에 영향을 미치는 핵심 요인
