### 안녕하세요, 정현빈입니다

음성인식 모델 학습·평가와 챗봇 개발을 1년 2개월 해 왔고, 측정해서 판단하는 백엔드·AI 시스템을 만듭니다.

#### 대표 프로젝트

**[stt-scratch](https://github.com/hyeonbin123/stt-scratch)**: 한국어 음성 인식 모델을 사전학습 가중치 없이 처음부터 설계·학습하고, 마이크 입력에 부분 자막을 돌려주는 스트리밍 서비스로 묶은 프로젝트. 단계마다 가설과 판정 기준을 측정 전에 커밋하고 test는 마지막에 한 번만 잼

- PyTorch로 직접 구현한 21M 청크 인과 Conformer-CTC(RoPE·QK-norm, 분할 depthwise conv, 동적 청크 학습, 캐시 스트리밍). 공개 데이터 51.8 h(Zeroth-Korean·FLEURS·Common Voice 22), RTX 2080 Ti 한 장 fp16. Zeroth 공식 test 문장의 99.8%가 Zeroth train에 있어 문장 단위 홀드아웃을 따로 둠
- "fp16 발산"처럼 보이던 학습 정지는 CTC 손실 코드 버그 2개, 정렬이 생기지 않던 것은 처음부터 건 SpecAugment가 원인임을 진단으로 확인해 커리큘럼으로 고침(따로 관측한 어텐션 로짓 폭주에는 QK-norm). 같은 업데이트 step 수로 맞춘 구조 비교에서 E-Branchformer·CR-CTC는 채택 기준을 넘지 못함
- 봉인 test: Zeroth 공식 test CER 3.67%로 whisper-large-v3-turbo 제로샷(4.48%)과 LoRA(1.96%) 사이. 다만 학습 세트에 있는 문장 1.85%, 처음 보는 문장 16.01%로 문장 암기와 일관됨. 처음 보는 문장만 모은 test_H 15.55%, FLEURS 한글 전용 14.62%, Common Voice 28.19%
- 스트리밍 서버(FastAPI WebSocket): 실제 캐시 스트리밍 = 시뮬레이션(0.03포인트 안), 패킷 송신 → 부분 자막 p50 62 ms(청크 대기 제외), 끝점 확정 EP50 1.3 s. GPU 계산 목표 50 ms를 못 지킨 것(p90 63 ms, 커널 실행 호출이 병목인 것으로 보임)과 첫 요청 0.5 s 지연을 예열로 고친 것을 기록. OWASP ZAP으로 찾은 보안 헤더 누락을 고침
- 봉인 test 뒤 후속(dev만): 끝점 뒤 전체 문맥 2차 패스로 확정 자막 CER 26.86% → 22.85%, 끝점 규칙 0.8 s로 EP50 1,295 → 905 ms, CUDA Graph로 청크당 GPU 계산 p90 72.5 → 43.0 ms(같은 부하 조건에서 다시 잰 eager 대비, 출력 비트 단위로 같음)를 재서 서버 기본값을 바꿈. 봉인 test 수치와 6단계 판정은 그대로
- Codex 읽기 전용 독립 검토 일곱 번 84건 반영, 테스트 109개, GitHub Actions
- PyTorch(CUDA Graph), torchaudio, Conformer-CTC, Montreal Forced Aligner, FastAPI, WebSocket, GitHub Actions, OWASP ZAP

**[ride-insight](https://github.com/hyeonbin123/ride-insight)**: 사이클링 파워미터 데이터(FIT 파일)를 받아 훈련 지표(NP·IF·TSS, 파워 커브, CTL/ATL/TSB)를 계산하고 FTP를 추정하는 웹 서비스. 지표 구현의 정확성, FTP 추정 방법, 변조 FIT 처리를 미리 정한 규칙으로 재고, AI 코딩 도구(Claude Code, Codex)가 만든 코드를 어떻게 검증했고 무엇이 잡혔는지를 기능마다 기록한 프로젝트

- 서버는 Kotlin(Spring Boot 4, JdbcClient + Flyway, PostgreSQL), 모델 실험은 Python, 화면은 React + D3. FIT 디코더는 Garmin SDK의 라이선스가 공개 저장소에 맞지 않아 직접 구현하고 python-fitparse의 디코딩 결과(레코드 31,156개)와 대조하는 테스트를 둠(버그 2개를 이 테스트가 잡음)
- 직접 구현한 지표를 GoldenCheetah OpenData(CC0, 선수 6,613명 중 표본 800명)의 값과 대조: 봉인 세트(선수 100명, 라이딩 32,907개 중 1초 기록 31,150개)에서 NP 99.61%, 16구간 MMP 99.65%가 1 W 안, TSS 99.97%(GoldenCheetah가 실제로 쓴 임계값으로 다시 계산, 정수·부재 TSS 제외)가 0.5 안에서 일치. 라이딩의 10~13%는 GoldenCheetah가 IF·TSS에 CP 설정이 아닌 존 FTP를 썼고 정수 TSS는 수동 입력으로 추정되어, 이 둘을 구분해야 구현 차이가 보인다는 것을 기록
- FTP 추정(봉인 test 선수 123명, 예측 단위 1,586개): 흔한 규칙 "20분 최대 × 0.95"는 선수가 설정한 FTP와 MAE 29.2 W [25.3, 33.6]이고 17 W 낮게 치우침(과대 추정한다는 가설 기각). dev에서 고른 Ridge는 22.9 W로 규칙보다 6.3 W [−9.1, −3.0] 가까웠지만 미리 정한 "5 W 이상 개선"을 구간으로 확정하지 못해 규칙대로 서비스는 규칙을 쓰고 오차를 함께 보여 줌(test에서 더 좋았던 LightGBM 20.6 W도 채택하지 않음)
- 변조 FIT 7종(크기 필드 조작, CRC, 정의 없는 데이터, 역행 타임스탬프, 10 MB 초과)이 모두 미리 적은 상태 코드를 돌려주고 500 없음. OWASP ZAP 1차가 존재하지 않는 경로에 500을 돌려주던 예외 처리기 버그와 보안 헤더 누락을 잡아 고쳤고, 2차는 High·Medium 0(5xx 12% → 0%). 동시 업로드 10개 p95 0.33~0.64 s
- 공개 뒤 다시 검증(기존 테스트·Codex 검토·첫 ZAP이 놓친 것): 4 MB FIT 하나가 힙 821 MB(입력의 205배)를 붙잡아 힙 512 MB 서버를 메모리 부족(500)으로 만드는 것을 측정으로 확인하고 고침(담는 필드 제한, 열 저장, record 50만 개 상한 413). 고정 seed 퍼징에서 범위 밖 심박·거리·timestamp를 받아들이던 계약 위반 56건(HTTP로 재현한 500은 1건), 내부 버그를 400으로 가리던 예외 매핑, UTC로 세던 날짜를 고침. 동시 업로드 처리 상한(2개)은 미리 등록한 메모리 지표로는 가설 기각이라 그대로 기록. FIT 업로드를 포함한 전체 스택 ZAP의 CSP unsafe-inline(Medium)을 요청별 nonce로 고쳐 High·Medium 0
- Codex 읽기 전용 검토 세 번 35건(봉인 규칙의 누수 허용, NaN 전파, 체인 블록별 메시지 상한, 신선 라벨 정의 불일치, 측정 조건 위반 등)을 반영. 테스트 Kotlin 75·Python 35·화면 11개, Docker Compose 전체 스택, GitHub Actions
- Kotlin, Spring Boot, JdbcClient, Flyway, PostgreSQL, Testcontainers, Python, scikit-learn, LightGBM, React, TypeScript, D3, MUI, React Query, Zustand, Docker Compose, GitHub Actions, OWASP ZAP

**[defect-inspect](https://github.com/hyeonbin123/defect-inspect)**: 정상 제품 사진만으로 만드는 외관 검사기(이상 탐지)를, 현장에서 실제로 굴릴 때 부딪히는 질문(결함 라벨이 몇 장 필요한가, 정상 이미지로 정한 임계값이 지켜지나, 합성 교란으로 실제 조명 변화를 대신할 수 있나, CPU로 충분한가)으로 미리 정한 규칙에 따라 잰 프로젝트

- PatchCore(WideResNet-50·DINOv2 특징, 코어셋)를 직접 구현해 Dinomaly, 고정한 DINOv2 특징 위에 결함 k장으로 학습한 지도 학습 헤드와 비교. 봉인 테스트(VisA 공식 분할의 test 4,328장)는 단계마다 정해 둔 것만 한 번 재고, 읽을 때마다 기록을 남김. 이미지 AUROC는 PatchCore 89.3(WRN-50)·94.1(DINOv2), Dinomaly 96.8. 지도 학습은 범주당 결함 5장(검증 포함 25장)이면 Dinomaly와 구별되지 않고 20장(검증 포함 40장)부터 분명히 앞서지만(+1.6%p), 학습에 없던 결함 유형만 보면 2.1~3.5%p 뒤짐(k = 5·10)
- 정상 이미지만으로 정한 임계값(목표 오검출률 5%): 빼 둔 정상으로 잡으면 실제 5.6%로 유한표본 이론 구간(3.6~6.3%) 안, 교차 적합은 4.0%, 뱅크를 만든 이미지로 잡으면 86%. 단 촬영 조건이 그대로일 때만 그렇고, PatchCore(WRN-50)는 실제 조명이 바뀌면(M2AD) 97%가 됨(새 조명의 정상 30장을 뱅크에 더하고 임계값을 다시 잡으면 4.4%). VisA 이미지를 20~30% 어둡게 하면 AUROC는 그대로인데 오검출률이 15~41%로 뛰어 AUROC 감시로는 보이지 않음
- "합성 교란은 실제 조명 변화의 절반에도 못 미친다"는 가설은 기각(가장 센 흐림·JPEG의 오검출 증가가 실제의 85% 안팎). 그러나 조명을 흉내 낸 밝기 배율(+14%p)과 감마(+52%p)는 실제 조명(모든 조건 +78%p 이상)에 못 미쳐, 밝기 교란만으로는 실제 조명 변화를 어림할 수 없음(M2AD 2개 범주, 규칙 밖의 해석). DINOv2 계열은 밝기 배율에는 흔들리지 않는 대신 256px 기준 1px 어긋남에 오검출 31.9%(PatchCore)·54.2%(Dinomaly)
- CPU 장당 200ms: PatchCore로는 기준을 못 지킴(가설 기각). 규칙대로 고른 구성(WRN-50 256px, 코어셋 1%)이 배포용 뱅크로 214ms(고를 때 쓴 dev 뱅크로는 184ms)이고 AUROC는 Dinomaly보다 6.8%p 낮음. INT8 정적 양자화는 점수를 중앙값 24% 바꿔 쓰지 않음. 처음 잰 지연은 다른 프로그램이 돌던 때여서, dev 결과를 본 뒤 규칙을 고쳐(재측정 전 커밋) 다시 쟀고 고른 구성이 바뀐 것도 기록함
- 이어서 Dinomaly의 인코더를 DINOv2 ViT-S로 줄이고(쓰지 않는 블록 제거, 입력 280px 고정) ONNX로 내보낸 모델을 같은 형식의 규칙으로 잼: onnxruntime CPU에서 AUROC 96.5(GPU Dinomaly ViT-B 96.8), 목표 5%에서 실제 오검출률 4.3%, 장당 150ms로 세 기준을 모두 지켜 검사 서비스 모델을 바꿈(결함 95%를 잡는 지점의 오검출률은 17.6%로 GPU 모델보다 높음). 같은 ONNX를 OpenVINO로 돌리면 35% 빨랐지만(부하 조건, dev만) 미리 정한 대로 onnxruntime 유지
- 조명이 바뀌었을 때의 운영 절차(M2AD): 기준선 PatchCore가 서비스의 `/calibrate`처럼 새 조명의 정상 30장으로 임계값만 다시 잡으면 오검출률은 3.2%로 돌아오지만 검출률이 18.6%로 결함 대부분을 놓침(서빙 구성은 21~54%). 결함 2장이 섞인 묶음에서 점수 상위 4장만 사람이 확인하고 등록하면 검출률 46.6%(거르지 않으면 21.3%). 서빙 모델 구성(Dinomaly ViT-S)도 기준 조명으로만 학습하면 다른 조명에서 오검출률 92.3%(조명 4~5개를 함께 학습해도 34.5%, 같은 구성을 M2AD로 학습한 값)라, README 운영 안내를 "조명이 바뀔 수 있는 라인이면 이 모델은 그대로 쓸 수 없다"로 고침. 사진 통계로 만든 재촬영 게이트(dev만)는 결함 사진도 20% 걸러 가설 기각
- 검사 서비스: torch 없이 onnxruntime CPU + FastAPI, Docker 이미지 약 650MB, 시작할 때 ONNX 모델 지문(SHA-256)을 대조해 다르면 뜨지 않음. 모델 교체 전후를 HawkScan·OWASP ZAP으로 스캔해 새 High·Medium 없음, 테스트 1,173개, GitHub Actions(최소 권한, 액션 SHA 고정, pip-audit)
- PyTorch, timm, DINOv2, anomalib(Dinomaly 모델 코드), ONNX Runtime, OpenVINO(비교), FastAPI, Docker, GitHub Actions, pip-audit, OWASP ZAP

**[call-summary](https://github.com/hyeonbin123/call-summary)**: 고객센터 상담 대화 전사를 상담 기록(문의 유형, 처리 결과, 핵심 값, 처리 목록, 후속 조치, 요약)의 JSON으로 바꾸는 소형 LLM을 직접 학습하고, 음성 인식을 거친 전사 조건과 서빙까지 미리 정한 규칙으로 잰 프로젝트

- 공개 데이터가 없어 정답을 먼저 만듦: seed 고정 명세 생성기가 정답을 정하고, 교사 모델(qwen2.5 14B)은 대화만 쓰고, 명세에 없는 번호·금액이 나온 대화는 코드가 버림. 합성 데이터의 점수 부풀림을 보려고 다른 모델이 쓴 대화와 학습에서 뺀 업종을 따로 시험 세트로 둠
- Qwen3-4B QLoRA(RTX 2080 Ti, fp16)로 구조 필드가 모두 맞은 비율을 학습 전 17.3% → 95.7%(학습 안 한 14B 26.0%). 다른 모델이 쓴 대화 90.0%, 학습에 없던 업종 55.0%로 이득이 줄어드는 폭까지 기록. 2026년 기준 같은 크기의 지시 모델(qwen3.5:4b 등)을 프롬프트만으로 붙여도 test-a 18.7%, 새 업종 4.5%로 학습한 모델에 크게 못 미침
- 음성 합성 → 전화 음질 → Whisper를 거친 전사에서는 51.7%로 떨어지고, 학습 데이터 절반을 음성 인식 전사로 바꾸면 80.7%(글 전사 성능은 그대로). 운송장번호를 범위로 읽던 측정 인공물을 고쳐 다시 잰 서빙 모델은 79.3%. 인식기 교체(dev)는 large-v3 등도 차이 구간이 0을 포함하고 Qwen3-ASR은 전화 조건에서 값 보존부터 낮아 채택하지 않음. 인식 오류를 바로잡은 값과 지어낸 값을 나눠 세고, 의심 번호는 응답에 따로 표시함
- GGUF(llama.cpp 양자화) + Ollama + FastAPI 서빙. JSON 스키마 강제 디코딩과 QLoRA 합치기 방식이 품질을 떨어뜨리는 원인을 찾아 고쳐 서비스 점수 = 학습한 모델(dev 94.6%). 부하 시험, Docker, DAST 스캔(처리기까지 닿는지 서버 로그로 확인), 테스트 291개. 학습 코드의 epoch 경계 버그를 고친 뒤 다시 학습해 미리 정한 규칙(짝지은 차이 ±3%p)으로 견주니 dev 차이가 0.0~+0.4%p라 기록 수치와 서빙 모델을 그대로 씀. 요약 판정은 값 확인 + qwen3.5:9b로 다시 보정했지만(kappa 0.603, 대부분 값 확인 몫) 보조 지표로만 씀
- PyTorch, Hugging Face Transformers, PEFT(LoRA·QLoRA), bitsandbytes, llama.cpp, Ollama, FastAPI, MeloTTS, faster-whisper, GitHub Actions

**[support-agent](https://github.com/hyeonbin123/support-agent)**: 한국어 고객센터 업무(주문 조회·취소, 반품·교환 접수, 배송지 변경, 보상 쿠폰, 상담원 이관)를 도구 호출로 처리하는 LLM 상담 에이전트와, 그 에이전트가 얼마나 믿을 만한지를 미리 정한 규칙으로 재는 평가 환경

- τ-bench 방식: 고객 역할은 LLM 시뮬레이터, 판정은 LLM이 아니라 "끝난 뒤의 DB 상태가 정답 동작만 실행한 상태와 같은가". 같은 과제를 4번씩 시켜 pass^k와 규정 위반 수를 냄. 로컬 7B는 시험용 과제의 16.2%만 끝까지 처리했고, 개선 후보 5개는 모두 기준을 넘지 못해 "개선 없음"으로 기록
- 에이전트 모델만 새 세대 4B(qwen3.5:4b)로 바꾸자 같은 서비스 구성의 시험용 성공률이 18.1% → 55.6%가 되어 서비스 기본 모델을 바꿈(과제가 요구하지 않은 쓰기 6 → 13건은 알려진 위험으로 기록). 고객 역할 모델만 바꿔도 같은 에이전트가 58.3% → 78.1%(개발용)라, 수치가 시뮬레이터에 기댄다는 한계도 적음(그 고객 모델로 다시 재도 7B와의 차이는 +54.2%p로 남음, 개발용·보고만). 4B 에이전트의 남은 실패를 겨냥한 가드 4개 중 개발용 조건을 만족한 하나만 시험용에서 재니 +1.9%p [−0.6, +4.4]라 개선을 확인하지 못해 서비스 기본값을 그대로 둠
- 고객의 말이 음성 합성 → 음성 인식을 거치면 성공률이 1.9%로 떨어지고(글자 오류율은 7.6%지만 주문 번호·이메일이 한 번도 그대로 전달되지 않음), 표기 규칙 네 개로 8.8%까지 되찾음. 규칙으로 못 고친 본인 확인은 받아 적는 대신 발신 번호 + 이름으로 바꿔 본인 확인 성공 52.1% → 80.2%(개발용, 에이전트 qwen3.5:4b), 시험용 43.8% → 58.1%(+14.4%p [+6.2, +23.1]), 성공률(pass^1)도 24.4% → 33.1%(+8.8%p [+1.2, +16.9], 구간 아래 끝이 0에 가까움. 송장 번호를 범위로 읽던 음성 합성 입력의 버그를 고쳐 그 에피소드만 다시 잰 값)
- 같은 에이전트를 웹 채팅 서비스(SSE, PostgreSQL, 감사 로그, 큰 환불의 사람 승인 대기열), MCP 서버, 음성 채널로 확장. 테스트 1,802개, CI에서 실제 PostgreSQL 통합 테스트(쓰기 도구의 행 잠금과 두 트랜잭션 경합 다섯 경우), README의 측정 수치를 reports/에서 다시 계산하는 테스트, DAST 스캔(HawkScan·OWASP ZAP)
- Python, FastAPI, SQLAlchemy 2.0, Alembic, PostgreSQL, Ollama, MCP, MeloTTS, faster-whisper, Docker Compose, GitHub Actions

**[voice-translator](https://github.com/hyeonbin123/voice-translator)**: 영어↔한국어 음성 번역 웹 서비스. 말하면 음성 인식 → 번역 → 음성 합성을 거쳐 원문·번역문과 번역 음성을 돌려줌

- 대화 모드, 동시통역(WebSocket으로 말하는 동안 자막 갱신), 두 사람 대화(한 마디마다 한국어·영어 자동 판별)
- 모델은 모두 로컬 GPU에서 돌리고, 공개 평가 데이터(FLEURS)로 후보를 측정해서 고름. 10초 음성 → 번역 음성까지 중앙값 0.89초
- 대화 모드는 192ms 쉼에서 인식·번역·합성을 미리 시작하고 1초 판정에서 확정해, 말을 멈춘 뒤 번역 음성까지 중앙값 1.86 → 1.24초(한국어)·2.25 → 1.43초(영어). 번역 LLM Hy-MT2-1.8B와 음성 합성 Supertonic 3은 미리 정한 규칙(chrF·COMET-22 구간, 다시 인식한 오류율)에서 떨어져 채택하지 않음
- 동시통역 영어 원문의 말 끝 → 번역 음성(validation 중앙값 1.69초)을 단계별로 나눠 보니 최종 처리 중 한국어 합성(약 0.58초)이 가장 크고, 최종 처리가 1초 확정보다 늦게 끝나는 것이 원인. nginx 보안 헤더를 넣고 첫 OWASP ZAP 스캔(로그인 없는 기준 스캔)에서 Medium 2 → 0, 브라우저 접근성 검사를 CI에서 돌림. 테스트 763개
- 비동기 FastAPI, WebSocket, PostgreSQL, React + TypeScript, faster-whisper, CTranslate2, Docker Compose(GPU), Playwright, OWASP ZAP

**[rag-doc-qa](https://github.com/hyeonbin123/rag-doc-qa)**: FastAPI 공식 문서(영어·한국어)에 질문하면 출처가 붙은 답변을 돌려주는 RAG API 서버

- 비동기 FastAPI, PostgreSQL + pgvector, SQL로 구현한 BM25, cross-encoder 재정렬, Docker Compose, GitHub Actions CI
- 검색·생성 방식을 바꿀 때마다 질문셋으로 측정하고, 기본값은 측정 전에 정한 규칙으로 판단함 (실험 v1~v15)
- 생성 모델(Qwen2.5-7B)을 Qwen3.5-9B·Gemma 4 12B·한국어 특화 A.X-4.0-Light(직접 GGUF 변환)와 비교해, 한국어 답 155문항의 한자·가나 섞임 5 → 0, 아껴 둔 테스트 80문항에서 판정 합 352 → 372로 조건을 통과한 A.X로 바꿈. 다만 판정 합의 우위는 같은 계열 판정기(Qwen2.5-7B)의 값이고, Claude 블라인드 라벨 80개와 더 잘 맞는 다른 계열 판정기 둘(가중 kappa 0.72~0.77 대 0.48)로 다시 재면 −13·−1점으로 보이지 않아 한계로 적음. 한 번 탈락한 한국어 임베딩은 새 규칙을 먼저 적고 다시 판정해 바꿈(MRR 0.867 → 0.909). OWASP ZAP으로 찾은 보안 헤더 누락과 500 오류를 고쳐 재스캔 Medium 0, 고친 판이 없는 권고가 붙은 JWT 라이브러리를 PyJWT로 바꾸고 urllib3·mako를 올려 의존성 권고 0
- 같은 답을 다시 판정해도 104문항 중 14개 판정 원문이 달라져 결정 판정을 3회 중앙값으로 바꿈. 답 없는 질문의 거절률(영어 20/20, 한국어 7/20)과 청크 겹침 버그를 고친 뒤의 재측정(한국어 답변이 떨어져 겹침을 끔)까지 기록. 테스트 283개
- 한국어 질문은 한국어 번역 문서에서 찾아 한국어로 답함 (언어별 임베딩 모델과 벡터 인덱스)

**[whisper-ko-ft](https://github.com/hyeonbin123/whisper-ko-ft)**: 공개 한국어 음성 데이터(Zeroth-Korean)로 Whisper를 파인튜닝하고, 미리 정한 규칙으로 전후를 측정한 프로젝트

- whisper-small 전체 파인튜닝과 whisper-large-v3-turbo LoRA. turbo + LoRA로 같은 도메인 CER 4.48% → 1.96% (test, 한 번 측정)
- 좋아진 것만이 아니라 잃은 것도 잼: 다른 도메인(FLEURS)은 허용 폭을 넘게 나빠져 "도메인 전용"으로 판정. 드물게 나오는 되풀이 오류는 재시도 디코딩으로 그 발화만 다시 디코딩해 없앨 수 있다는 것도 확인(그 발화의 다른 오류는 남음)
- 나빠진 원인이 숫자 표기 차이("5월"과 "오 월")임을 확인하고 학습 정답의 표기를 바꿔 다시 학습: 같은 이득을 지키면서 다른 도메인 CER 6.94% → 5.55%(기준선 5.21%), 판정이 "범용으로 쓸 수 있다"로 바뀜
- 다른 계열의 공개 모델 Qwen3-ASR-1.7B도 같은 규칙으로 잼: 학습 없이 다른 도메인 CER은 turbo보다 낮았지만(validation 4.75% → 4.22%, 학습 데이터에 FLEURS가 있었을 수 있음) 한 발화씩 약 6.5배 느렸고, 같은 데이터로 LoRA를 학습하자 같은 도메인 CER은 더 낮았지만(validation 0.92% 대 1.25%) 다른 도메인 이득이 없어 기존 모델을 유지. 그 어댑터의 갱신량을 절반으로 줄여 합치면 다른 도메인 망각은 허용 폭 안으로 돌아왔지만 같은 도메인이 기존 모델보다 높아 다른 도메인용 기준으로만 둠
- 서빙 양자화: 기존 모델을 CTranslate2 int8_float16으로 돌리면 GPU 메모리 −43%(2,402 → 1,377MiB), CER +0.05%p(미리 정한 허용 폭 안, 여유는 거의 없음)라 서빙 기본으로 권함. 판정에 쓴 비교 25개를 화자·문장 블록 부트스트랩으로 다시 계산해도 모두 같았음
- PyTorch, Hugging Face Transformers, PEFT(LoRA), faster-whisper, Qwen3-ASR, 부트스트랩 신뢰구간, GitHub Actions

**[bike-demand](https://github.com/hyeonbin123/bike-demand)**: 서울 따릉이 대여소별 시간당 대여 수를 예측하고, 곧 자전거가 부족해질 대여소를 지도로 보여 주는 데이터 파이프라인·서비스

- 대여이력 1억 4천만 건을 dbt-duckdb로 집계, Airflow가 실시간 대여정보(10분)와 단기예보를 모아 앞으로 48시간을 예측, FastAPI + 지도 대시보드(프로젝트를 마치고 2026-10-03에 수집을 멈춤)
- 후보와 판정 규칙을 측정 전에 커밋하고 시간 순서로 검증함. 검토에서 찾은 학습 데이터 누수 두 가지를 고쳐 다시 측정 (test MAE 1.044, 기준선 1.185)
- 서비스는 관측이 아닌 예보 날씨를 쓰므로 그 영향도 잼: 발표 46개에서 기온 예보 오차 0.7~1.1℃, 관측 날씨로 바꿔 예측하면 시간당 예측의 평균 절대 차이 0.068대, 예보 쪽 예측 평균이 0.9% 높고, 부족 상위 50곳은 99% 겹침(비 안 온 한 주). 실제 정확도는 대여이력이 공개되는 2027년 초에나 확인할 수 있고, 예측 수준이 지난해 같은 시기보다 15% 높을 수 있다는 점을 한계로 기록
- 테스트 288개, GitHub Actions에서 PostgreSQL 서비스 컨테이너를 붙여 DB 테스트까지 실행
- Airflow, dbt, DuckDB, PostgreSQL, LightGBM, FastAPI, Docker Compose, GitHub Actions

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

- 백엔드: Python, FastAPI, Django, Flask, SQLAlchemy 2.0(async), Kotlin, Spring Boot(JdbcClient·Flyway, Testcontainers), PostgreSQL + pgvector, MySQL, WebSocket, SSE, MCP, Docker, GitHub Actions, DAST(OWASP ZAP), 퍼징, pip-audit
- 음성·AI: Whisper 파인튜닝·평가(전체, LoRA), Qwen3-ASR, 소형 LLM 파인튜닝(QLoRA)과 GGUF 서빙, PEFT, faster-whisper, CTranslate2, MeloTTS, PyTorch, TensorFlow, Hugging Face Transformers, sentence-transformers, LangChain, Ollama, LLM 도구 호출 에이전트와 τ-bench 방식 평가, LLM 판정 보정·교차 판정, CTC 음성 인식 모델 처음부터 설계·학습(Conformer, 캐시 스트리밍, CUDA Graph)
- 컴퓨터 비전: 이상 탐지(PatchCore 직접 구현, Dinomaly), DINOv2·WideResNet-50 특징, ONNX Runtime CPU 서빙과 INT8 양자화 평가, OpenVINO(비교), timm
- 데이터: Airflow, dbt, DuckDB, LightGBM, scikit-learn
- 프론트엔드: React, TypeScript, D3, MUI, React Query, Zustand

#### 기타 경험

- 주식회사 엘젠 (2023.01 ~ 2023.12): 음성인식 학습 데이터 전사
