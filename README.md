# 레이아웃 이미지 학습 데이터 추출 파이프라인

## 개요
이 Jupyter 노트북은 색상 기반 객체 탐지를 사용하여 행사 배치도 이미지에서 학습용 데이터를 추출합니다.

## 주요 기능

### 1. 10가지 명확한 색상 범위
각 색상은 HSV 색공간에서 명확히 구분되도록 정의되어 오류 없이 추출 가능합니다:

| ID | 클래스 | 색상 | HSV 범위 |
|----|--------|------|----------|
| 0 | booth (부스) | 빨간색 | H:0-10, 160-180 |
| 1 | stage (무대) | 파란색 | H:100-130 |
| 2 | exit (출구) | 보라색 | H:130-160 |
| 3 | entrance (입구) | 핑크색 | H:160-180, S:30-100 |
| 4 | restroom (화장실) | 녹색 | H:40-80 |
| 5 | food_area (음식구역) | 청록색 | H:80-100 |
| 6 | seating (객석) | 노란색 | H:20-30 |
| 7 | system (시스템) | 주황색 | H:10-20 |
| 8 | boundary (경계선) | 갈색 | H:10-20, S:100-255 |
| 9 | other (기타) | 회색 | S:0-30 |

### 2. 완전한 파이프라인
```
이미지 로드 → 색상 필터링 → 객체 탐지 → 데이터 추출 → JSON 저장 → 이미지 재구성 → 비교
```

### 3. 출력 형식
- **JSON**: 구조화된 데이터 (좌표, 크기, 클래스 정보)
- **YOLO**: 객체 탐지 학습용 어노테이션
- **이미지**: 재구성된 레이아웃
- **통계**: 클래스별 추출 결과

## 사용 방법

### 1. 환경 설정
```python
# 필수 패키지 설치 (첫 실행 시)
install_packages()
```

### 2. 이미지 경로 설정
```python
IMAGE_PATH = '/path/to/your/layout/image.png'
```

### 3. 실행
노트북의 모든 셀을 순서대로 실행하면 됩니다.

### 4. 결과 확인
`output/` 디렉토리에 다음 파일들이 생성됩니다:
- `extracted_layout.json` - 추출된 객체 데이터
- `reconstructed_layout.png` - 재구성된 이미지
- `comparison.png` - 원본과 재구성 비교
- `comparison_detailed.png` - 통계 포함 비교
- `extraction_statistics.png` - 클래스별 통계 차트
- `annotations.txt` - YOLO 형식 어노테이션
- `yolo_dataset/` - YOLO 학습용 데이터셋 구조

## 색상 범위 조정

특정 클래스의 추출 정확도를 개선하려면 `COLOR_RANGES` 딕셔너리의 HSV 범위를 조정하세요:

```python
COLOR_RANGES['booth']['hsv_ranges'] = [
    {'lower': np.array([0, 120, 120]), 'upper': np.array([10, 255, 255])},
]
```

### 색상 마스크 디버깅
```python
visualize_color_mask(IMAGE_PATH, 'booth')
```

## 데이터 구조

### JSON 출력 예시
```json
{
  "metadata": {
    "source_image": "path/to/image.png",
    "layout_size": {
      "width": 120.0,
      "height": 48.0,
      "unit": "meters"
    },
    "num_objects": 81
  },
  "objects": [
    {
      "class_id": 0,
      "class_name": "booth",
      "shape": "rectangle",
      "x": 96.18,
      "y": 43.98,
      "width": 10.97,
      "height": 7.65
    }
  ]
}
```

### YOLO 형식 예시
```
0 0.801500 0.916250 0.091417 0.159375
1 0.489000 0.312500 0.125000 0.208333
```

## 장점

1. **오류 없는 색상 구분**: HSV 색공간에서 명확히 분리된 10개 색상
2. **자동화**: 수동 라벨링 없이 자동으로 데이터 추출
3. **검증 가능**: 재구성 이미지로 추출 정확도 확인
4. **다양한 출력**: JSON, YOLO, 통계, 시각화 등
5. **확장 가능**: 새로운 색상/클래스 쉽게 추가

## 주의사항

1. 입력 이미지는 명확한 색상으로 구분되어야 합니다
2. 조명이나 그림자로 인한 색상 변화는 HSV 범위 조정으로 해결
3. 너무 작은 객체는 노이즈로 필터링됩니다 (area < 100 pixels)
4. 실제 레이아웃 크기(미터)를 정확히 입력해야 합니다

## 다음 단계

1. 생성된 JSON 데이터를 검토하고 검증
2. 색상 범위 조정으로 정확도 개선
3. 여러 이미지에 적용하여 데이터셋 구축
4. YOLO 모델 학습 (선택사항)

## 문제 해결

### Q: 특정 색상이 잘 추출되지 않아요
A: `visualize_color_mask()` 함수로 마스크를 확인하고 HSV 범위를 조정하세요.

### Q: 너무 많은 작은 객체가 추출됩니다
A: `LayoutExtractor.extract_class()` 메서드의 `area < 100` 값을 증가시키세요.

### Q: 객체 경계가 부정확해요
A: 모폴로지 연산의 iteration 값을 조정하세요:
```python
mask = cv2.morphologyEx(mask, cv2.MORPH_CLOSE, kernel, iterations=3)
```

## 라이선스
MIT License
