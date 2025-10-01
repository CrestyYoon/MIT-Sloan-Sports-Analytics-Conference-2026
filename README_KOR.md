# SSAC 2025 — Soccer Momentum via State Space Models (공개 자료 묶음)

이 저장소(제출용 번들)는 **코드 비공개** 원칙을 유지하면서도,
심사위원/검증자가 결과를 확인할 수 있도록 **데이터와 문서**를 제공합니다.

## 코드 공개 범위(부분 공개)

재현성과 서비스 IP 보호를 균형 있게 고려하여 **특징 엔지니어링 노트북 3개**와
모든 **파생 데이터 산출물(CSV)** 만 공개합니다. 전체 학습/평가 파이프라인 코드는 비공개로 유지합니다.

**공개 노트북**
- `1_Event_Refine.ipynb` — 이벤트 정제/정규화, 포제션·시퀀스 태깅, `data/analysis_final/<match_id>.pkl` 저장.
- `3_Sequence_Line.ipynb` — 라인브레이크/Zone‑14·박스 진입 등 시퀀스 기하 파생, 다운스트림 특징 테이블 생성.
- `5_Attack_metric.ipynb` — 모델에 투입되는 공격 측 특징(비율·카운트) 생성.

**비공개 노트북(IP 보호)**
- `7_Analysis.ipynb` — 상태공간 학습/평가(팀 분리 듀얼‑스트림 업데이트), OOS 점수, 풀링.
- 수비 측 특징 설계 템플릿.

**비공개 코드 없이도 검증 가능**
논문 수치는 공개 CSV만으로 재현/검증할 수 있습니다:
`final_evaluation_summary.csv`, `oos_predictions_residuals.csv`,
`ols_screening_all.csv`, `ssm_pooled_all.csv`, `ssm_snapshot_flat.csv`.
자세한 절차는 *METHODS_KOR.md → 재현 가이드*를 참고하세요.


## 포함 파일
- `final_evaluation_summary.csv` : 모델 전반 비교(OLS vs SSM) 요약
- `ols_screening_all.csv` : OLS 스크리닝 요약
- `ssm_pooled_all.csv` : SSM 풀링 계수/유의성/성능 요약
- `split_info.json` : 학습/검증 규칙·키·카운트
- `split_table.csv` : (생성용) match_id,split 표 — 헤더만 포함(스크립트로 생성)
- `DATA_DICTIONARY.md` : 지표 정의서(열 이름/정의/단위/윈도우)
- `METHODS.md` : 방법 재현 문서(모형·A‑1 처리·풀링·평가·주의)
- `LICENSE` : 데이터/문서 라이선스 고지
- `make_split_table.py` : 로컬 분석 산출물에서 `split_table.csv` 생성 스크립트

## 빠른 검증 가이드(코드 없이)
1. `split_info.json`의 규칙(예: `match_id % 5 == 0` → test)으로 홀드아웃을 재구성합니다.  
   (정확 match_id 목록이 필요하면 아래 스크립트로 생성)
2. `ssm_pooled_all.csv`의 계수(`coef`)와 지표 테이블을 사용해 **μ 보정 선형예측** `ŷ = μ_train + Xβ` 를 계산합니다.
3. `final_evaluation_summary.csv`의 OOS RMSE/MAE와 비교해 일치 여부를 확인합니다.
4. 필요 시, DM 테스트는 보조적으로만 참고합니다(격자/대용량 특성상 과대 가능).

## split_table.csv 생성(선택)
- 로컬에 `open-data-master/data/analysis_final/*.pkl`(매치별 프레임)이 존재한다면, 아래 스크립트를 실행하세요.
```bash
python make_split_table.py
```
- 기본 탐색 경로는 다음 순서로 시도합니다.  
  1) `C:\analyst\StatsBomb Open Data\open-data-master\data\analysis_final`  
  2) `./open-data-master/data/analysis_final`
- 성공 시, 현재 디렉토리에 `split_table.csv`가 생성됩니다.

## 문의
- 본 공개본은 **데이터/문서 중심**입니다. 코드 공개는 선택사항이며, 본 제출에서는 비공개로 유지합니다.


### LLM 사용 고지
본 프로젝트는 문서 초안 정리, 코드 리팩터링 아이디어, 체크리스트 생성 등에서 LLM의 도움을 받았습니다. 
모든 모델링 의사결정, 데이터 정제와 검증은 저자(단독)가 수행했으며, LLM은 개인 데이터에 접근하거나 분석을 실행하지 않았습니다. 
모든 스크립트와 결과물은 로컬 환경에서 재현·검증되었습니다.


### 초록용 그림 · 표
- `summary_oos_table.png` — 핵심 수치 요약 1페이지 표
- `oos_rmse_comparison.png` — OOS RMSE 비교(축 생략 표시 포함)


### 패키지 구성(주요 파일)
- `final_evaluation_summary.csv`, `oos_predictions_residuals.csv`
- `ols_screening_all.csv`, `ssm_pooled_all.csv`, `ssm_snapshot_flat.csv`
- `attack_features.csv`, `defense_features.csv`
- `scale_summary.csv`, `zscore_summary.csv`, `split_info.*`
- `METHODS_KOR.md`, `README_KOR.md` (영문판 포함)
- `summary_oos_table.png`, `oos_rmse_comparison.png`
