### 안녕하세요, 정현빈입니다

음성인식 모델 학습·평가와 챗봇 개발을 1년 2개월 해 왔고, 측정해서 판단하는 백엔드·AI 시스템을 만듭니다.

#### 대표 프로젝트

**[defect-inspect](https://github.com/hyeonbin123/defect-inspect)**: 정상 제품 사진만으로 만드는 외관 검사기(이상 탐지)를, 현장에서 실제로 굴릴 때 부딪히는 질문(결함 라벨이 몇 장 필요한가, 정상 이미지로 정한 임계값이 지켜지나, 합성 교란으로 실제 조명 변화를 대신할 수 있나, CPU로 충분한가)으로 미리 정한 규칙에 따라 잰 프로젝트

- PatchCore(WideResNet-50·DINOv2 특징, 코어셋)를 직접 구현해 Dinomaly, 고정한 DINOv2 특징 위에 결함 k장으로 학습한 지도 학습 헤드와 비교. 봉인 테스트(VisA 공식 분할의 test 4,328장)는 단계마다 정해 둔 것만 한 번 재고, 읽을 때마다 기록을 남김. 이미지 AUROC는 PatchCore 89.3(WRN-50)·94.1(DINOv2), Dinomaly 96.8. 지도 학습은 범주당 결함 5장(검증 포함 25장)이면 Dinomaly와 구별되지 않고 20장(검증 포함 40장)부터 분명히 앞서지만(+1.6%p), 학습에 없던 결함 유형만 보면 2.1~3.5%p 뒤짐(k = 5·10)
- 정상 이미지만으로 정한 임계값(목표 오검출률 5%): 빼 둔 정상으로 잡으면 실제 5.6%로 유한표본 이론 구간(3.6~6.3%) 안, 교차 적합은 4.0%, 뱅크를 만든 이미지로 잡으면 86%. 단 촬영 조건이 그대로일 때만 그렇고, PatchCore(WRN-50)는 실제 조명이 바뀌면(M2AD) 97%가 됨(새 조명의 정상 30장을 뱅크에 더하고 임계값을 다시 잡으면 4.4%). VisA 이미지를 20~30% 어둡게 하면 AUROC는 그대로인데 오검출률이 15~41%로 뛰어 AUROC 감시로는 보이지 않음
- "합성 교란은 실제 조명 변화의 절반에도 못 미친다"는 가설은 기각(가장 센 흐림·JPEG의 오검출 증가가 실제의 85% 안팎). 그러나 조명을 흉내 낸 밝기 배율(+14%p)과 감마(+52%p)는 실제 조명(모든 조건 +78%p 이상)에 못 미쳐, 밝기 교란만으로는 실제 조명 변화를 어림할 수 없음(M2AD 2개 범주, 규칙 밖의 해석). DINOv2 계열은 밝기 배율에는 흔들리지 않는 대신 256px 기준 1px 어긋남에 오검출 31.9%(PatchCore)·54.2%(Dinomaly)
- CPU 장당 200ms 안에 들면서 정확도 기준을 지키는 구성은 없었음(가설 기각): 규칙대로 고른 PatchCore(WRN-50 256px, 코어셋 1%)가 배포용 뱅크로 214ms(고를 때 쓴 dev 뱅크로는 184ms)이고 AUROC는 Dinomaly보다 6.8%p 낮음. dev에서 200ms 안에 든 다섯 구성도 모두 dev AUROC가 Dinomaly보다 5%p 넘게 낮았음. INT8 정적 양자화는 점수를 중앙값 24% 바꿔 FP32 임계값으로는 오검출 68%, 임계값을 다시 잡아도 AUROC가 7.1%p 떨어지고 속도 이득은 3~10%뿐이라 쓰지 않음. 처음 잰 지연은 다른 프로그램이 돌던 때여서, dev 결과를 본 뒤 규칙을 고쳐(재측정 전 커밋) 다시 쟀고, 고른 구성이 바뀜(DINOv2 252px → WRN-50 256px). 검사 서비스는 torch 없이 onnxruntime CPU + FastAPI, Docker 이미지 약 650MB, DAST 스캔, 테스트 1,050개
- PyTorch, timm, DINOv2, anomalib(Dinomaly 모델 코드), ONNX Runtime, FastAPI, Docker, GitHub Actions

**[call-summary](https://github.com/hyeonbin123/call-summary)**: 고객센터 상담 대화 전사를 상담 기록(문의 유형, 처리 결과, 핵심 값, 처리 목록, 후속 조치, 요약)의 JSON으로 바꾸는 소형 LLM을 직접 학습하고, 음성 인식을 거친 전사 조건과 서빙까지 미리 정한 규칙으로 잰 프로젝트

- 공개 데이터가 없어 정답을 먼저 만듦: seed 고정 명세 생성기가 정답을 정하고, 교사 모델(qwen2.5 14B)은 대화만 쓰고, 명세에 없는 번호·금액이 나온 대화는 코드가 버림. 합성 데이터의 점수 부풀림을 보려고 다른 모델이 쓴 대화와 학습에서 뺀 업종을 따로 시험 세트로 둠
- Qwen3-4B QLoRA(RTX 2080 Ti, fp16)로 구조 필드가 모두 맞은 비율을 학습 전 17.3% → 95.7%(학습 안 한 14B 26.0%). 다른 모델이 쓴 대화 90.0%, 학습에 없던 업종 55.0%로 이득이 줄어드는 폭까지 기록
- 음성 합성 → 전화 음질 → Whisper를 거친 전사에서는 51.7%로 떨어지고, 학습 데이터 절반을 음성 인식 전사로 바꾸면 80.7%(글 전사 성능은 그대로). 인식 오류를 바로잡은 값과 지어낸 값을 나눠 셈
- GGUF(llama.cpp 양자화) + Ollama + FastAPI 서빙. JSON 스키마 강제 디코딩과 QLoRA 합치기 방식이 품질을 떨어뜨리는 원인을 찾아 고쳐 서비스 점수 = 학습한 모델(dev 94.6%). 부하 시험, Docker, DAST 스캔, 테스트 106개
- PyTorch, Hugging Face Transformers, PEFT(LoRA·QLoRA), bitsandbytes, llama.cpp, Ollama, FastAPI, MeloTTS, faster-whisper, GitHub Actions

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
- 서비스는 관측이 아닌 예보 날씨를 쓰므로 그 영향도 잼: 발표 46개에서 기온 예보 오차 0.7~1.1℃, 관측 날씨로 바꿔 예측하면 시간당 예측이 평균 0.068대(0.9%) 달라지고 부족 상위 50곳은 99% 겹침(비 안 온 한 주). 실제 정확도는 대여이력이 공개되는 2027년 초에나 확인할 수 있고, 예측 수준이 지난해 같은 시기보다 15% 높을 수 있다는 점을 한계로 기록
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
- 음성·AI: Whisper 파인튜닝·평가(전체, LoRA), 소형 LLM 파인튜닝(QLoRA)과 GGUF 서빙, PEFT, faster-whisper, CTranslate2, MeloTTS, PyTorch, TensorFlow, Hugging Face Transformers, sentence-transformers, LangChain, Ollama, LLM 도구 호출 에이전트와 τ-bench 방식 평가
- 컴퓨터 비전: 이상 탐지(PatchCore 직접 구현, Dinomaly), DINOv2·WideResNet-50 특징, ONNX Runtime CPU 서빙과 INT8 양자화 평가, timm
- 데이터: Airflow, dbt, DuckDB, LightGBM
- 프론트엔드: React, TypeScript

#### 기타 경험

- 주식회사 엘젠 (2023.01 ~ 2023.12): 음성인식 학습 데이터 전사
