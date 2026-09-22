### 안녕하세요, 정현빈입니다

음성인식 모델 학습·평가와 챗봇 개발을 1년 2개월 해 왔고, 측정해서 판단하는 백엔드·AI 시스템을 만듭니다.

#### 대표 프로젝트

**[support-agent](https://github.com/hyeonbin123/support-agent)**: 한국어 고객센터 업무(주문 조회·취소, 반품·교환 접수, 배송지 변경, 보상 쿠폰, 상담원 이관)를 도구 호출로 처리하는 LLM 상담 에이전트와, 그 에이전트가 얼마나 믿을 만한지를 미리 정한 규칙으로 재는 평가 환경

- τ-bench 방식: 고객 역할은 LLM 시뮬레이터, 판정은 LLM이 아니라 "끝난 뒤의 DB 상태가 정답 동작만 실행한 상태와 같은가". 같은 과제를 4번씩 시켜 pass^k와 규정 위반 수를 냄. 로컬 7B는 시험용 과제의 16.2%만 끝까지 처리했고, 개선 후보 5개는 모두 기준을 넘지 못해 "개선 없음"으로 기록
- 고객의 말이 음성 합성 → 음성 인식을 거치면 성공률이 1.9%로 떨어지고(글자 오류율은 7.6%지만 주문 번호·이메일이 한 번도 그대로 전달되지 않음), 표기 규칙 네 개로 8.8%까지 되찾음
- 같은 에이전트를 웹 채팅 서비스(SSE, PostgreSQL, 감사 로그, 큰 환불의 사람 승인 대기열), MCP 서버, 음성 채널로 확장. 테스트 1,090개, CI에서 실제 PostgreSQL 통합 테스트, DAST 스캔
- Python, FastAPI, SQLAlchemy 2.0, Alembic, PostgreSQL, Ollama, MCP, MeloTTS, faster-whisper, Docker Compose, GitHub Actions

**[voice-translator](https://github.com/hyeonbin123/voice-translator)**: 영어↔한국어 음성 번역 웹 서비스. 말하면 음성 인식 → 번역 → 음성 합성을 거쳐 원문·번역문과 번역 음성을 돌려줌

- 대화 모드, 동시통역(WebSocket으로 말하는 동안 자막 갱신), 두 사람 대화(한 마디마다 한국어·영어 자동 판별)
- 모델은 모두 로컬 GPU에서 돌리고, 공개 평가 데이터(FLEURS)로 후보를 측정해서 고름. 10초 음성 → 번역 음성까지 중앙값 0.89초
- 비동기 FastAPI, WebSocket, PostgreSQL, React + TypeScript, faster-whisper, CTranslate2, Docker Compose(GPU)

**[rag-doc-qa](https://github.com/hyeonbin123/rag-doc-qa)**: FastAPI 공식 문서(영어·한국어)에 질문하면 출처가 붙은 답변을 돌려주는 RAG API 서버

- 비동기 FastAPI, PostgreSQL + pgvector, SQL로 구현한 BM25, cross-encoder 재정렬, Docker Compose, GitHub Actions CI
- 검색·생성 방식을 바꿀 때마다 질문셋으로 측정하고, 기본값은 측정 전에 정한 규칙으로 판단함 (실험 v1~v10)
- 한국어 질문은 한국어 번역 문서에서 찾아 한국어로 답함 (언어별 임베딩 모델과 벡터 인덱스)

**[whisper-ko-ft](https://github.com/hyeonbin123/whisper-ko-ft)**: 공개 한국어 음성 데이터(Zeroth-Korean)로 Whisper를 파인튜닝하고, 미리 정한 규칙으로 전후를 측정한 프로젝트

- whisper-small 전체 파인튜닝과 whisper-large-v3-turbo LoRA. turbo + LoRA로 같은 도메인 CER 4.48% → 1.96% (test, 한 번 측정)
- 좋아진 것만이 아니라 잃은 것도 잼: 다른 도메인(FLEURS)은 허용 폭을 넘게 나빠져 "도메인 전용"으로 판정. 파인튜닝한 모델에서만 드물게 나오는 되풀이 오류를 추론 엔진의 재시도로 막을 수 있다는 것도 확인
- 나빠진 원인이 숫자 표기 차이("5월"과 "오 월")임을 확인하고 학습 정답의 표기를 바꿔 다시 학습: 같은 이득을 지키면서 다른 도메인 CER 6.94% → 5.55%(기준선 5.21%), 판정이 "범용으로 쓸 수 있다"로 바뀜
- PyTorch, Hugging Face Transformers, PEFT(LoRA), faster-whisper, 부트스트랩 신뢰구간, GitHub Actions

**[bike-demand](https://github.com/hyeonbin123/bike-demand)**: 서울 따릉이 대여소별 시간당 대여 수를 예측하고, 곧 자전거가 부족해질 대여소를 지도로 보여 주는 데이터 파이프라인·서비스

- 대여이력 1억 4천만 건을 dbt-duckdb로 집계, Airflow가 실시간 대여정보(10분)와 단기예보를 모아 앞으로 48시간을 예측, FastAPI + 지도 대시보드
- 후보와 판정 규칙을 측정 전에 커밋하고 시간 순서로 검증함. 검토에서 찾은 학습 데이터 누수 두 가지를 고쳐 다시 측정 (test MAE 1.044, 기준선 1.185)
- Airflow, dbt, DuckDB, PostgreSQL, LightGBM, FastAPI, Docker Compose

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

- 백엔드: Python, FastAPI, Django, Flask, SQLAlchemy 2.0(async), PostgreSQL + pgvector, MySQL, WebSocket, SSE, MCP, Docker, GitHub Actions
- 음성·AI: Whisper 파인튜닝·평가(전체, LoRA), PEFT, faster-whisper, CTranslate2, MeloTTS, PyTorch, TensorFlow, Hugging Face Transformers, sentence-transformers, LangChain, Ollama, LLM 도구 호출 에이전트와 τ-bench 방식 평가
- 데이터: Airflow, dbt, DuckDB, LightGBM
- 프론트엔드: React, TypeScript

#### 기타 경험

- 주식회사 엘젠 (2023.01 ~ 2023.12): 음성인식 학습 데이터 전사
