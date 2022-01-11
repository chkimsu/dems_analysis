### Code, File summary

* `/preprocessing` : Input 및 전처리단
* `/preprocessing/output` : Input 및 전처리단의 출력물 (전처리 수행 후 산출되는 학습마트 R 객체 등)
* `/preprocessing/preprocessing.R` : Input 및 전처리단 R 코드 (preprocessing.rmd 의 산출물)
* `/preprocessing/preprocessing.html` : Input 및 전처리단의 코드실행 리포트 (preprocessing.rmd 의 산출물)
* `/preprocessing/preprocessing.rmd` : Input 및 전처리 Rmarkdown 코드 원본
* `/preprocessing/preprocessing.Rout` : Input 및 전처리단 R 코드 실행에 대한 로그

* `/modeling` : 모델링단
* `/modeling/output` : 모델링단의 출력물 (선정모델 R 객체 등)
* `/modeling/modeling.R` : 모델링단 R 코드 (modeling.rmd 의 산출물)
* `/modeling/modeling.html` : 모델링단의 코드실행 리포트 (modeling.rmd 의 산출물)
* `/modeling/modeling.rmd` : 모델링 Rmarkdown 코드 원본
* `/modeling/modeling.Rout` : 모델링단 R 코드 실행에 대한 로그

* `/predict` : 예측테이블 계산 및 Out단
* `/predict/predict.R` : 예측테이블 계산 및 Out단 R 코드 (predict.rmd 의 산출물)
* `/predict/predict.html` : 예측테이블 계산 및 Out단의 코드실행 리포트 (predict.rmd 의 산출물)
* `/predict/predict.rmd` : 예측테이블 계산 및 Out의 Rmarkdown 코드 원본
* `/predict/predict.Rout` : 예측테이블 계산 및 Out단 R 코드 실행에 대한 로그

* `/sh/run.sh` : 전체 스크립트 실행 스위치 쉘
* `/sh/log` : `run.sh` 실행에 대한 표준출력 및 표준에러 로그

* `/.gitignore` : git 버전관리에 무시될 파일포맷 설정
* `/README.md`



### Create `pred_elec` table query

```sql
CREATE TABLE pred_elec (
  regDate DATETIME NOT NULL,
  predDate DATE NOT NULL,
  predTime VARCHAR(2) NOT NULL,
  zone VARCHAR(1) NOT NULL,
  predElecMV DECIMAL(10, 2),
  predElecMVCost DECIMAL(10, 2),
  predElec DECIMAL(10, 2),
  predElecCost DECIMAL(10, 2),
  predDownCost DECIMAL(10, 2),
  PRIMARY KEY (predDate, predTime, zone)
)
```

### `pred_elec` table 변수별 의미

* `regDate` : 데이터 생성날짜시각
* `predDate` : 예측시점 날짜
* `predTime` : 예측시점 시각 (고정폭 2width string으로 변경)
* `zone` : Zone 구분
* `predElecMV` : 과거 패턴기반 항온항습 전력량 예측값
* `predElecMVCost` : 과거 패턴기반 항온항습 전력량 기준 전력사용비용 예측값
* `predElec` : 설정온도 기반 항온항습 전력량 예측값
* `predElecCost` : 설정온도 기반 항온항습 전력량 기준 전력사용비용 예측값
* `predDownCost` : `predElecMVCost` - `predElecCost` 값

### `pred_elec` table 성격

현재 1시간 간격으로 Zone 별, 하루미래 00~23시 까지의 24개 예측값이 생성되므로 한시간 간격 120개의 행 데이터가 append 되는 형태임

### `pred_elec` table 연동방법

최적운전 -> 수요예측 페이지의 

1. 수요예측 선 : x축은 `predDate`, `predTime` y축은 `predElecMV` 값을 이용하여 그래프 플롯팅
2. 예측 전력량 선 : x축은 `predDate`, `predTime` y축은 `predElec` 값을 이용하여 그래프 플롯팅

### 최적운전 -> 수요예측 페이지의 그래프 컨텐츠의 갱신 주기

매일 00:00 정각  
본 분석스크립트 배치 실행 주기는 매일 23:00  
crontab 을 통해 아래처럼 스케줄링 등록  

```
hjsong@BI02:~$ crontab -l
0 23 * * * /home/hjsong/DEMS/03_dev/sh/run.sh > /home/hjsong/DEMS/03_dev/sh/log 2>&1
```
