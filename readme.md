# Google Cloud CX Agent Studio (CXAS) — Custom Voice 생성 및 온보딩 단계별 가이드

> **문서 개요**: 본 가이드는 **Google Cloud Gemini Enterprise for Customer Experience** 환경에서 실제 성우 또는 대표 상담원의 목소리를 녹음하여 **브랜드 전용 커스텀 보이스(Custom Voice)** 를 셀프서비스로 생성하고 에이전트에 적용하는 **단계별(Step-by-Step) 실무 가이드**입니다.
>
> **Custom Voice UI 공식 GA 출시 (2026년 10월 28일)** 에 따라, 별도의 프로젝트 허용 목록(Allowlist) 승인 대기 없이 담당자가 직접 웹 콘솔(UI)에서 **오디오 파일 2종(동의서 + 음성 샘플)** 을 업로드하거나 브라우저 마이크로 녹음하여 즉시 모든 아키텍처에 적용할 수 있습니다.

---

## 📑 목차 (Table of Contents)

- [Custom Voice 온보딩 전체 흐름 한눈에 보기](#-custom-voice-온보딩-전체-흐름-한눈에-보기)
1. [Step 1: 사전 준비 — 필수 오디오 2종 규격 및 3대 자동 검증 기준 이해하기](#1-step-1-사전-준비--필수-오디오-2종-규격-및-3대-자동-검증-기준-이해하기)
   - [1.1. 제출해야 하는 오디오 파일 2종 요약](#11-제출해야-하는-오디오-파일-2종-요약)
   - [1.2. 저장 시 수행되는 3대 실시간 생체·보안 검증 기준](#12-저장-시-수행되는-3대-실시간-생체보안-검증-기준)
2. [Step 2: 오디오 2종 녹음하기 (동의서 문구 + 표준 샘플 대본)](#2-step-2-오디오-2종-녹음하기-동의서-문구--표준-샘플-대본)
   - [2.1. 파일 ①: 음성 동의서 (`Consent audio` — 8~12초 필수 낭독 문구)](#21-파일--음성-동의서-consent-audio--812초-필수-낭독-문구)
   - [2.2. 파일 ②: 참조 음성 샘플 (`Voice sample` — 20~60초 표준 대본 3선)](#22-파일--참조-음성-샘플-voice-sample--2060초-표준-대본-3선)
3. [Step 3: CX Agent Studio 웹 콘솔에서 대상 언어 및 지원 모델 설정하기](#3-step-3-cx-agent-studio-웹-콘솔에서-대상-언어-및-지원-모델-설정하기)
4. [Step 4: `Create custom voice` 화면에서 Custom Voice 생성하기](#4-step-4-create-custom-voice-화면에서-custom-voice-생성하기)
   - [4.1. 억양 및 발화 스타일 지시문(`Voice instruction`) 설정하기](#41-억양-및-발화-스타일-지시문voice-instruction-설정하기)
   - [4.2. 산업별·상황별 `Voice instruction` 프롬프트 템플릿](#42-산업별상황별-voice-instruction-프롬프트-템플릿)
5. [Step 5: 대상 언어에 Custom Voice 매핑 및 통화 행동 최적화하기](#5-step-5-대상-언어에-custom-voice-매핑-및-통화-행동-최적화하기)
   - [5.1. 대상 언어에 생성한 Custom Voice 연결하기](#51-대상-언어에-생성한-custom-voice-연결하기)
   - [5.2. 발화 속도(Tempo), 배경 효과음, 끼어들기(Barge-in), 묵음 대기 설정](#52-발화-속도tempo-배경-효과음-끼어들기barge-in-묵음-대기-설정)
6. [Step 6: 시뮬레이터(Simulator) 실시간 음성 테스트 및 검증](#6-step-6-시뮬레이터simulator-실시간-음성-테스트-및-검증)
7. [부록 A: CES v1 REST API를 활용한 Custom Voice 자동화 및 운영 주의사항](#7-부록-a-ces-v1-rest-api를-활용한-custom-voice-자동화-및-운영-주의사항)
   - [A.1. 아키텍처별 내부 동작 및 5일 주기 자동 갱신(Re-mint) 주의사항](#a1-아키텍처별-내부-동작-및-5일-주기-자동-갱신re-mint-주의사항)
   - [A.2. Cloud Storage(GCS) 권한 부여 및 REST API (`GET` / `PATCH`) 예제](#a2-cloud-storagegcs-권한-부여-및-rest-api-get--patch-예제)
8. [부록 B: 자주 묻는 질문 및 에러 해결 가이드 (Troubleshooting & FAQ)](#8-부록-b-자주-묻는-질문-및-에러-해결-가이드-troubleshooting--faq)

---

## 📌 Custom Voice 온보딩 전체 흐름 한눈에 보기

**CX Agent Studio Custom Voice (셀프서비스 UI)** 는 실제 성우나 상담원이 녹음한 **① 동의서 오디오(8~12초)** 와 **② 참조 음성 샘플(20~60초)** 두 개의 파일만으로 브랜드 고유의 AI 상담원 목소리를 즉시 생성하고 원하는 언어(한국어, 영어, 일본어 등 30개 이상의 다국어 로케일)에 적용하는 기능입니다.

```mermaid
flowchart LR
    subgraph S12["🎙️ Step 1 ~ 2: 오디오 2종 녹음"]
        direction TB
        R1["파일 ①: 음성 동의서 (Consent audio)\n• 8~12초 필수 동의 문구 낭독"]
        R2["파일 ②: 참조 음성 샘플 (Voice sample)\n• 20~60초 표준 대본 낭독"]
        R1 & R2 --> Same["동일 화자 / 동일 마이크 / 동일 공간\n연속 녹음 필수"]
    end

    subgraph S34["🖥️ Step 3 ~ 4: 웹 콘솔에서 보이스 생성"]
        direction TB
        Lang["Step 3: Language controls에서\n대상 언어 추가 및 지원 모델 확인"]
        Create["Step 4: Create custom voice 패널\n• 파일 업로드(Upload file) 또는 직접 녹음(Record audio)\n• 억양/톤 지시문(Voice instruction) 입력"]
        Lang --> Create
    end

    subgraph S56["✅ Step 5 ~ 6: 보이스 매핑 및 통화 테스트"]
        direction TB
        Verify["3대 실시간 보안/생체 검증 통과\n(동의 문구 전사 / 화자 일치 / 실제 육성 확인)"]
        Map["Step 5: 대상 언어 음성을\nCustom으로 전환 후 보이스 선택"]
        Sim["Step 6: 시뮬레이터(Simulator)에서\n실시간 음성 대화 테스트"]
        Verify --> Map --> Sim
    end

    S12 ==> S34 ==> S56
```

---

## 1. Step 1: 사전 준비 — 필수 오디오 2종 규격 및 3대 자동 검증 기준 이해하기

### 1.1. 제출해야 하는 오디오 파일 2종 요약

커스텀 보이스 1개를 생성하려면 **동일한 화자(성우/상담원)가 한 자리에서 연속으로 녹음한 2개의 오디오 파일**이 필요합니다.

| 구분 | 콘솔 UI 항목명 | 권장 녹음 길이 | 지원 파일 포맷 및 상세 규격 |
| :--- | :--- | :--- | :--- |
| **파일 ①** | **`Consent audio`**<br/>(음성 동의서) | **8 ~ 12초** | • **지원 포맷**: `WAV` (강력 권장), `MP3`, `M4A`, `OGG`, `FLAC`<br/>• **파일 크기 제한**: 파일당 최대 **10 MB** 이하<br/>• **권장 녹음 규격**: 무손실 PCM `WAV` (24 kHz 또는 48 kHz, 16-bit / 24-bit, Mono) |
| **파일 ②** | **`Voice sample`**<br/>(참조 음성 샘플) | **20 ~ 60초**<br/>*(최적 구간: **30 ~ 40초**)* | • 서버 업로드 시 내부 FFmpeg 파이프라인이 모든 지원 오디오 포맷을 자동으로 **24 kHz / 16-bit / Mono 표준 WAV**로 변환하여 처리합니다.<br/>• **묵음(Silence) 비율 제한**: 전체 녹음 길이 중 말이 없는 무음 구간이 **20% 미만**이어야 합니다. |

---

### 1.2. 저장 시 수행되는 3대 실시간 생체·보안 검증 기준

본인 동의 없는 타인 목소리 복제 및 딥페이크 악용을 방지하기 위해, 콘솔에서 **`Create` / `Save`** 버튼을 누르면 백엔드에서 아래 **3가지 보안 검증**을 실시간으로 수행합니다.

| 검증 항목 | 검증 원리 및 통과 기준 | 업로드가 거부되는 주요 원인 |
| :--- | :--- | :--- |
| **① 동의 문구 전사 일치 검증**<br/>*(ASR Transcript Match)* | `Consent audio`(파일 ①)를 STT로 텍스트 변환하여 공식 동의 문구와 일치하는지 대조합니다.<br/>• **통과 기준**: **단어 오류율(WER) 15% 미만** | • 지정된 동의 문구의 조사나 단어를 임의로 바꾸거나 생략한 경우<br/>• 문구 앞뒤에 `"안녕하세요"`, `"녹음 시작하겠습니다"` 같은 불필요한 멘트를 추가한 경우 |
| **② 화자 신원 일치 검증**<br/>*(Speaker Identity Match)* | 음향 임베딩(Acoustic embedding) 모델이 **파일 ①(동의서)의 화자**와 **파일 ②(음성 샘플)의 화자**가 동일 인물인지 성문 유사도를 검증합니다. | • 성우 본인이 아닌 다른 사람이 동의서를 대독한 경우<br/>• **같은 화자라도 서로 다른 날, 다른 방, 다른 마이크(예: 휴대폰 vs 스튜디오 마이크)로 따로 녹음하여 음향 특성이 달라진 경우** |
| **③ 실제 육성 확인 검증**<br/>*(DeepMind SynthID / SSD)* | 업로드된 오디오가 AI 합성 음성이 아닌 **실제 사람의 육성(Live human speaker)** 인지 판별합니다. | • 외부 TTS나 AI로 생성한 합성 음성을 업로드한 경우<br/>• 오디오 편집 프로그램에서 과도한 노이즈 제거 필터나 피치 변조 이펙트를 적용해 원음이 왜곡된 경우 |

> [!CAUTION]
> **🚨 필수 주의사항: 두 파일(파일 ① & 파일 ②)은 반드시 한 자리에서 연속 녹음하세요!**
> 동의서 파일(`Consent audio`)과 샘플 파일(`Voice sample`)의 마이크, 공간 울림, 마이크 거리가 서로 다르면 동일 인물이더라도 화자 검증 알고리즘이 불일치로 판단하여 다음 에러와 함께 생성을 차단합니다:
> `APP_ERROR: Voice mismatch detected. The speaker in the consent audio does not match the speaker in the voice sample.`

---

## 2. Step 2: 오디오 2종 녹음하기 (동의서 문구 + 표준 샘플 대본)

### 2.1. 파일 ①: 음성 동의서 (`Consent audio` — 8~12초 필수 낭독 문구)

조용한 환경에서 차분하고 또렷한 발음으로 아래 문장을 **토씨 하나 틀리지 않고 그대로(Word-for-Word)** 소리 내어 읽고 첫 번째 파일(`speaker_01_consent.wav`)로 저장하세요. 앞뒤에 다른 인사말을 넣지 말고 아래 문장만 읽어야 합니다.

#### 📌 공식 음성 동의 문구 (Gemini TTS / Voices API 기준)
```text
"저는 이 음성의 소유자이며, Google Cloud를 사용하여 제 음성의 합성 모델을 생성하는데 동의했습니다."
```

<details>
<summary>💡 <b>(참고) 영문 또는 Chirp 3 모델용 공식 동의 문구 보기</b></summary>

성우가 마이크 앞에 앉았을 때 아래 문장(각 10초 내외)도 같은 자리에서 함께 녹음해 두시면, 다국어 에이전트 구성이나 세부 TTS 모델(`Chirp3-HD` 등) 변경 시 성우 재섭외 없이 즉시 활용할 수 있습니다:

1. **영문(`en-US`) 공식 동의 문구 (`speaker_01_consent_en.wav`)**:
   ```text
   "I am the owner of this voice and have consented to the creation of a synthetic model of my voice through the use of Google Cloud."
   ```
2. **Chirp 3: Instant Custom Voice 공식 동의 문구 (`speaker_01_consent_chirp.wav`)**:
   ```text
   "나는 이 음성의 소유자이며 구글이 이 음성을 사용하여 음성 합성 모델을 생성할 것을 허용합니다."
   ```
3. **기타 30개 이상 다국어 공식 동의 문구**: 일본어(`ja-JP`), 스페인어(`es-ES`), 프랑스어(`fr-FR`), 독일어(`de-DE`) 등 전체 언어별 공식 동의 문구는 [Gemini Enterprise Agent Platform — Replicate voices 공식 문서](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/text-to-speech/voice-replication#consent-statements)에서 확인하실 수 있습니다.
</details>

---

### 2.2. 파일 ②: 참조 음성 샘플 (`Voice sample` — 20~60초 표준 대본 3선)

AI 모델이 화자의 성문 프로필을 가장 풍부하고 자연스럽게 학습할 수 있도록 평음·경음·격음 파열음, 마찰음, 파찰음, 비음·유음 연음, 이중모음, 숫자 발음, 그리고 친절한 종결 어미 억양을 균형 있게 배치한 **표준 스크립트 3선**입니다.

동의서 녹음 직후, **같은 자리·같은 마이크 그대로** 아래 **Option A, B, C 중 브랜드 성격에 맞는 대본 1개를 선택하여 약 35~40초 분량**으로 낭독하고 두 번째 파일(`speaker_02_sample.wav`)로 저장하세요.

---

#### 🎙️ 표준 대본 Option A: 일반 고객 상담 및 멤버십 안내 (가장 범용적인 고객센터용)
```text
"안녕하세요, 고객님! 구글 클라우드의 맞춤형 음성 기술로 구현된 인공지능 전담 상담원입니다. 매월 청구되는 요금 내역을 확인하시거나, 프리미엄 멤버십 혜택으로 변경하시는 것은 물론, 갑작스러운 서비스 이용 불편을 해결하실 때까지 제가 곁에서 빠르고 편리하게 도와드리겠습니다. 저희 고객센터는 매시간 삼백육십 건이 넘는 다양한 문의를 처리하고 있지만, 지금 이 순간 저의 가장 중요한 목표는 고객님께 정확하고 신뢰할 수 있는 답변을 드리는 것입니다. 오늘 어떤 업무를 먼저 도와드릴까요?"
```
- **글자 수 / 예상 시간**: 68 어절 (공백 포함 267자) / 자연스러운 상담 속도 기준 **약 35 ~ 40초**
- **핵심 음운 학습 포인트**:
   - 비음·유음 연음(`구글 클라우드`, `청구되는`, `삼백육십[삼뱅뉵씹]`), 경음·격음(`청구`, `빠르고`, `께`), 이중모음(`요금`, `내역`, `혜택`, `문의`), 친절한 의문형 끝올림 억양(`~도와드릴까요?`)

---

#### 🎙️ 표준 대본 Option B: 기술 지원 및 서비스 장애 트러블슈팅 (IT·통신·가전 고객지원용)
```text
"고객지원센터에 전화해 주셔서 감사합니다. 고객님의 전담 인공지능 음성 어시스턴트로서, 본인 인증 확인부터 결제 수단 변경, 그리고 플랫폼 전반의 정밀 진단 도구 연결까지 지체 없이 바로 안내해 드리겠습니다. 현재 저희 자동화 시스템은 일 분당 칠십오 건 이상의 복잡한 기술 문의를 실시간으로 분석하여 대기 시간 없는 쾌적한 해결책을 제공하고 있습니다. 지금 바로 고객님의 프로필을 살펴보고 사십이 번 접수 티켓의 처리 상태를 확인하여 모든 서비스가 정상 작동하도록 조치해 드리겠습니다. 어떤 항목부터 먼저 점검해 볼까요?"
```
- **글자 수 / 예상 시간**: 71 어절 (공백 포함 282자) / 차분한 기술 상담 속도 기준 **약 35 ~ 40초**
- **핵심 음운 학습 포인트**:
   - 선명한 파열음·파찰음(`결제`, `진단`, `지체`, `점검`, `조치`, `티켓`, `처리`), 치찰음(`시스템`, `실시간`, `서비스`, `상태`), 숫자 발음(`일 분당 칠십오 건`, `사십이 번`)

---

#### 🎙️ 표준 대본 Option C: 금융·보험 및 VIP 자산관리 서비스 (금융권·엔터프라이즈용)
```text
"반갑습니다, 고객님. 저희 금융 자문 팀을 찾아주셔서 진심으로 감사드립니다. 구글 클라우드의 정교한 음성 합성 기술을 통해 고객님의 재무 목표에 꼭 맞춘 안전하고 전문적인 맞춤형 가이드를 제공해 드리고 있습니다. 실시간 신용카드 거래 내역 모니터링과 해외 외화 송금 예약부터, 오 년 만기 투자 포트폴리오 점검에 이르기까지, 일 년 삼백육십오 일 스물네 시간 내내 고객님의 소중한 자산을 안전하게 보호합니다. 아흔 개가 넘는 정밀 보안 절차를 간소화하여 더욱 안심하고 자산을 관리하실 수 있습니다. 오늘 어떤 금융 서비스를 안내해 드릴까요?"
```
- **글자 수 / 예상 시간**: 74 어절 (공백 포함 289자) / 신뢰감 있는 금융 상담 속도 기준 **약 35 ~ 42초**
- **핵심 음운 학습 포인트**:
   - 마찰음·치찰음(`신용`, `송금`, `스물네 시간`, `소중한`, `자산`, `서비스`), 복합 외래어(`모니터링`, `포트폴리오`), 한자어 수사와 고유어 수사의 교차 발화(`오 년`, `삼백육십오 일`, `스물네 시간`, `아흔 개`)

---

#### ✅ 녹음 전후 5단계 빠른 점검표 (Quick Recording Checklist)

| 단계 | 점검 항목 | 확인 기준 |
| :--- | :--- | :--- |
| **Step 1. 공간 점검** | 조용한 방, 문 닫기, 에어컨/공조기 끄기, 울림 확인 | 손뼉을 한 번 쳤을 때 메아리나 금속성 울림이 남지 않는지 확인 |
| **Step 2. 마이크 점검** | 외장 마이크 연결, 입에서 15~30cm 거리 및 45도 각도 배치 | 5초 테스트 녹음 후 헤드폰으로 팬 소음(Hum)이나 숨소리 파열음이 없는지 확인 |
| **Step 3. 파일 ① (동의서)** | [2.1절의 공식 동의 문구](#21-파일--음성-동의서-consent-audio--812초-필수-낭독-문구)를 단어 가감 없이 그대로 낭독 | **8~12초** 분량, 앞뒤 인사말 없이 동의 문구만 또렷하게 녹음되었는지 확인 |
| **Step 4. 파일 ② (샘플)** | 같은 자리·같은 마이크로 [2.2절의 표준 대본](#22-파일--참조-음성-샘플-voice-sample--2060초-표준-대본-3선) 낭독 | **20-60초** 분량, 긴 침묵 없이 일정한 볼륨과 템포인지 확인 |
| **Step 5. 최종 검토** | 두 파일 모두 클리핑 없음(`-12dB ~ -6dB`), 노이즈 없음 확인 | `speaker_01_consent.wav`, `speaker_02_sample.wav`로 각각 저장 |

---

## 3. Step 3: CX Agent Studio 웹 콘솔에서 대상 언어 및 지원 모델 설정하기

오디오 파일 준비가 끝났다면 [CX Agent Studio 콘솔](https://ces.cloud.google.com/)에 접속하여 에이전트가 커스텀 보이스를 사용할 수 있도록 기본 환경을 먼저 확인합니다.

1. **[CX Agent Studio 콘솔](https://ces.cloud.google.com/) 접속 및 에이전트 열기**
   - 상단에서 대상 **GCP 프로젝트(Project)** 와 **리전(Location)** 을 선택하고, 음성을 적용할 **에이전트 애플리케이션(Agent Application)** 을 클릭합니다.
2. **에이전트 설정(⚙️ Settings) 열기**
![alt text](settings_panel.png)
   - 빌더 화면 우측 상단의 **톱니바퀴 아이콘(⚙️ Settings)** 을 클릭하여 **Agent application settings** 패널을 엽니다.
3. **대상 언어 활성화 확인 (`Language controls`)**
   - **`Basic`** 탭 > **`Interactions`** 섹션 > **`Language controls`** 로 이동합니다.
   - **Default language(기본 언어)** 또는 **Additional languages(추가 언어)** 에 커스텀 보이스를 적용할 대상 언어(예: `Korean (ko-KR)`, `English (en-US)` 등)가 추가되어 있는지 확인합니다. (없다면 추가 후 저장합니다.)
   ![alt text](language_controls.png)
4. **보이스 클로닝 지원 모델 확인 (`Model`)**
   - 콘솔에서 `Custom` 보이스 기능은 보이스 클로닝을 지원하는 음성/에이전트 모델이 선택된 경우에만 활성화됩니다.
    ![alt text](model_select.png)
   - **지원 모델 목록**:
     - **Composite 모델 (권장)**: `gemini-composite-v1`, `gemini-composite-v2`

---

## 4. Step 4: `Create custom voice` 화면에서 Custom Voice 생성하기

이제 콘솔 UI에서 준비한 오디오 2종을 등록하고 커스텀 보이스를 생성합니다. 전체 진행 순서는 다음과 같습니다:

1. **`Create custom voice` 패널 열기**: `Settings (⚙️)` > `Basic` > `Behavior` > `Voice` 영역에서 **`Custom voices`** 버튼 또는 **대상 언어 행**을 클릭하여 보이스 생성 패널을 엽니다.
  ![alt text](create_custom_voice.png)
2. **보이스 이름 및 오디오 2종 등록**: 보이스 식별 이름(`Name`, 예: `brand-voice-01`)을 입력하고, **`Upload file` (파일 업로드)** 또는 **`Record audio` (직접 녹음)** 방식으로 **참조 음성 샘플(`Voice sample`, 20~60초)** 과 **음성 동의서(`Consent audio`, 8~12초)** 를 등록합니다.
3. **억양 및 발화 스타일 지시문(`Voice instruction`) 모드 선택 (4.1 & 4.2절 참고)**: **`Generate instructions`** (AI 자동 생성 — 기본값), **`No instructions`** (원본 그대로 복제), **`Add instructions`** (억양/톤 지시문 직접 입력) 중 하나를 선택합니다.
예시는 **`Add instructions`** 진행합니다.
  ![alt text](instruction_select.png)
4. **생성 완료(`Create`) 및 미리듣기(`▶`) 확인**: 프로젝트 내 `<projectId>-ces-voice-samples` 버킷에 오디오가 자동 저장되고, 3대 실시간 생체·보안 검증을 통과하면 커스텀 보이스 생성이 완료됩니다.
  ![alt text](create_custom_voice_done.png)

---

### 4.1. 억양 및 발화 스타일 지시문(`Voice instruction`) 설정하기

오디오 등록 아래의 **`Voice instruction`** 섹션에는 3개의 세그먼트 버튼이 제공됩니다. 원하는 방식을 선택하세요:

| 세그먼트 버튼 옵션 | API 모드 (`voiceInstructionMode`) | 동작 방식 및 설정 가이드 |
| :--- | :--- | :--- |
| **`Generate instructions`**<br/>*(기본값)* | `GENERATE_INSTRUCTION` | • **AI 자동 생성 모드**: 저장 시 백엔드 AI가 업로드된 음성 샘플을 분석하여 화자의 음색·톤·속도 특징을 설명하는 지시문을 자동 생성합니다.<br/>• 생성이 완료된 후 해당 커스텀 보이스를 다시 클릭해 **Edit(편집) 화면**으로 들어가면, AI가 작성한 지시문을 텍스트창에서 확인하고 맞춤형 표현으로 수정할 수 있습니다. |
| **`No instructions`** | `NO_INSTRUCTION` | • **지시문 없음 모드**: 별도의 스타일 프롬프트 없이 업로드한 성우 녹음본의 원래 음색, 억양, 말투를 100% 있는 그대로 복제합니다.<br/>• 성우가 이미 완벽한 상담 톤으로 녹음한 경우 가장 깔끔한 결과를 제공합니다. |
| **`Add instructions`** | `CUSTOM_INSTRUCTION` | • **사용자 직접 입력 모드**: 선택 즉시 하단에 텍스트 입력창이 열리며, 원하는 **지역 억양(Accent), 감정 톤, 발화 속도, 숫자/전문용어 발음 가이드**를 직접 입력할 수 있습니다. |

---

### 4.2. 산업별·상황별 `Voice instruction` 프롬프트 템플릿

**`Add instructions`** 모드를 선택했거나 생성 후 편집(Edit)할 때, 아래 프롬프트 중 서비스 성격에 맞는 문구를 복사하여 입력창에 붙여넣으세요:

| 에이전트 유형 | 복사해서 바로 쓰는 `Voice instruction` 프롬프트 |
| :--- | :--- |
| **① 일반 고객센터 / VIP 상담**<br/>*(따뜻하고 친절한 상담원 톤)* | `"표준 억양을 구사하는 친절하고 전문적인 고객센터 상담원 목소리입니다. 미소를 머금은 듯한 따뜻한 톤(Vocal Smile)으로 말하며, 문장 끝을 부드럽게 맺고 숫자나 요금 안내 시에는 또렷하고 차분한 속도로 발화하세요."` |
| **② 금융 · 보험 · 보안 데스크**<br/>*(차분하고 신뢰감 있는 아나운서 톤)* | `"신뢰감 있고 차분한 금융 전문 자문가의 표준 발음입니다. 과장된 억양 변화 없이 안정적인 중저음 톤과 일정한 페이싱을 유지하며, 약관 안내나 금액·숫자를 말할 때 한 음절씩 명확하게 전달하세요."` |
| **③ 쇼핑 · 리테일 · 예약 컨시어지**<br/>*(밝고 경쾌한 에너지 톤)* | `"밝고 활기찬 에너지가 느껴지는 쇼핑 컨시어지 상담원 목소리입니다. 자연스럽고 친근한 억양으로 경쾌한 리듬감을 유지하며, 고객의 질문에 적극적으로 공감하는 대화 톤으로 발화하세요."` |
| **④ IT 기술 지원 · 헬프데스크**<br/>*(명료하고 침착한 해결사 톤)* | `"침착하고 명료한 기술 지원 엔지니어의 목소리입니다. 고객이 당황하지 않도록 안정감 있는 속도로 말하며, 단계별 안내 사항과 영문/숫자 티켓 번호를 또렷하게 구분하여 발음하세요."` |

---

## 5. Step 5: 대상 언어에 Custom Voice 매핑 및 통화 행동 최적화하기

커스텀 보이스 생성을 마쳤다면, 에이전트가 대화 시 해당 커스텀 보이스로 말하도록 연결하고 통화 옵션을 다듬습니다.

### 5.1. 대상 언어에 생성한 Custom Voice 연결하기
  ![alt text](language_setting.png)
1. **Agent application settings (⚙️)** > **`Basic`** 탭 > **`Behavior`** 섹션의 **`Voice`** 요약 테이블에서 커스텀 보이스를 적용할 **언어 행(예: `Korean (ko-KR)`, `English (en-US)` 등)** 을 클릭합니다.
2. 우측에 열리는 **Voice settings** 패널에서 상단 음성 유형 토글을 **`Built-in`** 에서 **`Custom`** 으로 전환합니다.
3. **Custom voice** 드롭다운 메뉴를 열어 Step 4에서 생성한 커스텀 보이스(예: `brand-voice-01`)를 선택합니다.
4. 하단에 표시되는 **내장 오디오 플레이어(`ces-audio-player`)** 의 재생 버튼을 눌러 최종 음성을 확인하고(재생 속도 조절 및 구간 이동 가능), 패널을 저장합니다.
5. 이제 `Voice` 요약 테이블의 해당 언어 행에 방금 지정한 커스텀 보이스 이름이 표시됩니다.

---

### 5.2. 발화 속도(Tempo), 배경 효과음, 끼어들기(Barge-in), 묵음 대기 설정

실제 전화 상담 환경에서 가장 자연스러운 대화 경험을 제공하기 위해 아래 설정값을 함께 구성하는 것을 권장합니다:

| 설정 탭 및 섹션 | 설정 항목 | 권장 설정값 및 설명 |
| :--- | :--- | :--- |
| `Basic` > `Behavior` | **Speaking style**<br/>*(발화 속도/템포)* | • **권장값**: **`1.0`** (일반 상담) 또는 **`0.95`** (금융/고령층 안내)<br/>• `0.25`(느림) ~ `2.0`(빠름) 사이에서 발화 템포를 조절합니다. |
| `Basic` > `Behavior` | **Ambient sounds**<br/>*(배경 효과음)* | • **권장값**: **`office_1`** 또는 **`room_1`**<br/>• 통화 연결 중 완전한 무음(Dead air)으로 인해 통화가 끊긴 것처럼 느껴지는 현상을 방지합니다. |
| `Basic` > `Behavior` | **Allow user interruptions & Adapt when interrupted** | • **권장값**: **두 항목 모두 활성화 (On)**<br/>• 에이전트가 안내하는 도중 고객이 말을 걸면(Barge-in) 즉시 발화를 멈추고, 고객이 앞선 안내를 끝까지 듣지 못했음을 감안해 다음 답변을 이어갑니다. |
| `Advanced` > `Speech` | **Silence timeout**<br/>*(묵음 대기 시간)* | • **권장값**: **`4s` ~ `5s`**<br/>• 고객이 4~5초간 아무 말도 하지 않을 때 *"고객님, 제 말씀 잘 들리시나요?"* 등 재참여 유도 멘트를 발화합니다. |
| `Advanced` > `Speech` | **Ambient sound volume gain** | • **권장값**: **`-6.0` dB**<br/>• 배경 효과음이 상담원 목소리를 방해하지 않도록 은은하게 깔리도록 게인을 낮춥니다. |
| `Advanced` > `Speech` | **Keypad input (DTMF)** | • **권장값**: **활성화 (Enabled — 기본값)**<br/>• 전화 통화 시 생년월일, 휴대폰 번호, 계좌번호 등의 키패드 버튼 입력을 수신합니다. |

---

## 6. Step 6: 시뮬레이터(Simulator) 실시간 음성 테스트 및 검증

모든 설정을 저장했다면 에이전트 빌더 화면 우측의 **Preview / Simulator** 에서 실제 음성 대화를 테스트합니다.

1. 우측 상단 설정 패널을 닫고 빌더 우측의 **Preview / Simulator** 창을 엽니다.
2. 언어 선택이 테스트할 대상 언어로 되어 있는지 확인한 후, 하단 **마이크 아이콘(🎙️)** 을 클릭하여 음성 모드로 대화를 시작합니다.
3. **최종 품질 점검 체크리스트 5가지**:
   - [ ] **음색 및 억양 확인**: 첫 인사말부터 내장 음성이 아닌 우리가 등록한 **Custom Voice**와 지정한 `Voice instruction` 톤앤매너로 자연스럽게 발화하는가?
   - [ ] **숫자 및 고유명사 발음 확인**: 금액, 날짜, 전화번호, 서비스 명칭 등을 발화할 때 발음이 뭉개지지 않고 명확한가?
   - [ ] **끼어들기(Barge-in) 반응 확인**: 에이전트가 긴 문장을 말할 때 중간에 *"잠시만요, 다른 질문이 있어요"* 라고 끼어들면 즉시 말을 멈추고 자연스럽게 응대하는가?
   - [ ] **묵음 타임아웃(Silence timeout) 확인**: 아무 말도 하지 않고 5초간 기다렸을 때 에이전트가 먼저 자연스럽게 대화를 유도하는가?
   - [ ] **배경음(Ambient sound) 밸런스 확인**: 배경 효과음이 목소리를 가리지 않고 적당한 볼륨(`-6.0 dB`)으로 재생되는가?

---

## 7. 부록 A: CES v1 REST API를 활용한 Custom Voice 자동화 및 운영 주의사항

### A.1. 아키텍처별 내부 동작 및 5일 주기 자동 갱신(Re-mint) 주의사항

커스텀 보이스는 매 대화 턴마다 새로 생성되는 것이 아니라 **저장 시점(Design-time)에 화자 복제 아티팩트를 생성**해 두고 런타임에 즉시 불러와 사용합니다.

| 구분 | Native Audio (`Rev23 A2A`) | Gemini TTS (`Composite V1` / `Composite V2` — A2T) |
| :--- | :--- | :--- |
| **사용 모델** | Audio-to-Audio (A2A) 네이티브 오디오 모델 | `gemini-3.1-flash-tts-preview` / `gemini-3.1-flash-lite-tts-preview` / `Chirp3-HD` |
| **저장되는 데이터** | 음향 임베딩 + 16 kHz 참조 오디오 사본 | 음성 복제 키(`voiceKey`, 7일 TTL) 또는 저장된 보이스 ID(`voice_...`, 마지막 사용 후 1년 유지) + 원본 오디오 사본 |
| **🚨 운영 시 핵심 주의사항** | 최초 생성 후 별도 갱신 불필요 | • 7일 유효기간의 `voiceKey`를 사용하는 환경에서는 백그라운드 작업이 **5일마다 GCS 버킷의 원본 오디오 파일을 읽어 키를 자동 갱신(Re-mint)** 하며, 모델 업데이트 시에도 저장된 원본 오디오를 사용해 보이스 아티팩트를 재생성합니다.<br/>• 따라서 **`<projectId>-ces-voice-samples` 버킷(또는 직접 지정한 GCS 버킷)의 원본 오디오 파일 2종을 절대 삭제하거나 권한을 회수하지 마세요.** |

---

### A.2. Cloud Storage(GCS) 권한 부여 및 REST API (`GET` / `PATCH`) 예제

웹 콘솔 UI 대신 **CES v1 REST API (`ces.googleapis.com`)** 로 직접 지정한 GCS 버킷의 오디오 파일 2종을 연결할 때는 아래 명령어를 사용합니다.

#### ① CES 서비스 에이전트에 GCS 버킷 읽기 권한(`storage.objectViewer`) 부여
```bash
PROJECT_ID="your-project-id"
PROJECT_NUMBER=$(gcloud projects describe "${PROJECT_ID}" --format="value(projectNumber)")
BUCKET_NAME="your-voice-bucket"

gcloud storage buckets add-iam-policy-binding "gs://${BUCKET_NAME}" \
  --member="serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-ces.iam.gserviceaccount.com" \
  --role="roles/storage.objectViewer"
```

#### ② 현재 에이전트 앱의 음성 설정 조회 (`GET`)
```bash
PROJECT_ID="your-project-id"
LOCATION="us"
APP_ID="your-app-id"

curl -s -X GET \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "x-goog-user-project: ${PROJECT_ID}" \
  "https://ces.googleapis.com/v1/projects/${PROJECT_ID}/locations/${LOCATION}/apps/${APP_ID}" \
  | jq '{languageSettings: .languageSettings, audioProcessingConfig: .audioProcessingConfig, defaultChannelProfile: .defaultChannelProfile}'
```

#### ③ Custom Voice 및 통화 설정 일괄 업데이트 (`PATCH`)
```bash
PROJECT_ID="your-project-id"
LOCATION="us"
APP_ID="your-app-id"

curl -s -X PATCH \
  -H "Authorization: Bearer $(gcloud auth print-access-token)" \
  -H "Content-Type: application/json; charset=utf-8" \
  -H "x-goog-user-project: ${PROJECT_ID}" \
  "https://ces.googleapis.com/v1/projects/${PROJECT_ID}/locations/${LOCATION}/apps/${APP_ID}?updateMask=languageSettings,audioProcessingConfig,defaultChannelProfile" \
  -d '{
    "languageSettings": {
      "defaultLanguageCode": "ko-KR",
      "enableMultilingualSupport": true
    },
    "audioProcessingConfig": {
      "synthesizeSpeechConfigs": {
        "ko-KR": {
          "voice": "brand-voice-01",
          "speakingRate": 1.0,
          "model": "gemini-3.1-flash-tts-preview"
        }
      },
      "customVoiceSamples": [
        {
          "name": "brand-voice-01",
          "consentAudioGcsUri": "gs://your-voice-bucket/samples/speaker_01_consent.wav",
          "voiceSampleGcsUri": "gs://your-voice-bucket/samples/speaker_02_sample.wav",
          "voiceInstructionMode": "CUSTOM_INSTRUCTION",
          "voiceInstruction": "표준 억양을 구사하는 친절하고 전문적인 고객센터 상담원 목소리입니다. 미소를 머금은 듯한 따뜻한 톤(Vocal Smile)으로 말하며, 문장 끝을 부드럽게 맺고 숫자나 요금 안내 시에는 또렷하고 차분한 속도로 발화하세요.",
          "previewText": "안녕하세요, 고객님! 오늘 어떤 업무를 먼저 도와드릴까요?"
        }
      ],
      "bargeInConfig": {
        "bargeInAwareness": true
      },
      "inactivityTimeout": "5s",
      "ambientSoundConfig": {
        "prebuiltAmbientSound": "office_1",
        "volumeGainDb": -6.0
      }
    },
    "defaultChannelProfile": {
      "disableDtmf": false,
      "disableBargeInControl": false,
      "noiseSuppressionLevel": "moderate"
    }
  }' | jq .
```

---
