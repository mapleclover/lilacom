# 행사장 배치도 안전 검증 솔루션 - 시스템 다이어그램

## 1. 전체 서비스 흐름도 (High-Level)

```mermaid
flowchart TB
    subgraph INPUT["📥 입력"]
        A[행사장 배치도<br/>이미지 파일]
        B[안전제한사항<br/>PDF 문서]
    end

    subgraph PROCESS["⚙️ 처리"]
        direction TB

        subgraph IMG["이미지 분석 모듈"]
            C1[객체 인식<br/>부스/무대/출구/소화전 등]
            C2[수치 추출<br/>좌표/거리/면적/높이]
        end

        subgraph PDF["PDF 파싱 모듈"]
            D1[텍스트 추출]
            D2[검증 항목 구조화]
        end

        E[(구조화된 데이터<br/>JSON/DB)]

        subgraph AI["AI 검증 엔진"]
            F1[규정별 검증 로직]
            F2[위반사항 탐지]
        end
    end

    subgraph OUTPUT["📤 출력"]
        G[검증 결과 리포트<br/>- 통과/미통과 항목<br/>- 위반 사유<br/>- 개선 권고사항]
    end

    A --> IMG
    B --> PDF
    IMG --> E
    PDF --> E
    E --> AI
    AI --> G
```

## 2. 상세 데이터 흐름도

```mermaid
flowchart LR
    subgraph STEP1["STEP 1: 데이터 수집"]
        A1[고객 업로드:<br/>배치도 이미지]
        A2[고객 업로드:<br/>안전규정 PDF]
    end

    subgraph STEP2["STEP 2: 데이터 변환"]
        B1["이미지 → 구조화 데이터<br/>─────────────────<br/>• 부스: {id, x, y, width, height}<br/>• 무대: {id, x, y, height}<br/>• 출구: {id, x, y, type}<br/>• 통로: {id, width, path}"]

        B2["PDF → 검증 규칙<br/>─────────────────<br/>• 이격거리 규칙<br/>• 높이 제한 규칙<br/>• 위치 제한 규칙<br/>• 필수 설비 규칙"]
    end

    subgraph STEP3["STEP 3: 검증 수행"]
        C1["각 규칙 × 각 객체<br/>순회 검증"]
    end

    subgraph STEP4["STEP 4: 결과 생성"]
        D1["✅ 통과 항목"]
        D2["❌ 미통과 항목<br/>+ 위반 사유<br/>+ 해당 위치"]
    end

    A1 --> B1
    A2 --> B2
    B1 --> C1
    B2 --> C1
    C1 --> D1
    C1 --> D2
```

## 3. 이미지 분석 상세 흐름

```mermaid
flowchart TB
    subgraph INPUT["배치도 이미지"]
        IMG[이미지 파일<br/>PNG/JPG/CAD Export]
    end

    subgraph PREPROCESS["전처리"]
        P1[이미지 정규화]
        P2[노이즈 제거]
        P3[스케일 보정<br/>축척 계산]
    end

    subgraph DETECT["객체 탐지"]
        D1[부스 영역 인식]
        D2[무대 영역 인식]
        D3[출입구/비상구 인식]
        D4[소화전/소화기 인식]
        D5[통로 영역 인식]
        D6[기둥/벽체 인식]
    end

    subgraph MEASURE["수치 계산"]
        M1[각 객체 좌표 추출]
        M2[객체 간 거리 계산]
        M3[객체 크기/면적 계산]
        M4[통로 폭 계산]
    end

    subgraph OUTPUT["출력"]
        O1["구조화된 배치 데이터<br/>{objects, distances, areas}"]
    end

    IMG --> PREPROCESS
    PREPROCESS --> DETECT
    DETECT --> MEASURE
    MEASURE --> O1
```

## 4. 검증 엔진 상세 흐름

```mermaid
flowchart TB
    subgraph INPUT["입력"]
        I1[구조화된 배치 데이터]
        I2[검증 규칙 목록]
    end

    subgraph RULES["검증 규칙 예시"]
        R1["이격거리 검증<br/>─────────────────<br/>• 부스 ↔ 기존설치물: 3m 이상<br/>• 부스 ↔ 벽체: 1m 이상<br/>• 통로 폭: 3m 이상"]

        R2["높이 제한 검증<br/>─────────────────<br/>• 부스 높이: 12m 미만<br/>• 무대 높이 5m 초과 시<br/>  → 구조계산 필요 플래그"]

        R3["위치 제한 검증<br/>─────────────────<br/>• 비상구 전면 노출 확인<br/>• 소화전 접근성 확인<br/>• 취식구역 홀 제한 확인"]
    end

    subgraph ENGINE["검증 엔진"]
        E1[규칙 로더]
        E2[객체별 규칙 매칭]
        E3[조건 평가]
        E4[결과 집계]
    end

    subgraph OUTPUT["검증 결과"]
        O1["결과 JSON<br/>─────────────────<br/>{<br/>  passed: [...],<br/>  failed: [<br/>    {rule, object, reason}<br/>  ],<br/>  warnings: [...]<br/>}"]
    end

    I1 --> ENGINE
    I2 --> ENGINE
    R1 -.-> ENGINE
    R2 -.-> ENGINE
    R3 -.-> ENGINE
    ENGINE --> O1
```

## 5. 개발 로드맵 (1차 → 2차)

```mermaid
flowchart LR
    subgraph PHASE1["🎯 1차 개발"]
        direction TB
        P1A[배치도 이미지 분석]
        P1B[안전규정 PDF 파싱]
        P1C[정적 검증 엔진]
        P1D[검증 결과 리포트]

        P1A --> P1C
        P1B --> P1C
        P1C --> P1D
    end

    subgraph PHASE2["🚀 2차 개발"]
        direction TB
        P2A[시간대별 프로그램<br/>데이터 입력]
        P2B[유동인구<br/>데이터 입력]
        P2C[인파 집중<br/>시뮬레이션]
        P2D[AI 최적화 제안<br/>• 부스 위치 조정<br/>• 프로그램 순서 변경]

        P2A --> P2C
        P2B --> P2C
        P2C --> P2D
    end

    PHASE1 --> PHASE2
```

## 6. 시스템 컴포넌트 구성 (개발팀용)

```mermaid
flowchart TB
    subgraph CLIENT["클라이언트"]
        WEB[웹 인터페이스]
    end

    subgraph API["API 서버"]
        REST[REST API<br/>Spring Boot / FastAPI]
    end

    subgraph SERVICES["서비스 레이어"]
        S1[이미지 분석 서비스<br/>Python + OpenCV/YOLO]
        S2[PDF 파싱 서비스<br/>Python + pdfplumber]
        S3[검증 엔진 서비스<br/>규칙 기반 + AI]
        S4[리포트 생성 서비스]
    end

    subgraph STORAGE["저장소"]
        DB[(데이터베이스<br/>PostgreSQL/MySQL)]
        FILE[파일 스토리지<br/>S3/MinIO]
        CACHE[캐시<br/>Redis]
    end

    subgraph AI["AI/ML"]
        MODEL[객체 탐지 모델<br/>YOLO/Faster R-CNN]
        LLM[LLM<br/>규칙 해석/제안 생성]
    end

    WEB <--> REST
    REST <--> S1
    REST <--> S2
    REST <--> S3
    REST <--> S4
    S1 <--> MODEL
    S3 <--> LLM
    S1 <--> FILE
    S2 <--> FILE
    S3 <--> DB
    S4 <--> DB
    REST <--> CACHE
```

---

## 다이어그램 설명

### 핵심 처리 흐름
1. **입력**: 고객이 배치도 이미지와 안전규정 PDF를 업로드
2. **이미지 분석**: 배치도에서 객체(부스, 무대, 출구 등) 인식 + 좌표/거리/면적 수치 추출
3. **PDF 파싱**: 안전규정 문서에서 검증 항목을 구조화된 규칙으로 변환
4. **AI 검증**: 추출된 데이터를 모든 규칙에 대해 검증
5. **출력**: 통과/미통과 항목과 위반 사유를 포함한 리포트 생성

### 기술 스택 제안
- **이미지 분석**: Python + OpenCV + YOLO/Faster R-CNN
- **PDF 파싱**: Python + pdfplumber/PyPDF2
- **백엔드 API**: Spring Boot 또는 FastAPI
- **AI/LLM**: 규칙 해석 및 개선 제안 생성
- **데이터베이스**: PostgreSQL/MySQL
- **파일 저장**: S3 또는 MinIO
