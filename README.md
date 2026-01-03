# 🎓 AI Instructor Agent
### PPT-based Lecture Video Generation using State-driven Multimodal Agent

본 프로젝트는 PPT 파일을 입력으로 받아  
슬라이드 분석 → 강의 맥락 요약 → 페이지별 설명 생성 → 강의 스크립트 작성 →  
음성 변환(TTS) → 영상 제작까지를 수행하는  
**State 기반 멀티모달 AI 강사 Agent 시스템**입니다.

LangGraph를 활용해 각 단계를 독립적인 노드로 구성하고,  
슬라이드 수에 따라 자동으로 반복·종료되는 **Agent Graph 구조**로 설계했습니다.

---

## 📌 Project Overview

### 🎯 목표
- PPT 기반 강의 콘텐츠 자동 생성
- 슬라이드 단위 처리 + 전체 강의 흐름 유지
- 단순 나레이션이 아닌 **강의 맥락을 고려한 설명 영상 생성**

---

## 🧩 Input & Output

### 📥 Input
- PPTX 파일
- 강의 옵션
  - 강의 난이도 (Beginner / Intermediate / Advanced)
  - 강의 톤 (설명형 / 강의형)
  - 음성 스타일

### 📤 Output
- 슬라이드 기반 강의 영상 (`.mp4`)
- 슬라이드별 음성 및 영상 중간 결과

---

## 🧠 Problem Definition

### 왜 Agent 구조인가?
- 슬라이드 수가 고정되지 않음
- 각 단계가 **이전 결과에 의존**
- 반복·분기·누적 결과 관리 필요

👉 따라서 단순 파이프라인이 아닌  
**State를 공유하는 Agent Graph 구조**로 문제를 재정의

---

## 🗂️ State Design

모든 노드는 공통 State를 중심으로 동작합니다.

```python
class State(TypedDict):
    pptx_path: str
    work_dir: str
    slide_index: int
    slides: List[Dict]

    texts: List[str]
    tables: List[List[List[str]]]
    images: List[str]

    page_content: str
    script: str

    audio: str
    video_parts: List[str]
    video_path: str

### 🧾 State Key Description

- **slides** : 전체 슬라이드 원본 정보 (강의 맥락 유지용)
- **texts / tables / images** : 현재 슬라이드 처리용 데이터
- **slide_index** : 슬라이드 반복 제어 핵심 키
- **video_parts** : 슬라이드별 영상 누적 결과

---

## 🔍 Step 1. PPT Information Extraction

- **python-pptx** 활용
- 슬라이드별 정보 분해
  - 제목
  - 텍스트
  - 표
  - 이미지
- 슬라이드 스냅샷 자동 생성

📌 **슬라이드 전체를 순회하며 구조화된 데이터로 변환**
---

## 🧠 Step 2. Lecture Context Summarization

- 모든 슬라이드를 기반으로 **전체 강의 흐름 요약**
- 슬라이드 간 단절을 방지하기 위한 전역 컨텍스트 생성
- 각 페이지 스크립트 생성 시 공통 참조 정보로 활용

📌 목적  
> 페이지별 설명이 아닌,  
> **하나의 강의처럼 자연스럽게 이어지는 설명 생성**

---

## 🔎 Step 3. External Search-based Enrichment

- 슬라이드 제목을 기준으로 **SerpAPI 검색**
- PPT에 포함되지 않은 배경 지식, 개념 설명 보완
- 기술/이론/사례 중심 정보 자동 확장

📌 효과  
- 정보 밀도 향상  
- 설명의 전문성 및 신뢰도 강화

---

## ✍️ Step 4. Lecture Script Generation

- 슬라이드 위치에 따른 **동적 프롬프트 전략**
  - 첫 장: 강의 도입 및 주제 제시
  - 중간 장: 앞 슬라이드와의 흐름 연결
  - 마지막 장: 전체 내용 요약 및 마무리
- 입력 정보
  - `page_content`
  - `context_summary`
  - 이전 슬라이드 스크립트

📌 결과  
- **말로 읽기 자연스러운 강의 대본 생성**
- 단순 요약이 아닌 설명 중심 스크립트

---

## 🔊 Step 5. Text-to-Speech (TTS)

- OpenAI TTS 모델 활용
- 강의 톤에 맞는 음성 스타일 적용
- 슬라이드별 음성 파일 생성

📌 특징  
- 강의용 말투 유지  
- 페이지 단위 음성 분리로 재사용 가능

---

## 🎬 Step 6. Video Generation

- 슬라이드 스냅샷 + 음성 → 슬라이드별 영상 생성
- `ffmpeg` 기반 영상 처리
- 모든 슬라이드 영상 병합하여 최종 강의 영상 생성

📌 설계 포인트  
- 슬라이드 단위 영상 생성 → 디버깅 및 안정성 향상  
- `-c copy` 옵션을 통한 빠른 병합

---

## 🔁 Agent Graph & Iteration Logic

- `slide_index`를 상태(State)로 관리
- 각 슬라이드 처리 후 index 증가
- 조건 노드를 통해 반복 여부 판단
  - 남은 슬라이드 존재 → CONTINUE
  - 마지막 슬라이드 → DONE

📌 슬라이드 수와 무관한 **자동 반복 구조**

---

## 🖥️ Web Interface (Gradio)

- PPT 파일 업로드
- 강의 옵션 선택
  - 난이도
  - 강의 톤
  - 음성 스타일
- 생성된 강의 영상 다운로드

📌 데모 및 서비스 관점 인터페이스 제공

---

## 🛠️ Tech Stack

- **Language**: Python  
- **LLM**: OpenAI GPT-4o-mini  
- **TTS**: OpenAI TTS  
- **Agent Framework**: LangGraph  
- **Search**: SerpAPI  
- **Video**: FFmpeg  
- **UI**: Gradio  
- **Environment**: Google Colab  

---

## 🔑 Key Insights

- LLM 활용에서 **문제 구조 설계가 성능만큼 중요**
- State 기반 Agent 구조는
  - 강의 흐름 유지
  - 반복 제어
  - 결과 누적 관리에 효과적
- 단순 자동화가 아닌 **강의 품질 중심의 설계**

---

## ⚠️ Limitations

- 음성 자연도 개선 여지
- 자막 자동 생성 미포함
- 실시간 상호작용 미지원

---

## 🚀 Future Work

- 자동 자막 생성
- 강의 스타일 프리셋 관리
- 사내 문서 / 논문 기반 RAG 연동
- 아바타 기반 강의 영상 확장

---

## ✍️ Summary

> PPT를 입력으로 받아  
> 슬라이드 단위로 상태를 관리하며  
> 분석·요약·스크립트·음성·영상을 단계적으로 생성하는  
> **State 기반 멀티모달 AI 강사 Agent 프로젝트**

