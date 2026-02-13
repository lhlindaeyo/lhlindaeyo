---
title: "openSMILE: The Munich Versatile and Fast Open-Source Audio Feature Extractor (2010)"
date: 2026-02-10 15:45:00 +0900
categories: [PaperReview]
tags: [finance, NLP, BTM]
math: true
---
**알아둬야 할 개념들**
- **MFCC(Mel-Frequency Cepstral Coefficients)**: 음성 인식 및 오디오 분류에서 가장 널리 사용되는 특징량으로 인간의 청각 시스템이 저주파수 대역의 변화헤는 민감하고 고주파수 대역에는 둔감하다는 특성을 반영
  - 용도: 음성 인식(ASR), 화자 식별, 음악 장르 분류
  - 핵심 원리: 소리의 스펙트럼에서 포먼트(Formant) 구조를 추출하기 위해 로그 에너지값에 이산 코사인 변환(DCT)을 수행하여 복잡한 신호를 단순화
- **CHROMA (Chromagram)**: 인간의 청각이 옥타브 차이가 나는 음들을 유사하게 인지한다는 점에 착안한 특징으로 주파수 스펙트럼을 음악의 12개 반음(C, C#, D, ... B)으로 매핑(harmorny 정보를 추출하는데 탁월)
  - 용도: 음악 검색(Shazam 등), 코드 인식, 곡의 전개 분석
  - 핵심 원리: 모든 옥타브의 'C' 음들을 하나로 합쳐서 표현하므로, 같은 멜로디라면 어떤 악기로 연주하든 유사한 크로마 벡터가 생성
- **CENS (Chroma Energy Normalized Statistics)**: 크로마 특징을 개선한 버전으로 소리의 짧은 시간 변화나 연주 스타일(템포, 강약)의 차이를 극복하기 위해 통계적 처리를 더함
  - 용도: 같은 곡의 다른 버전(커버곡, 라이브 공연) 찾기, 오디오 정렬
  - 핵심 원리: 크로마 벡터에 Smoothing와 Quantization 과정을 거쳐 에너지를 정규화
- **LPC (Linear Predictive Coding)**: 인간의 발성 모델을 수학적으로 모델링한 기법으로 성대에서 난 소리가 목과 입술을 거쳐 변하는 과정을 필터로 해석
  - 용도: 음성 합성(TTS), 초창기 휴대전화 음성 코덱, 포먼트 분석
  - 핵심 원리: "현재의 신호 샘플은 과거 샘플들의 선형 결합으로 예측 가능하다"는 가정하에 선형 필터의 계수를 산출
- **PLP (Perceptual Linear Prediction)**: MFCC와 LPC의 장점을 섞은 개념으로 인간의 심리 음향학적 모델(청취 특성)을 LPC 계산 과정에 도입
  - 용도: 화자 독립 음성 인식 시스템
 - 핵심 원리: 등감도 곡선(Equal-loudness) 적용 및 소리의 크기-강도 변환(Intensity-loudness power law)을 거친 후 선형 예측 분석을 수행
- **기본 주파수 (F0, Fundamental Frequency)**: 신호의 주기적인 파동 중 가장 낮은 주파수를 의미하며, 우리가 흔히 느끼는 pitch
 - 특징: 성대의 진동 횟수와 직결(남성은 낮고, 여성과 아이는 높음)
 - 용도: 억양 분석, 감정 인식, 가창 분석, 성별 판별
 - 핵심 원리: 시간 영역에서는 Autocorrelation를 통해 주기를 찾거나, 주파수 영역에서 Harmonics 사이의 간격을 측정하여 계산

 **비교 테이블**

 | 특징량 | 주요 분석 대상 | 핵심 키워드 | 주요 특징 및 용도 |
| :--- | :--- | :--- | :--- |
| **MFCC** | 음색, 음성 | 멜 스케일(Mel Scale), 범용성 | 인간의 청각 특성을 반영하여 저주파 대역에 집중. 음성 인식의 표준. |
| **CHROMA** | 화성, 멜로디 | 12개 반음, 음악 분석 | 옥타브를 무시하고 12개 반음으로 매핑. 코드 인식 및 화성 분석에 특화. |
| **CENS** | 곡의 유사성 | 변동성 억제, 통계적 정규화 | 크로마 특징에 통계적 처리를 더해 연주 스타일이나 템포 차이를 극복. |
| **LPC** | 발성 구조 | 선형 예측, 음성 압축 | 성대의 물리적 모델을 수학적으로 재현. 음성 합성 및 데이터 압축에 유리. |
| **PLP** | 음성 인식 | 심리 음향 모델, 잡음 강인성 | MFCC와 유사하나 심리 음향학적 모델을 더 강화하여 화자 차이를 보정. |
| **F0** | 음높이(Pitch) | 성대 진동, 주성분 주파수 | 소리의 가장 기본이 되는 주파수. 억양, 성별 판별, 감정 분석에 필수적. |

## Abstract
openSMILE(Speech & Music Interpretation by Large-space Extraction)은 음성 처리와 음악 정보 검색(MIR) 커뮤니티의 특징 추출 알고리즘을 통합한 툴킷이다.

## Introduction
자동 음성 인식(ASR), 비언어적 음성 분석, 음악 정보 검색(MIR)에서 특징 추출은 필수적이다.

그러난 기존 도구들은 특정 도메인(ex. ASR, MIR)에 편중되어있거나 오프라인 처리에만 초점을 맞추는 한계가 존재한다.

따라서 연구 단계에서 사용된 특징들을 실제 라이브 데모 시스템이나 상용 애플리케이션에서도 동일하게 사용할 수 있도록 실시간 증분 처리를 지원하기 위해 openSMILE을 개발했다.
> **증분처리(Incremental Processing)**: 실시간으로 오디오가 입력되는 즉시 처리 하기 위해, 파일용량이 크고 시계열데이터인 오디오 데이터를 처리하기 위해, 대량으로 처리하기 위해 필요한 것으로 사용예시는 다음과 같다<br>
> 1. Stream 생성: 전체 파일을 읽지 않고 제너레이터(Generator) 형식으로 조금씩 읽는다
> 2. Windowing: 20~40ms 단위로 겹치게(Overlap) 자른다
> 3. Accumulation: 추출된 특징량(MFCC 등)을 기존 벡터나 DB에 누적 업데이트

## Related Toolkits
- 음성 연구 도구: HTK, PRAAT, SFS, SNACK
  - HTK (Hidden Markov Model Toolkit): 은닉 마르코프 모델(HMM)을 사용하여 음성 인식기를 구축하기 위해 개발된 소프트웨어 툴로 음성 신호의 특징 추출(MFCC 등)부터 모델 학습, 평가까지 음성 인식의 전 과정을 지원
  - PRAAT: 언어학자 및 음성 과학자들이 가장 많이 사용하는 무료 소프트웨어로 음성 신호를 시각화(스펙트로그램)하고 Pitch(F0), Formant, Intensity 등을 정밀하게 측정
  - SFS (Speech Filing System): 음성 과학 연구를 위해 고안된 통합 컴퓨팅 환경으로 데이터 저장•신호 처리•통계 분석 및 그래픽 표시를 위한 수백 개의 도구 세트를 포함
  - SNACK (Snack Sound Toolkit): 프로그래밍 언어에서 오디오 신호를 처리하기 위해 사용하는 확장 라이브러로 실시간 오디오 입력/출력, 스펙트로그램 시각화, 피치 추출 기능 등을 코드 레벨에서 구현이 용이
- 음악 연구 도구: libXtract, jAudio, Marsyas, MIRtoolbox 
  - libXtract: C언어 기반 특징 추출 라이브러리 약 50여 개의 Low-level 오디오 특징 추출 함수를 제공
  - jAudio: Java 기반 특징 추출 과정을 표준화하고 중복된 노력을 줄이기 위해 개발된 도구
  - Marsyas (Music Analysis, Retrieval and Synthesis for Audio Signals): C++기반 오디오 신호처리 프레임 워크로 단순한 특징 추출을 넘어 신호 처리, 기계 학습, 데이터 변환을 모두 포함하는 거대한 생태계
  - MIRtoolbox: MATLAB 환경에서 동작하는 오디오 특징 추출 및 음악 분석용 툴박스

## Modeling

openSMILE은 Data Memory를 중심으로 모든 구성요소가 연결된 형태를 가지는데, 이때 Data Memory는 Ring-Buffer방식으로 구현되었다.

> Ring-Buffer: 메모리 효율성을 위해 오래된 데이터는 새로운 데이터로 덮어쓰는 구조

![openSMILE 아키텍처](/assets/img/posts/openSMILE_1.png)

전체적인 흐름은 **Data Source → Data Memory → Data Processor → Data Memory → Data Sink**이다.

구체적인 파일 처리 방식인 증분처리 방식은 다음과 같다.
![openSMILE 처리 방법 구체화](/assets/img/posts/openSMILE_2.png)

1. wave 레벨: cWaveSource가 원시 오디오 샘플을 기록
2. frames 레벨: cFramer가 샘플들을 모아 일정 크기의 프레임(예: 윈도우)으로 생성
3. pitch/feature 레벨: cPitch 같은 프로세서가 프레임으로부터 피치나 특징점을 추출
4. func 레벨: 추출된 특징값들의 통계적 수치(최대값, 최소값 등 기능적 특징)를 계산

**openSMILE에서 추출 가능한 특징**<br>
- Audio Low-Level Descriptors: 오디오 신호에서 프레임 단위로 추출되는 특징들
  - Waveform: Zero-Crossings, Extremes, DC오프셋
  - Signal Energy: RMS 에너지, 로그 에너지, Loudness, Intensity
  - Spectral: FFT(크기, 위상), Mel/Bark/Octave 스펙트럼, CSS, Centroid, Variance, Skewness, Kurtosis, Flux, Roll-off 등
  - Cepstral: MFCC, PLP, PLP-CC
  - Pitch & Voice Quality: 기본 주파수, 주파수 변동, 진폭 변동, HNR(Harmonics-to-Noise Ratio), 발성 확률
  - LPC & Formants: LPC 계수, LSP(Line Spectral Pairs), 포먼트 주파수 및 대역폭
  - Tonal: CHROMA, CENS, 코드 미 조성 인식 특징

이 외에도 OpenCV 라이브러리를 통해 비디오 신호에서도 특징을 추출가능 하다. 하지만 오디오 중심으로 알아보고자 해서 생략하고자 한다.

## Conclusion

openSMILE은 효율적이고, 온라인 처리가 가능하며, 다양한 플랫폼을 지원하는 오픈 소스 특징 추출dlek.

해당 모델은 INTERSPEECH 2009 Emotion Challenge의 공식 특징 추출기로 사용되었으며, 감정 인식 툴킷인 openEAR의 기반이 되었다.

해당 모델은 OpenCV와의 연동을 통한 시청각 특징 융합, MPEG-7 출력 지원, 그리고 커뮤니티 기여를 통한 새로운 기능 추가를 목표로 하고 있다.


**참고**<br>

| 비교 항목 | **librosa** | **openSMILe** |
| :--- | :--- | :--- |
| **핵심 철학** | 유연한 신호 처리 및 연구용 (DIY 방식) | 표준화된 고속 특징 추출 (Factory 방식) |
| **기반 언어** | Python (Numpy/Scipy 기반) | C++ (Python 라이브러리 지원) |
| **특징량 추출** | 함수별 개별 추출 (세밀한 제어 가능) | 정의된 Feature Set 통째로 추출 (eGeMAPS 등) |
| **처리 속도** | 중저속 (대량 데이터 시 병목 발생 가능) | 매우 빠름 (C++ 엔진 기반 고속 처리) |
| **실시간성** | 낮음 (파일 단위 Batch 처리 위주) | 높음 (Real-time 스트리밍 분석 설계) |
| **주요 용도** | 음악(MIR), 딥러닝 모델링, 프로토타이핑 | 음성 감정 인식(SER), 부가언어학, 산업용 |

> https://audeering.github.io/opensmile/about.html