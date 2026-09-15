### 안녕하세요, 정현빈입니다

음성인식 모델 학습·평가와 챗봇 개발을 1년 2개월 해 왔고, 측정해서 판단하는 백엔드·AI 시스템을 만듭니다.

#### 대표 프로젝트

**[voice-translator](https://github.com/hyeonbin123/voice-translator)**: 영어↔한국어 음성 번역 웹 서비스. 말하면 음성 인식 → 번역 → 음성 합성을 거쳐 원문·번역문과 번역 음성을 돌려줌

- 대화 모드, 동시통역(WebSocket으로 말하는 동안 자막 갱신), 두 사람 대화(한 마디마다 한국어·영어 자동 판별)
- 모델은 모두 로컬 GPU에서 돌리고, 공개 평가 데이터(FLEURS)로 후보를 측정해서 고름. 10초 음성 → 번역 음성까지 중앙값 0.89초
- 비동기 FastAPI, WebSocket, PostgreSQL, React + TypeScript, faster-whisper, CTranslate2, Docker Compose(GPU)

**[rag-doc-qa](https://github.com/hyeonbin123/rag-doc-qa)**: FastAPI 공식 문서(영어·한국어)에 질문하면 출처가 붙은 답변을 돌려주는 RAG API 서버

- 비동기 FastAPI, PostgreSQL + pgvector, SQL로 구현한 BM25, cross-encoder 재정렬, Docker Compose, GitHub Actions CI
- 검색·생성 방식을 바꿀 때마다 질문셋으로 측정하고, 기본값은 측정 전에 정한 규칙으로 판단함 (실험 v1~v10)
- 한국어 질문은 한국어 번역 문서에서 찾아 한국어로 답함 (언어별 임베딩 모델과 벡터 인덱스)

#### 경력

- 주식회사 엘젠 (2024.01 ~ 2025.02, 1년 2개월): 음성인식·챗봇·자연어처리 개발
  - Whisper 한국어 파인튜닝 전담, 음성인식 모델의 학습 데이터 정제·학습·평가
  - AICC(AI 콜센터) 통화 음성 인식 서버 유지보수, 추가 학습한 Whisper로 모델 교체
  - RAG 기반 문서 QA 챗봇 프로토타입 개발, 공공 과제 챗봇 개발 참여
  - 녹취·전사 웹 유지보수, 음성 분할·전사 서버 개발 참여

#### 그 밖의 프로젝트

- [mjc](https://github.com/hyeonbin123/mjc): 이미지를 올리면 YOLOv10으로 객체를 찾고 한국어 LLM이 설명을 쓰는 Streamlit 웹앱 (2024, 학교 과제)
- [CordingTest](https://github.com/hyeonbin123/CordingTest): 백준·프로그래머스 문제 Python 풀이

#### 기술

- 백엔드: Python, FastAPI, Django, Flask, SQLAlchemy 2.0(async), PostgreSQL + pgvector, MySQL, WebSocket, Docker, GitHub Actions
- 음성·AI: Whisper 파인튜닝·평가, faster-whisper, CTranslate2, PyTorch, TensorFlow, Hugging Face Transformers, sentence-transformers, LangChain, Ollama
- 프론트엔드: React, TypeScript

#### 기타 경험

- 주식회사 엘젠 (2023.01 ~ 2023.12): 음성인식 학습 데이터 전사
