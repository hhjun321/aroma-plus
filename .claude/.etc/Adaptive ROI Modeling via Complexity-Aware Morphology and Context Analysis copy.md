# AROMA: Adaptive ROI Modeling via Complexity-Aware Morphology and Context Analysis

---

## 1. 연구의 패러다임 전환 (Conceptual Shift)

기존의 규칙 기반 ROI 엔지니어링(CASDA 등)이 가진 도메인 확장성 한계를 극복하기 위해, AROMA는 **데이터 구동형 ROI 정책 학습(Data-Driven ROI Policy Learning)** 구조를 제안합니다.

* **규칙에서 메타 학습으로:** 데이터셋의 가우시안분포 등을 예단하지 않고, 내재적 복잡도를 분석하여 클러스터링 알고리즘을 매칭합니다.
* **결합 분포($P(\text{Morphology}, \text{Context})$) 모델링:** 조건부 확률이 아닌 결합 확률을 추적하여 수학적으로만 유효하고 물리적으로는 불가능한 결함 생성을 원천 차단합니다.
* **시맨틱 프롬프트 결합:** 클러스터 특성을 확산 모델(Diffusion Model)이 이해할 수 있는 자연어 수정자(Modifier)로 정렬하여 생성 제어력을 극대화합니다.

---

## 2. 하이브리드 ROI 정책 생성기 (Hybrid Policy Generator)

수동 규칙의 해석 가능성과 데이터 구동형 최적화의 유연성을 동시에 확보하기 위해 **2단계 하이브리드 아키텍처**를 채택합니다.

```
[Target Dataset] ──> [Complexity Analysis (MCI, CCI)]
                             │
                             ▼
               ┌───────────────────────────┐
               │ 1단계: 위상학적 라우팅      │ (큰 틀의 알고리즘 카테고리 결정)
               │ - Multimodal ──> GMM      │ -> 구조적 해석 가능성 확보
               │ - Unimodal   ──> Percentile│
               └─────────────┬─────────────┘
                             │
                             ▼
               ┌───────────────────────────┐
               │ 2단계: 데이터 구동형 최적화  │ (세부 파라미터 미세 조정)
               │ - Bayesian / Optuna       │ -> 대리 목적 함수 극대화
               └───────────────────────────┘

```

### 2.1. 메타 피처 정규화

서로 다른 통계 스케일을 통일하기 위해 기준 데이터셋 풀(Reference Pool) 기반의 Min-Max 정규화를 적용합니다.

$$\tilde{x}_i = \frac{x_i - \min(X_{i,\text{ref}})}{\max(X_{i,\text{ref}}) - \min(X_{i,\text{ref}})}$$

### 2.2. 자원 절약형 대리 목적 함수 (Proxy Objective Function)

파라미터 변경 시마다 디퓨전 모델을 무겁게 학습시키는 리스크를 방지하기 위해, 추출된 통계 피처 벡터 공간 내에서 작동하는 대리 지표를 설계합니다.

$$\max_{\Theta} f(\Theta) = \text{Silhouette}(\mathcal{M}_\Theta) + \text{Silhouette}(\mathcal{C}_\Theta) + \lambda \cdot I(\mathcal{M}_\Theta; \mathcal{C}_\Theta)$$

* $\text{Silhouette}$: 형태학($\mathcal{M}$) 및 맥락($\mathcal{C}$) 클러스터의 내부 응집도와 외부 분리도 극대화
* $I(\mathcal{M}_\Theta; \mathcal{C}_\Theta)$: 형태학과 맥락 간의 상호 정보량(Mutual Information)을 측정하여 데이터 고유의 인과관계를 포착

$$I(\mathcal{M}; \mathcal{C}) = \sum_{i} \sum_{j} P(\mathcal{M}_i, \mathcal{C}_j) \log \frac{P(\mathcal{M}_i, \mathcal{C}_j)}{P(\mathcal{M}_i)P(\mathcal{C}_j)}$$

---

## 3. 리스크 최소화형 데이터셋 전략

모든 클래스를 감당하는 컴퓨팅 자원 소모 리스크를 방지하기 위해, 특성이 뚜렷한 **3종의 타깃 데이터셋에서 각각 2~3개의 핵심 클래스만 핀포인트 샘플링**하여 복잡도 대조군을 형성합니다.

| 구분 | $CCI$ (맥락 복잡도) 낮음 | $CCI$ (맥락 복잡도) 높음 |
| --- | --- | --- |
| **$MCI$ (형태 복잡도) 낮음** | **MVTec AD**<br>

<br>*(Percentile / Otsu 위주 가벼운 정책 작동)* | **VisA (PCB 등)**<br>

<br>*(맥락 분리 및 상호 정보량 최적화 검증)* |
| **$MCI$ (형태 복잡도) 높음** | **Severstal Steel**<br>

<br>*(GMM / 계층적 형태 클러스터링 활성화)* | **High-Complexity Extreme 환경**<br>

<br>*(하이브리드 최적화 전체 작동)* |

---

## 4. 엔드투엔드(End-to-End) 5단계 연구 프로세스

* **Step 1: 데이터셋 복잡도 분석 및 하이브리드 정책 라우팅**
* 결함 마스크/배경에서 피처 추출 ➡️ $MCI, CCI$ 연산 ➡️ 결정 트리 분기 ➡️ 베이지안 최적화($\Theta^*$) 수행 (CPU 연산 위주)


* **Step 2: ROI 추출 및 형태·맥락 피처 분석**
* 동적 클러스터링($M_1 \sim M_5, C_1 \sim C_5$) ➡️ 사전 확률 학습 ➡️ Deficit-Aware ROI 점수 기반 희소 조합 샘플링 우선권 부여


* **Step 3: 패치 정보 기반 결함 이미지 및 시맨틱 프롬프트 생성**
* ROI 메타데이터 기반 템플릿 자연어 변환 ➡️ 프롬프트 조건부 디퓨전 인페인팅(Inpainting) 가동


* **Step 4: 합성 및 증강 데이터셋 가공 (★핵심 디테일 수정 구간)**
* 블렌딩 노이즈 필터링 및 숏컷 방지 기법 적용 ➡️ YOLO 표준 포맷(`.txt`) 정규화 라벨링 자동 생성


* **Step 5: YOLO 기반 다운스트림 성능 평가**
* 가장 가벼운 **YOLO Nano(YOLOv8n)** 모델 활용 ➡️ **TSTR (Train on Synthetic, Test on Real)** 프로토콜 기반 실제 데이터 분류/탐지 정확도(mAP50, F1-Score) 평가



---

## 5. [상세 설계] Step 4: 리스크 대응형 합성 및 데이터셋 가공

기존 CASDA 연구 등에서 발생했던 **'그라디언트 세척(에지 뭉개짐)'** 및 **'숏컷 학습(경계면 노이즈만 학습)'** 문제를 근본적으로 해결하기 위한 특화 엔지니어링 단계입니다.

### 5.1. 잠재 공간 마스크 블렌딩 (Latent-Space Blending)

픽셀을 사후에 깎아내거나 문지르는 포아송 방식 대신, 디퓨전 모델의 디노이징 타임스텝 $t$의 잠재 공간(Latent Space) 내에서 노이즈 레벨로 정렬합니다.

$$z_{\text{final}}^{(t)} = M \odot z_{\text{defect}}^{(t)} + (1 - M) \odot z_{\text{bg}}^{(t)}$$

> **💡 핵심 효과:** 주변 배경과 광원 레이어는 완벽히 조화를 이루면서도, 결함 고유의 날카로운 고주파 에지(Crisp Edge) 성분이 뭉개지지 않고 그대로 보존됩니다.

### 5.2. 구조 보존형 질감 조화 (Structure-Preserving Harmonization)

카피-페이스트 연산 시에는 에지 보존 필터(양방향 필터 등)를 통해 고주파 성분(High-pass)을 분리 격리한 후, 저주파 색조만 매칭하고 다시 결합하여 에지 선명도를 사수합니다.

### 5.3. 숏컷 방지를 위한 무작위 마스크 확장 (Anti-Shortcut Augmentation)

YOLO가 합성 이음새를 불량의 특징으로 오버피팅하는 것을 막기 위해, 마스크 외곽을 무작위로 $3\sim7$ 픽셀만큼 다이레이션(Dilation)하여 정답 좌표를 생성합니다. YOLO가 경계선 노이즈가 아닌 내부의 진짜 오브젝트 특징에 집중하도록 강제합니다.

### 5.4. 자동 어노테이션 정규화 변환

수동 공수를 배제하기 위해 ROI 마스크 좌표를 YOLO 정규화 수식에 따라 즉시 텍스트 포맷으로 패키징합니다.

$$\text{Center\_X} = \frac{x_{\min} + x_{\max}}{2 \cdot \text{Image\_Width}}, \quad \text{Center\_Y} = \frac{y_{\min} + y_{\max}}{2 \cdot \text{Image\_Height}}$$

$$\text{Width} = \frac{x_{\max} - x_{\min}}{\text{Image\_Width}}, \quad \text{Height} = \frac{y_{\max} - y_{\min}}{\text{Image\_Height}}$$

---

## 6. 최종 성능 평가 장표 설계

오직 AROMA 파이프라인으로만 빌드된 합성 데이터로 YOLO Nano를 학습시키고, 100% 실제 데이터로 테스트하여 방법론의 완전성을 계량화합니다.

| 훈련 데이터셋 (YOLOv8n 학습) | 테스트 데이터셋 (실제 데이터) | Severstal (mAP50) | MVTec AD (mAP50) | VisA (mAP50) |
| --- | --- | --- | --- | --- |
| **Real Only (10% 소량 소스)** | 실제 Target 데이터 | Baseline | Baseline | Baseline |
| **CASDA Augmentation** | 실제 Target 데이터 | $x.xx$ | $x.xx$ | $x.xx$ |
| **AROMA (Heuristic)** | 실제 Target 데이터 | $\Delta \text{up}$ | $\Delta \text{up}$ | $\Delta \text{up}$ |
| **AROMA (Hybrid Optimization)** | 실제 Target 데이터 | **최고 성능** | **최고 성능** | **최고 성능** |