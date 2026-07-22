# AI-RAG-PROJECT

정부·공공 RFP(입찰공고) 문서를 대상으로 한 RAG(Retrieval-Augmented Generation) 기반 검색 시스템입니다.
서로 다른 포맷(PDF, HWP, HTML, XML)으로 흩어진 문서를 하나의 검색 가능한 데이터로 통합하고, 검색·생성 파이프라인을 구축하는 것을 목표로 팀 프로젝트로 진행했습니다.

이 저장소는 팀 공용 저장소(AI-Sprint-Team3/AI-RAG-Team3)에서 개인 기록·포트폴리오 용도로 옮겨온 개인 저장소입니다.

> 이 저장소에는 팀 프로젝트의 전체 코드(생성 단계, UI 등 다른 팀원 담당 파트 포함)가 그대로 담겨 있습니다. 아래 "My Contribution" 항목에 명시된 부분이 본인이 직접 작업한 범위이며, 그 외 코드는 팀원들의 작업물입니다.

## My Contribution

- **기간**: 2025.09.11 – 2025.10.02
- **팀 내 역할**: 데이터 전처리 파이프라인 담당

### 담당 업무 (팀 공식 역할)

- 정부·공공 RFP 문서(PDF·HWP·HWPX·HTML·XML) 100건의 원본 수집 상태를 메타데이터(CSV)와 대조 검증(누락·중복 0건)
- pyhwp(hwp5txt/hwp5html) 기반 HWP → HTML/XML 변환 파이프라인 설계·구현
- PyMuPDF(fitz) 기반 PDF 파싱(570페이지, 파싱 실패 0건), pdfplumber 기반 표 영역 별도 추출
- 소스별(xml/html/ocr/pdf) 텍스트를 우선순위(xml > html > ocr > pdf) 기준으로 병합하는 통합 스키마 설계, 섹션 구조 보존을 위한 클린 작업
- OCR 포함 처리 시 약 5시간이 소요되는 병목을 발견해 MD5 기반 캐싱 도입, 이후 해시 충돌 가능성을 인지해 SHA3로 교체
- 섹션 우선 청킹(800자/200자 오버랩 슬라이딩 윈도우) 파이프라인 구현
- 병합 데이터 41건 본문 누락 이슈를 필드 스키마 불일치 관점에서 진단, 이후 릴리즈에서 110건 전량 해소 확인
- SHA256 해시 기반 무결성 검증, MANIFEST 기반 정본(CANON) 관리 등 데이터 릴리즈 프로세스에 기여
- 정제 완료된 고품질 텍스트셋(문서 110건, 약 476만 자)을 팀에 릴리즈해 이후 임베딩·검색 단계로 이관

### 개인 탐색 (팀 공식 결과와 별개, 자체 실험)

팀 역할과 별개로, 평가 질의 5개짜리 자체 테스트셋을 구성해 아래 실험을 진행했습니다.

- OpenAI `text-embedding-3-small`과 한국어 특화 임베딩 모델 3종(ko-sroberta-multitask, bge-m3-korean, Upstage)을 비교 실험
- MMR·Multi-Query·Cross-Encoder 리랭킹 3가지 검색 전략을 자체 평가셋으로 정량 비교
  - Cross-Encoder: 필드 회수율 200%(최고), 컨텍스트 토큰 4,060(최대, 비용 가장 큼)
  - MMR: 필드 회수율 160%, 문서 다양성 최고(4.0), 발주기관 등 본문형 필드에 강함
  - Multi-Query: 필드 회수율 120%(최저), 컨텍스트 토큰 1,867(최소, 최저비용)
  - 결론: "MMR 기본 + 계약방식/예산 등 표 기반 질의는 Cross-Encoder 조건부 전환" 하이브리드 전략 설계
- 검색 성능 저하(코사인 유사도 약 0.5, 목표 0.8)의 원인을 헤더·푸터 중복, 청크 길이 불균형, OCR 노이즈로 규명
- HWPX 포맷을 4개 문서로 별도 테스트했으나, 최종 파이프라인에는 XML+HTML+PDF 조합만 채택

## 파이프라인 요약

```
원본 수집·검증 → 포맷 변환(HWP→HTML/XML) → 개별 소스 파싱(PDF/표)
→ 통합 스키마 병합 → 클린 작업 → 섹션 우선 청킹 → 임베딩·인덱싱
→ 릴리즈(정본 관리) → 검색 전략 실험(개인)
```

## 기술 스택

Python, PyMuPDF, pyhwp, pdfplumber, OCR, OpenAI Embedding, FAISS, Cross-Encoder, Git

<details>
<summary><h3>프로젝트 상세 정보 (목적 · 파일 구조 · 설치 및 실행 · 팀 기술 스택)</h3></summary>

### 목적

- 정부 발주 사업 문서(RFP)를 RAG 기반으로 검색·요약·QA할 수 있는 시스템 구축
- 사용자가 방대한 문서에서 필요한 정보를 빠르게 찾고 이해할 수 있도록 지원

### 핵심 과제

- HWP 등 한글 문서의 구조적 정보(표, 문단)를 손실 없이 추출
- 단순 키워드 검색을 넘어 문맥을 이해하는 시맨틱 검색 구현
- LLM을 활용한 신뢰도 높은 요약 및 답변 생성

### 파일 구조

```
rag-project/
├── config/
│   ├── settings.py            # 공통 환경설정 (API 키, DB 설정, 모델 선택 등)
│   └── path.py                # 경로 관련
│
├── data/
│   ├── raw/                   # 원본 데이터 (pdf, hwp, csv 등)
│   │   ├── data_list.csv
│   │   └── files/
│   │
│   ├── processed/             # 전처리된 데이터 (JSON, 청크 단위 텍스트)
│   └── embeddings/            # 생성된 벡터 인덱스 저장 (FAISS, Chroma)
│
├── src/
│   ├── loaders/                # 데이터 로딩 관련
│   ├── preprocessing/          # 텍스트 전처리 & 청킹
│   ├── embeddings/             # 임베딩 처리 & Vector DB
│   ├── retrieval/              # 검색 관련
│   ├── generation/             # LLM 관련
│   ├── evaluation/             # 성능 평가
│   └── pipelines/              # End-to-End 파이프라인
│
├── notebooks/                  # 실험용 Jupyter 노트북
├── streamlit/                  # Streamlit 관련
│   └── app.py                  # 메인 UI
│
├── requirements.txt            # Python 의존성
└── main.py                     # CLI 엔트리포인트
```

주의: `data/raw`는 용량 문제로 `.gitignore` 처리되어 있으며, 전처리된 데이터는 팀 공용 구글 드라이브로 공유됩니다.

### 설치 및 실행

도커 실행:

```bash
sudo docker system prune -a --volumes -f
docker compose pull
docker compose build --no-cache rag-app
docker compose up -d
docker compose ps
```

Streamlit UI 실행:

```bash
streamlit run streamlit/app.py
```

파이프라인 단독 실행:

```bash
python -m src.pipelines.llm_pipeline
```

### 팀 기술 스택

Python 3.10+, OpenAI API, LangChain, ChromaDB/FAISS, Streamlit

</details>

## 트러블슈팅 하이라이트

- **필드 누락 진단**: `docs_merged.jsonl`에서 41건이 본문 누락으로 확인되어, 필드 스키마 후보(texts.merged / merged_text / chars.merged)를 모두 점검하는 진단 스크립트를 작성해 원인을 추적하고 이후 릴리즈에서 해소 확인
- **캐시 전략 개선**: OCR 포함 처리 병목(약 5시간)을 캐싱으로 해결하면서, MD5의 해시 충돌 가능성을 인지하고 SHA3로 교체
- **릴리즈 무결성 관리**: SHA256 해시로 정본 파일의 변조 여부를 검증하고, MANIFEST.csv로 정본/참고용 파일을 구분해 다운스트림 팀에 인수인계

## 상세 작업 기록

이 저장소의 [`data/`](../../tree/main/data) 폴더가 위 "My Contribution"에 명시된 작업의 결과물입니다.

## 팀 정보

- PM: 노준혁
- Data: 김하나, 전수현
- Retrieval: 노준혁
- Generation: 백예나
- Evaluation: 조민수
- UI/Infra: 백예나, 노준혁

원본 팀 저장소: [AI-Sprint-Team3/AI-RAG-Team3](https://github.com/AI-Sprint-Team3/AI-RAG-Team3)

## 보고서 및 발표자료

- 발표자료(수정본): [RAG_기반_챗봇_서비스.pdf](https://drive.google.com/file/d/1TiU4ibGvlsDzf4_YjGhkVQWDgBz3-m3E/view?usp=drive_link)
