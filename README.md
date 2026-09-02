# NPB 선수들에 대한 유형별 클러스터링

---

## 1. 분석 목적

* **배경**: NPB는 전반적으로 투고타저 특성이 강해, 단일 스탯(타율, ERA 등)만으로는 선수별 역할 및 성향 구분이 어려움.
* **목적**: 타자와 투수를 분리하여 클러스터링을 수행, 해석 가능한 선수 유형 도출.
  * **타자**: 출루형, 파워형, 컨택형 등
  * **투수**: 선발형, 탈삼진형, 범타유도형 등
* **활용**: 라인업 구성, 매치업 전략, 대체 전력 판단 등 실무적 의사결정 활용 구조 마련.

---

## 2. 데이터 및 전처리 과정

### 데이터 수집 및 정제
* **NPB 데이터**: 공식 사이트(`[성적·기록]` $\rightarrow$ `[시즌 성적]`) 내 팀별 투수/타자 성적 수집 및 `pitchers_npb`, `hitters_npb` 통합 파일 생성.
* **이닝 수 조정**: 원본 CSV의 이닝 수 소수점 표기(`0.1`, `0.2`)를 실제 아웃카운트 비율($1/3$, $2/3$)로 수정하여 파생지표 오산 방지.
* **샘플 필터링**:
  * **타자 (65타석 이상)**: 센트럴리그 투수의 타석 입장으로 인한 데이터 오염 방지 (최다 이닝 투수 무라카미 쇼키의 61타석 감안, 65타석 이상 전문 타자 218개 추출).
  * **투수 (20이닝 이상)**: 적은 이닝 샘플의 이상치 제거 목적 (전문 투수 219개 추출).
* **KBO 데이터 확장**: 샘플 정제 후 데이터 규모 확장을 위해 스탯티즈 데이터 수집 및 동일 기준(65타석, 20이닝) 적용 (광고로 인한 CSV 결측행 제거 진행).

### 파생지표 생성
선수 성향 판단을 위한 파생지표 산출.

| 구분 | 파생지표 | 산출 공식 및 설명 |
| :--- | :--- | :--- |
| **투수** | `K/9`, `BB/9`, `HR/9`, `H/9` | 9이닝 당 삼진, 볼넷, 피홈런, 피안타 수 |
| | `LOB%` | 잔루 비율: $\frac{\text{피안타} + \text{사사구} - \text{실점}}{\text{피안타} + \text{사사구} - 1.4 \times \text{피홈런}}$ |
| **타자** | `OPS` | 출루율 + 장타율 |
| | `ISO` | 순장타율 |
| | `K%`, `BB%` | 전체 타석 당 삼진 및 볼넷 비율 |
| | `BABIP` | 인플레이 타구 안타율: $\frac{\text{안타} - \text{홈런}}{\text{타수} - \text{삼진} - \text{홈런} + \text{희생플라이}}$ |

### 지표 표준화 (Standardization)
`OPS`(0.3~1.2)와 `ISO`(0.3 미만) 등 지표 간 스케일 차이로 인한 편향 방지를 위해 Standard Scaling 적용.

**[NPB 표준화 분포]**  
![NPB Pitcher Standardized](images/NPB%20pitcher%20standard.png)  
![NPB Hitter Standardized](images/NPB%20hitter%20standard.png)  

**[KBO 표준화 분포]**  
![KBO Pitcher Standardized](images/KBO%20pitcher%20standard.png)  
![KBO Hitter Standardized](images/KBO%20hitter%20standard.png)  

---

## 3. 분석 방법 및 결과

### 3-1. k-means 클러스터링
* **방법**: IQR 기반 이상치 제거 후 k-means 클러스터링 수행.

**[NPB k-means 결과]**  
![NPB k-means](images/NPB%20k-means.png)  

**[KBO k-means 결과]**  
![KBO k-means](images/KBO%20k-means.png)  

* **결과**: Elbow point가 불명확하고 Silhouette score 또한 전 구간 0.25 미만으로 형성되어 군집화 부적합 판단.

### 3-2. DBSCAN
* **방법**: 비구형 군집 포착을 위해 `eps`(0.2~2.0) 및 `min_samples`(2~20) 파라미터 그리드 탐색 진행.

**[NPB DBSCAN 결과]**  
![NPB DBSCAN Pitcher](images/NPB%20DBSCAN%20pitcher.png)  
![NPB DBSCAN Hitter](images/NPB%20DBSCAN%20hitter.png)  

**[KBO DBSCAN 결과]**  
![KBO DBSCAN Pitcher](images/KBO%20DBSCAN%20pitcher.png)  
![KBO DBSCAN Hitter](images/KBO%20DBSCAN%20hitter.png)  

* **결과**: 실루엣 스코어가 전반적으로 매우 낮게 형성되어 의미 있는 군집 생성 실패.

### 3-3. PCA (주성분 분석)
* **방법**: 데이터의 연속성 분포 가설 하에 차원 축소 기법인 PCA 적용 및 리그별 주성분 해석.

**[NPB PCA 결과]**  
![NPB PCA Pitcher](images/NPB%20PCA%20pitcher.png)  
![NPB PCA Hitter](images/NPB%20PCA%20hitter.png)  

**[KBO PCA 결과]**  
![KBO PCA Pitcher](images/KBO%20PCA%20pitcher.png)  
![KBO PCA Hitter](images/KBO%20PCA%20hitter.png)  

* **결과**: 누적 설명력은 다소 제한적이나, PC 축별 주요 변수 조합을 통해 리그 간 선수 특성 차이 도출.

---

## 4. 인사이트 및 결론

### 인사이트 1 : 선수 성적의 연속적 분포 확인 (군집화 부적합)
* 타율, 홈런 등의 스탯은 특정 구간으로 이단 분할되지 않고 연속적 스펙트럼 형태로 분포.
* **원인 분석**: "거포형", "제구형" 등의 명칭은 극단적 장점을 묘사하는 표현이며, 대다수 선수는 평균적 성적대에 위치함. 연속형 데이터 특성을 고려하지 않은 초기 전제의 오류 확인.

### 인사이트 2 : 양 리그 투수진의 평가 축 동일성
* NPB 및 KBO 모두 **PC1은 `H/9`(피안타)**, **PC2는 `BB/9`(볼넷)**가 주요 양의 상관관계를 형성.
* 두 리그 모두 투수 성향을 설명하는 핵심 지표 축이 동일함을 검증.

### 인사이트 3 : 양 리그 타자진의 구분 축 차이 (삼진 vs 볼넷)
* **공통점**: PC1은 `OPS`, `ISO`, `BB%` 등 생산성/파워 지표 중심으로 형성.
* **차이점**: PC2에서 **NPB는 `K%`(삼진 비율)**가, **KBO는 `BB%`(볼넷 비율)**가 주요 주성분으로 작용.
* **원인 추론**: NPB 투수진의 구위 상향평준화로 인해 NPB 타자들은 적극적인 타격 성향을 보이는 반면, KBO 타자들은 볼넷을 골라내는 대기 성향이 상대적으로 큼.

---

## 5. 요약 및 향후 과제

* **요약**: NPB/KBO 선수 성적 데이터를 활용해 k-means, DBSCAN 클러스터링을 시도하였으나 실패(Negative Result)를 통해 데이터의 연속성을 검증함. 선회 적용한 PCA를 통해 양 리그 투수의 공통 평가 축과 타자의 리그별 타석 임하는 성향 차이를 발굴함.
* **향후 과제**: 
  1. 다년도(Multi-year) 데이터 수집을 통한 데이터 샘플 크기 확장.
  2. Statcast 기반 타구 속도, 발사각, 투구 궤적 등 세부 트래킹 지표 도입.
  3. 연속형 데이터 특성에 적합한 신규 분석 모델 탐색 및 군집화 재도전.
