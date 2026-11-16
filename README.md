### 안녕하세요, 정현빈입니다

음성인식 모델 학습·평가와 챗봇 개발을 1년 2개월 해 왔고, 측정해서 판단하는 백엔드·AI 시스템을 만듭니다.

#### 한눈에 보기

| 프로젝트 | 무엇을 만들었나 | 대표 결과(자세한 조건은 아래 항목과 각 저장소에) |
|---|---|---|
| **[korean-voice-ai](https://github.com/hyeonbin123/korean-voice-ai)** (대표) | 한국어 콜센터 음성 AI. 통화 한 건이 지나가는 네 단계에 해당하는 다섯 구성 요소로 이뤄진 프로젝트 | Whisper LoRA 파인튜닝으로 Zeroth test CER 4.48% → 1.96%(낭독 음성, 처음 듣는 화자가 학습에서 본 문장을 읽는 조건. 다른 도메인 FLEURS는 5.21% → 6.94%로 나빠짐). 상담 에이전트의 모델만 바꿔 시험용 40과제 성공률 18.1% → 55.6%(과제가 요구하지 않은 쓰기 호출도 6 → 13건으로 늘어남) |
| **[ride-insight](https://github.com/hyeonbin123/ride-insight)** | 사이클링 파워 데이터로 훈련 지표를 계산하고 FTP를 추정하는 웹 서비스(Kotlin, Spring Boot, React) | 직접 구현한 지표를 GoldenCheetah 값과 대조: 봉인 세트의 1초 기록 라이딩 31,150개에서 NP 99.61%가 1 W 안, TSS 99.97%(GoldenCheetah가 실제로 쓴 임계값으로 다시 계산, 정수·부재 TSS 제외)가 0.5 안에서 일치 |
| **[rag-doc-qa](https://github.com/hyeonbin123/rag-doc-qa)** | FastAPI 공식 문서에 질문하면 출처가 붙은 답을 주는 RAG API(FastAPI, pgvector) | 한국어 임베딩 교체로 검색 MRR 0.867 → 0.909(아껴 둔 한국어 테스트 40문항) |
| **[bike-demand](https://github.com/hyeonbin123/bike-demand)** | 따릉이 대여소별 시간당 대여 수 예측과 부족 대여소 지도(Airflow, dbt, LightGBM) | 대여이력 1억 4천만 건 집계, 서비스 모델 validation MAE 0.927(test는 다시 재지 않음) |
| **[defect-inspect](https://github.com/hyeonbin123/defect-inspect)** | 정상 제품 사진만으로 만드는 외관 검사기(이상 탐지)와 CPU 검사 서비스 | onnxruntime CPU에서 장당 150ms, AUROC 96.5, 목표 5%에서 실제 오검출률 4.3%(촬영 조건이 그대로일 때. 조명이 바뀔 수 있는 라인이면 그대로 쓸 수 없음) |

#### 대표 프로젝트

**[korean-voice-ai](https://github.com/hyeonbin123/korean-voice-ai)**: 한국어 콜센터 음성 AI. 통화 한 건이 지나가는 네 단계에 해당하는 다섯 구성 요소로 이뤄진 프로젝트로, 수치는 측정 규칙을 먼저 커밋한 뒤 잼

통화 한 건의 흐름: 음성 인식(whisper-ko-ft, stt-scratch) → 실시간 음성 계층(voice-translator) → 상담 에이전트(support-agent) → 상담 후처리(call-summary). 통화 대본을 상담 에이전트와 후처리 오프라인 앱에 차례로 넣어 리포트까지 잇는 배선 시연(모델 없음, 품질 수치 아님)도 둠

- whisper-ko-ft: Whisper 파인튜닝. Zeroth test CER(숫자 표기 차이도 오류로 셈): large-v3-turbo 4.48% → LoRA 파인튜닝 1.96%(처음 듣는 화자가 학습에서 본 문장을 읽는 조건). 다른 도메인 FLEURS 한국어 test는 5.21% → 6.94%(나빠졌다. 판정은 "도메인 전용"이고 범용 모델은 따로 골랐다). 테스트(CI, 2026-11-10) 통과 779
- stt-scratch: 처음부터 만든 스트리밍 인식. 1회 잰 봉인 test CER(비스트리밍, 한글 전용 발화, 공백 제외): Zeroth 공식 test 3.67%(문장 대부분이 학습 세트에도 있어 문장 암기 가능성), 처음 보는 문장만 15.55%. 테스트(CI, 2026-11-10) 통과 284
- voice-translator: 음성 번역 서비스. 한국어 인식 CER 4.57%(FLEURS test, 고른 설정만 1회). 대화 모드에서 말을 멈춘 뒤 번역 음성을 다 받기까지(재생 시작 전) 중앙값: 한국어 원문 1.86초 → 1.24초(최종 처리를 미리 시작하기 전 → 후, 실제 서비스 test). 테스트(CI, 2026-11-10) 통과 742(백엔드 pytest), Vitest 184, Playwright 8
- support-agent: 상담 에이전트와 평가 환경. 시험용 40과제 × 4회 pass^1(한 번 시켜 성공한 비율), 규정 위반을 도구에서도 막는 구성: qwen2.5 7B 18.1% → `qwen3.5:4b` 55.6%(에이전트 모델만 바꿈). 과제가 요구하지 않은 쓰기 호출도 6 → 13건으로 늘었다. 테스트(CI, 2026-11-10) 통과 2,160
- call-summary: 상담 후처리 LLM. 가상 대화 exact(구조 필드 모두 일치), 글 전사로만 학습한 Qwen3-4B QLoRA: test-a(같은 생성기) 95.7%, test-b(다른 모델이 쓴 대화) 90.0%, test-c(학습에 없던 업종) 55.0%(서빙 모델은 다름). 테스트(CI, 2026-11-10) 통과 495
- 위 인식 CER은 모두 낭독 음성 데이터(Zeroth-Korean, FLEURS)에서 잰 값
- Python, PyTorch, torchaudio, Conformer-CTC, Hugging Face Transformers, PEFT(LoRA·QLoRA), bitsandbytes, faster-whisper, CTranslate2, llama.cpp, Ollama, MeloTTS, FastAPI, WebSocket, SQLAlchemy 2.0, PostgreSQL, MCP, React + TypeScript, Playwright, Docker Compose, GitHub Actions, OWASP ZAP

#### 직무별 프로젝트

**[ride-insight](https://github.com/hyeonbin123/ride-insight)**: 사이클링 파워미터 데이터(FIT 파일)를 받아 훈련 지표(NP·IF·TSS, 파워 커브, CTL/ATL/TSB)를 계산하고 FTP를 추정하는 웹 서비스. 지표 구현의 정확성, FTP 추정 방법, 변조 FIT 처리를 미리 정한 규칙으로 재고, AI 코딩 도구(Claude Code, Codex)가 만든 코드를 어떻게 검증했고 무엇이 잡혔는지를 기능마다 기록한 프로젝트

- Kotlin, Spring Boot, JdbcClient, Flyway, PostgreSQL, Testcontainers, Python, scikit-learn, LightGBM, React, TypeScript, D3, MUI, React Query, Zustand, Docker Compose, GitHub Actions, OWASP ZAP

<details>
<summary>측정 결과와 검증 기록 펼치기</summary>

- 서버는 Kotlin(Spring Boot 4, JdbcClient + Flyway, PostgreSQL), 모델 실험은 Python, 화면은 React + D3. FIT 디코더는 Garmin SDK의 라이선스가 공개 저장소에 맞지 않아 직접 구현하고 python-fitparse의 디코딩 결과(레코드 31,156개)와 대조하는 테스트를 둠(버그 2개를 이 테스트가 잡음)
- 직접 구현한 지표를 GoldenCheetah OpenData(CC0, 선수 6,613명 중 표본 800명)의 값과 대조: 봉인 세트(선수 100명, 라이딩 32,907개 중 1초 기록 31,150개)에서 NP 99.61%, 16구간 MMP 99.65%가 1 W 안, TSS 99.97%(GoldenCheetah가 실제로 쓴 임계값으로 다시 계산, 정수·부재 TSS 제외)가 0.5 안에서 일치. 라이딩의 10~13%는 GoldenCheetah가 IF·TSS에 CP 설정이 아닌 존 FTP를 썼고 정수 TSS는 수동 입력으로 추정되어, 이 둘을 구분해야 구현 차이가 보인다는 것을 기록
- FTP 추정(봉인 test 선수 123명, 예측 단위 1,586개): 흔한 규칙 "20분 최대 × 0.95"는 선수가 설정한 FTP와 MAE 29.2 W [25.3, 33.6]이고 17 W 낮게 치우침(과대 추정한다는 가설 기각). dev에서 고른 Ridge는 22.9 W로 규칙보다 6.3 W [−9.1, −3.0] 가까웠지만 미리 정한 "5 W 이상 개선"을 구간으로 확정하지 못해 규칙대로 서비스는 규칙을 쓰고 오차를 함께 보여 줌(test에서 더 좋았던 LightGBM 20.6 W도 채택하지 않음)
- 변조 FIT 7종(크기 필드 조작, CRC, 정의 없는 데이터, 역행 타임스탬프, 10 MB 초과)이 모두 미리 적은 상태 코드를 돌려주고 500 없음. OWASP ZAP 1차가 존재하지 않는 경로에 500을 돌려주던 예외 처리기 버그와 보안 헤더 누락을 잡아 고쳤고, 2차는 High·Medium 0(5xx 12% → 0%). 동시 업로드 10개 p95 0.33~0.64 s
- 공개 뒤 다시 검증(기존 테스트·Codex 검토·첫 ZAP이 놓친 것): 4 MB FIT 하나가 힙 821 MB(입력의 205배)를 붙잡아 힙 512 MB 서버를 메모리 부족(500)으로 만드는 것을 측정으로 확인하고 고침(담는 필드 제한, 열 저장, record 50만 개 상한 413). 고정 seed 퍼징에서 범위 밖 심박·거리·timestamp를 받아들이던 계약 위반 56건(HTTP로 재현한 500은 1건), 내부 버그를 400으로 가리던 예외 매핑, UTC로 세던 날짜를 고침. 동시 업로드 처리 상한(2개)은 미리 등록한 메모리 지표로는 가설 기각이라 그대로 기록. FIT 업로드를 포함한 전체 스택 ZAP의 CSP unsafe-inline(Medium)을 요청별 nonce로 고쳐 High·Medium 0
- Codex 읽기 전용 검토 세 번 35건(봉인 규칙의 누수 허용, NaN 전파, 체인 블록별 메시지 상한, 신선 라벨 정의 불일치, 측정 조건 위반 등)을 반영. 테스트 Kotlin 99·Python 94·화면 26개, Docker Compose 전체 스택, GitHub Actions

</details>

**[rag-doc-qa](https://github.com/hyeonbin123/rag-doc-qa)**: FastAPI 공식 문서(영어·한국어)에 질문하면 출처가 붙은 답변을 돌려주는 RAG API 서버

- 비동기 FastAPI, PostgreSQL + pgvector, SQL로 구현한 BM25, cross-encoder 재정렬, Docker Compose, GitHub Actions CI

<details>
<summary>측정 결과와 검증 기록 펼치기</summary>

- 검색·생성 방식을 바꿀 때마다 질문셋으로 측정하고, 기본값은 측정 전에 정한 규칙으로 판단함 (실험 v1~v16. 2026-11-03에 청크 절 경로 버그를 고친 청커로 본 DB를 다시 적재해, v1~v15 수치는 그 전 청크로 낸 값)
- 생성 모델(Qwen2.5-7B)을 Qwen3.5-9B·Gemma 4 12B·한국어 특화 A.X-4.0-Light(직접 GGUF 변환)와 비교해, 한국어 답 155문항의 한자·가나 섞임 5 → 0, 아껴 둔 테스트 80문항에서 판정 합 352 → 372로 조건을 통과한 A.X로 바꿈. 다만 판정 합의 우위는 같은 계열 판정기(Qwen2.5-7B)의 값이고, Claude 블라인드 라벨 80개와 더 잘 맞는 다른 판정기 둘(다른 계열인 Gemma 4 12B와, A.X와 같은 Qwen 계열의 다른 세대인 Qwen3.5-9B. 가중 kappa 0.72~0.77 대 0.48)로 다시 재면 −13·−1점으로 보이지 않아 한계로 적음. 한 번 탈락한 한국어 임베딩은 새 규칙을 먼저 적고 다시 판정해 바꿈(아껴 둔 한국어 테스트 40문항 검색 MRR 0.867 → 0.909). OWASP ZAP으로 찾은 보안 헤더 누락과 500 오류를 고쳐 재스캔 Medium 0, 고친 판이 없는 권고가 붙은 JWT 라이브러리를 PyJWT로 바꾸고 urllib3·mako를 올려 의존성 권고 0
- 같은 답을 다시 판정해도 104문항 중 14개 판정 원문이 달라져 결정 판정을 3회 중앙값으로 바꿈(v15 I3~I5의 1,847문항에서 3회 중앙값이 점수를 한 번도 바꾸지 않아 v16부터 결정 판정은 1회로 하고, 결정 조건의 값이 경계와 1점 이내일 때만 두 번 더 판정해 중앙값을 씀). 답 없는 질문의 거절률(영어 20/20, 한국어 7/20)과 청크 겹침 버그를 고친 뒤의 재측정(한국어 답변이 떨어져 겹침을 끔)까지 기록. 테스트 639개
- 한국어 질문은 한국어 번역 문서에서 찾아 한국어로 답함 (언어별 임베딩 모델과 벡터 인덱스)

</details>

**[bike-demand](https://github.com/hyeonbin123/bike-demand)**: 서울 따릉이 대여소별 시간당 대여 수를 예측하고, 곧 자전거가 부족해질 대여소를 지도로 보여 주는 데이터 파이프라인·서비스

- Airflow, dbt, DuckDB, PostgreSQL, LightGBM, FastAPI, Docker Compose, GitHub Actions

<details>
<summary>측정 결과와 검증 기록 펼치기</summary>

- 대여이력 1억 4천만 건을 dbt-duckdb로 집계, Airflow가 실시간 대여정보(10분)와 단기예보를 모아 앞으로 48시간을 예측, FastAPI + 지도 대시보드(프로젝트를 마치고 2026-10-03에 수집을 멈춤)
- 후보와 판정 규칙을 측정 전에 커밋하고 시간 순서로 검증함. test MAE 1.044(기준선 1.185)는 누수를 고치기 전 첫 모델(v1)의 값이고 예측 평균이 실제보다 14% 높음. test 기간은 v1·v2에서 두 번 봤고(v2 값은 참고용), 검토에서 찾은 학습 데이터 누수 두 가지를 고친 뒤(v3)에는 validation만 다시 재서 서비스 모델 MAE 0.927이고 test는 다시 재지 않음
- 서비스는 관측이 아닌 예보 날씨를 쓰므로 그 영향도 잼: 발표 46개에서 기온 예보 오차 0.7~1.1℃, 관측 날씨로 바꿔 예측하면 시간당 예측의 평균 절대 차이 0.068대, 예보 쪽 예측 평균이 0.9% 높고, 부족 상위 50곳 겹침은 평균 99%·최소 92%(비 안 온 한 주). 실제 정확도는 대여이력이 공개되는 2027년 초에나 확인할 수 있고, 예측 수준이 지난해 같은 시기보다 15% 높을 수 있다는 점을 한계로 기록
- 테스트 288개, GitHub Actions에서 PostgreSQL 서비스 컨테이너를 붙여 DB 테스트까지 실행

</details>

**[defect-inspect](https://github.com/hyeonbin123/defect-inspect)**: 정상 제품 사진만으로 만드는 외관 검사기(이상 탐지)를, 현장에서 실제로 굴릴 때 부딪히는 질문(결함 라벨이 몇 장 필요한가, 정상 이미지로 정한 임계값이 지켜지나, 합성 교란으로 실제 조명 변화를 대신할 수 있나, CPU로 충분한가)으로 미리 정한 규칙에 따라 잰 프로젝트

- PyTorch, timm, DINOv2, anomalib(Dinomaly 모델 코드), ONNX Runtime, OpenVINO(비교), FastAPI, Docker, GitHub Actions, pip-audit, OWASP ZAP

<details>
<summary>측정 결과와 검증 기록 펼치기</summary>

- PatchCore(WideResNet-50·DINOv2 특징, 코어셋)를 직접 구현해 Dinomaly, 고정한 DINOv2 특징 위에 결함 k장으로 학습한 지도 학습 헤드와 비교. 봉인 테스트(VisA 공식 분할의 test 4,328장)는 단계마다 정해 둔 것만 한 번 재고, 읽을 때마다 기록을 남김. 이미지 AUROC는 PatchCore 89.3(WRN-50)·94.1(DINOv2), Dinomaly 96.8. 지도 학습은 범주당 결함 5장(검증 포함 25장)이면 Dinomaly와 구별되지 않고 20장(검증 포함 40장)부터 분명히 앞서지만(+1.6%p), 학습에 없던 결함 유형만 보면 2.1~3.5%p 뒤짐(k = 5·10)
- 정상 이미지만으로 정한 임계값(목표 오검출률 5%): 빼 둔 정상으로 잡으면 실제 5.6%로 유한표본 이론 구간(3.6~6.3%) 안, 교차 적합은 4.0%, 뱅크를 만든 이미지로 잡으면 86%. 단 촬영 조건이 그대로일 때만 그렇고, PatchCore(WRN-50)는 실제 조명이 바뀌면(M2AD) 97%가 됨(새 조명의 정상 30장을 뱅크에 더하고 임계값을 다시 잡으면 4.4%). VisA 이미지를 20~30% 어둡게 하면 AUROC는 그대로인데 오검출률이 15~41%로 뛰어 AUROC 감시로는 보이지 않음
- "합성 교란은 실제 조명 변화의 절반에도 못 미친다"는 가설은 기각(가장 센 흐림·JPEG의 오검출 증가가 실제의 85% 안팎). 그러나 조명을 흉내 낸 밝기 배율(+14%p)과 감마(+52%p)는 실제 조명(모든 조건 +78%p 이상)에 못 미쳐, 밝기 교란만으로는 실제 조명 변화를 어림할 수 없음(M2AD 2개 범주, 규칙 밖의 해석). DINOv2 계열은 밝기 배율에는 흔들리지 않는 대신 256px 기준 1px 어긋남에 오검출 31.9%(PatchCore)·54.2%(Dinomaly)
- CPU 장당 200ms: PatchCore로는 기준을 못 지킴(가설 기각). 규칙대로 고른 구성(WRN-50 256px, 코어셋 1%)이 배포용 뱅크로 214ms(고를 때 쓴 dev 뱅크로는 184ms)이고 AUROC는 Dinomaly보다 6.8%p 낮음. INT8 정적 양자화는 점수를 중앙값 24% 바꿔 쓰지 않음. 처음 잰 지연은 다른 프로그램이 돌던 때여서, dev 결과를 본 뒤 규칙을 고쳐(재측정 전 커밋) 다시 쟀고 고른 구성이 바뀐 것도 기록함
- 이어서 Dinomaly의 인코더를 DINOv2 ViT-S로 줄이고(쓰지 않는 블록 제거, 입력 280px 고정) ONNX로 내보낸 모델을 같은 형식의 규칙으로 잼: onnxruntime CPU에서 AUROC 96.5(GPU Dinomaly ViT-B 96.8), 목표 5%에서 실제 오검출률 4.3%, 장당 150ms로 세 기준을 모두 지켜 검사 서비스 모델을 바꿈(결함 95%를 잡는 지점의 오검출률은 17.6%로 GPU 모델보다 높음). 같은 ONNX를 OpenVINO로 돌리면 35% 빨랐지만(부하 조건, dev만) 미리 정한 대로 onnxruntime 유지
- 조명이 바뀌었을 때의 운영 절차(M2AD): 기준선 PatchCore가 서비스의 `/calibrate`처럼 새 조명의 정상 30장으로 임계값만 다시 잡으면 오검출률은 3.2%로 돌아오지만 검출률이 18.6%로 결함 대부분을 놓침(서빙 구성은 21~54%). 결함 2장이 섞인 묶음에서 점수 상위 4장만 사람이 확인하고 등록하면 검출률 46.6%(거르지 않으면 21.3%). 서빙 모델 구성(Dinomaly ViT-S)도 기준 조명으로만 학습하면 다른 조명에서 오검출률 92.3%(조명 4~5개를 함께 학습해도 34.5%, 같은 구성을 M2AD로 학습한 값)라, README 운영 안내를 "조명이 바뀔 수 있는 라인이면 이 모델은 그대로 쓸 수 없다"로 고침. 사진 통계로 만든 재촬영 게이트(dev만)는 결함 사진도 20% 걸러 가설 기각
- 검사 서비스: torch 없이 onnxruntime CPU + FastAPI, Docker 이미지 약 650MB, 시작할 때 ONNX 모델 지문(SHA-256)을 대조해 다르면 뜨지 않음. 모델 교체 전후를 HawkScan·OWASP ZAP으로 스캔해 새 High·Medium 없음. 그 뒤 2026-10-30·11-07 재스캔에서는 불리언 SQL 주입(40018, `POST /calibrate`) High 1건이 다시 나왔고, SQL이 없는 서비스라 오탐으로 판단해 고치지 않음(통과 조건을 글자 그대로 적용하면 통과가 아님). 테스트 통과 881개(CI, 2026-11-07), GitHub Actions(최소 권한, 액션 SHA 고정, pip-audit)

</details>

#### 경력

- 주식회사 엘젠 (2024.01 ~ 2025.02, 1년 2개월): 음성인식·챗봇·자연어처리 개발
  - Whisper 한국어 파인튜닝 전담, 음성인식 모델의 학습 데이터 정제·학습·평가
  - AICC(AI 콜센터) 통화 음성 인식 서버 유지보수, 추가 학습한 Whisper로 모델 교체
  - RAG 기반 문서 QA 챗봇 프로토타입 개발, 공공 과제 챗봇 개발 참여
  - 녹취·전사 웹 유지보수, 음성 분할·전사 서버 개발 참여

#### 그 밖의 프로젝트

- [mjc](https://github.com/hyeonbin123/mjc): 이미지를 올리면 YOLOv10으로 객체를 찾고 한국어 LLM이 설명을 쓰는 Streamlit 웹앱 (2024, 학교 과제)
- [coding-test](https://github.com/hyeonbin123/coding-test): 백준·프로그래머스 문제 Python 풀이

#### 기술

- 백엔드: Python, FastAPI, Django, Flask, SQLAlchemy 2.0(async), Kotlin, Spring Boot(JdbcClient·Flyway, Testcontainers), PostgreSQL + pgvector, MySQL, WebSocket, SSE, MCP, Docker, GitHub Actions, DAST(OWASP ZAP), 퍼징, pip-audit
- 음성·AI: Whisper 파인튜닝·평가(전체, LoRA), Qwen3-ASR, 소형 LLM 파인튜닝(QLoRA)과 GGUF 서빙, PEFT, faster-whisper, CTranslate2, MeloTTS, PyTorch, TensorFlow, Hugging Face Transformers, sentence-transformers, LangChain, Ollama, LLM 도구 호출 에이전트와 τ-bench 방식 평가, LLM 판정 보정·교차 판정, CTC 음성 인식 모델 처음부터 설계·학습(Conformer, 캐시 스트리밍, CUDA Graph)
- 컴퓨터 비전: 이상 탐지(PatchCore 직접 구현, Dinomaly), DINOv2·WideResNet-50 특징, ONNX Runtime CPU 서빙과 INT8 양자화 평가, OpenVINO(비교), timm
- 데이터: Airflow, dbt, DuckDB, LightGBM, scikit-learn
- 프론트엔드: React, TypeScript, D3, MUI, React Query, Zustand

#### 기타 경험

- 주식회사 엘젠 (2023.01 ~ 2023.12): 음성인식 학습 데이터 전사
