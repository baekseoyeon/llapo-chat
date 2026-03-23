# PROJECT

## Overview

LLaPo-chat은 사용자가 AI의 성격을 직접 조절하고, 그 조절 경험이 대화 UX에 어떤 영향을 주는지 확인하기 위해 만든 챗봇 시스템입니다.

이 프로젝트는 Personality Vector Merging 기반 성격 제어 기술을 사용자 인터페이스로 확장한 사례입니다. 사용자는 Big Five 성격 축을 슬라이더로 조절한 뒤, 적용된 성격으로 챗봇과 대화할 수 있습니다.

이 문서는 시스템의 목적, 구조, 주요 기능, 사용자 흐름을 정리합니다.

## Goal

이 프로젝트의 목표는 두 가지입니다.

- 사용자가 대화형 AI의 성격을 직접 조절할 수 있는 인터페이스를 설계하는 것
- 조절 기능이 실제 대화 경험에 어떤 변화를 만드는지 검증 가능한 구조로 구현하는 것

## Scope

LLaPo-chat은 기술 실험 자체보다, 성격 제어 기술을 서비스 형태로 구성하고 사용자 경험을 관찰하는 데 초점을 둡니다.

이 레포에서 다루는 범위는 다음과 같습니다.

- slider 기반 성격 조절 인터페이스
- 성격 설정값을 모델 병합 계수로 반영하는 흐름
- 대화 진행, reset, 사후 설문까지 이어지는 사용자 흐름
- LLaPo-chat / LLaPo-base 비교가 가능한 실험 구조

## System structure

시스템은 다음 구조로 구성됩니다.

- **User**: 성격을 조절하고 챗봇과 대화하는 참가자
- **Streamlit App**: 슬라이더, 버튼, 채팅 인터페이스를 제공하는 프론트엔드
- **FastAPI Server**: 성격 설정을 모델 병합에 반영하고 응답을 생성하는 백엔드
- **LLM**: personality vector merging을 적용한 대화 모델

기본 흐름은 아래와 같습니다.

1. 사용자가 성격 슬라이더를 조절
2. 설정값을 내부 병합 계수로 변환
3. 병합된 모델로 응답 생성
4. 대화 후 reset 또는 다음 세션으로 이동

## Personality control design

LLaPo-chat은 사용자가 여러 trait를 자유롭게 조절할 수 있도록 하되, 설정이 지나치게 분산되거나 불안정해지지 않도록 제한된 포인트 체계를 사용합니다.

### Personality budget

- 사용자는 정해진 총 포인트 안에서 Big Five trait에 값을 분배합니다.
- 각 trait는 한 축에서 High 또는 Low 중 한 방향으로만 조절합니다.
- 내부적으로는 포인트가 병합 계수로 변환되어 모델에 반영됩니다.

이 구조는 두 가지 목적을 가집니다.

- 사용자가 여러 성격 특성을 직접 조합할 수 있도록 함
- 실험 조건이 지나치게 분산되지 않도록 제어 범위를 일정하게 유지함

### Internal mapping

- trait point: `n_p`
- coefficient: `c_p = 0.2 × n_p`
- total constraint: `Σ c_p ≤ 2.0`

사용자에게는 직관적인 포인트 분배 방식으로 보이지만, 내부적으로는 모델 안정성을 고려한 병합 계수 규칙을 따릅니다.

## Session conditions

실험은 두 조건을 비교할 수 있도록 설계했습니다.

- **LLaPo-chat**: 사용자가 성격을 직접 조절할 수 있는 조건
- **LLaPo-base**: 동일한 백본 모델을 사용하지만 성격 조절 기능은 제공하지 않는 조건

두 조건은 가능한 한 같은 흐름으로 진행하고, 차이는 성격 조절 가능 여부에만 두었습니다.

## User flow

LLaPo-chat의 기본 흐름은 다음과 같습니다.

1. **Notice**  
   실험 안내와 주의사항을 확인합니다.

2. **Adjust**  
   사용자가 Big Five 슬라이더를 조절하고 적용합니다.

3. **Merging**  
   선택한 설정이 모델 병합에 반영됩니다.

4. **Chatting**  
   적용된 성격으로 챗봇과 대화를 진행합니다.

5. **Reset / Survey**  
   필요 시 초기화한 뒤, 직전 대화에서 느낀 성격과 경험을 기록합니다.

6. **Next session or Finish**  
   다음 조건 또는 다음 토픽으로 이동합니다.

## Interaction rules

실험 중 조건 간 차이를 줄이고 흐름을 안정적으로 유지하기 위해 phase별로 UI 동작을 제한했습니다.

- merging 중에는 슬라이더와 채팅 입력을 비활성화
- chatting 중에는 reset만 허용
- reset survey 중에는 사이드바 전체 비활성화
- base 조건에서는 슬라이더를 표시하지 않음

이 방식은 사용자가 어느 단계에 있는지 명확히 알 수 있게 하고, 실험 흐름이 섞이지 않도록 돕습니다.

## Topic design

대화는 일상적인 주제를 기준으로 진행했습니다.

- What happened over the weekend
- Recent concerns or worries
- Current interests or hobbies

토픽은 특정 정답을 요구하지 않으면서도 성격 차이가 대화 방식에 자연스럽게 드러날 수 있는 주제로 구성했습니다.

## What this repository includes

- `README.md`  
  프로젝트 개요와 핵심 흐름

- `docs/PROJECT.md`  
  시스템 목적, 구조, 기능, 사용자 흐름 정리

- `docs/STUDY.md`  
  실험 설계와 평가 구조 정리

- `image/`  
  시스템 화면 및 설명 이미지

## Research context

LLaPo-chat은 personality vector 기반 성격 제어 기술을 사용자 조절형 서비스로 확장한 프로젝트입니다.

기술 구현 자체는 별도 레포에서 다루고, 이 레포는 시스템 설계와 UX 검증에 집중합니다.

## Related work

Core personality control method:

**Personality Vector: Modulating Personality of Large Language Models by Model Merging**  
EMNLP 2025 Main Conference  
[arXiv](https://arxiv.org/abs/2509.19727)

## Thesis

**One Model Fits You: Controlling Large Language Models through Personality Vector Merging**
