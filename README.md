# 0. 프로젝트 개요
## 🎯 목적
- 정부 발주 사업 문서(RFP)를 RAG 기반으로 검색/요약/QA 할 수 있는 시스템 구축
- 사용자가 방대한 문서에서 필요한 정보를 빠르게 찾고 이해할 수 있도록 지원
## 🔑 핵심과제
- HWP 등 **한글 문서의 구조적 정보(표, 문단)**를 손실 없이 추출
- 단순 키워드 검색을 넘어 문맥을 이해하는 시맨틱 검색 구현
- LLM을 활용한 신뢰도 높은 요약 및 답변 생성
## 📏 아키텍처
`데이터 수집 → 전처리/구조화 → 임베딩 → 검색 → 생성`
## 💡 기대효과
- 방대한 정부 문서를 빠르게 탐색 가능
- 키워드 기반보다 문맥 기반 정보 탐색 가능
- 요약 및 QA 자동화를 통한 효율성

---

# 1. 파일트리
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
│   ├── loaders/               # 데이터 로딩 관련      
│   ├── preprocessing/         # 텍스트 전처리 & 청킹      
│   ├── embeddings/            # 임베딩 처리 & Vector DB      
│   ├── retrieval/             # 검색 관련      
│   ├── generation/            # LLM 관련      
│   ├── evaluation/            # 성능 평가      
│   └── pipelines/             # End-to-End 파이프라인      
│        
├── notebooks/                 # 실험용 Jupyter 노트북      
├── streamlit/                 # Streamlit 관련      
│   └── app.py                 # 메인 UI      
│        
├── requirements.txt           # Python 의존성        
└── main.py                    # CLI 엔트리포인트        
      
```
> ⚠️ 주의사항
> 1. data/raw는 .gitignore 처리 (용량 문제)
> 2. 전처리된 데이터는 공용 구글 드라이브 공유
> 3. .gitkeep 파일은 제거

---

# 2. 설치 및 실행 방법

## 1) 도커 실행
```
sudo docker system prune -a --volumes -f
docker compose pull
docker compose build --no-cache rag-app
docker compose up -d
docker compose ps
```
## 2) 실행 (추후 업데이트)
**Streamlit UI 실행**
```
streamlit run streamlit/app.py
```

** 파이프라인 단독 실행
```
python -m src.pipelines.llm_pipeline
```

---

# 3. 기술 스택
- Python 3.10+
- OpenAI API
- LangChain
- ChromaDB / FAISS
- Streamlit

---

# 4. 팀정보
- PM: 노준혁
- Data: 김하나
- Retrieval: 노준혁
- Generation: 백예나
- Evaluation: 조민수
- UI/Infra: 백예나/노준혁

---

# 5. 향후 계획
- 더 정교한 fallback 전략 추가
- 멀티턴 대화 기능 추가
- streamlit ui 개선
- ...

---

# 6. 협업 툴
- 팀 노션: https://lateral-pram-01c.notion.site/26cdf0f96e6d80bc9f81d421c9734fd1
- 협업일지: https://www.notion.so/26cdf0f96e6d80d6b3bece895da98d51?v=26cdf0f96e6d80ada1a9000cd06fe97c

---
# 7. 보고서(발표자료)
파트별 정리 사항: https://www.notion.so/277df0f96e6d806db567e60a2a95570c                  
발표자료: [3팀_최종발표자료_rag.pdf](https://github.com/user-attachments/files/22635108/3._._rag.pdf)
발표자료(수정본): [3팀_발표자료_최종.pdf](https://github.com/user-attachments/files/22653200/3._._.pdf)


