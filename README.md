![image](https://user-images.githubusercontent.com/64716178/151357077-cda9e50f-d966-437f-aa04-ac3188c396d7.png)
***
## Contents
1. [프로젝트 소개](#1-프로젝트-소개)
2. [프로젝트 결과](#2-프로젝트-결과)
3. [진행 순서](#3-진행-순서)
4. [음원 특성 추출](#4-음원-특성-추출)
5. [유사도 측정](#5-유사도-측정)
6. [클러스터링](#6-클러스터링)
7. [한계점](#7-한계점)
8. [참고 문헌](#8-참고-문헌)
9. [프로젝트팀 멤버](#9-프로젝트팀-멤버)

## 1. 프로젝트 소개
- 주제
    - 음원 간의 유사도를 측정해 유사 음원을 찾는 모델 개발
- 배경
    - 저작권에 대한 관심이 늘어나면서 음원 창작자들의 권리를 보호하고 피해를 최소화할 수 있는 방안에 대한 중요성도 커지고 있다.
- 의의
    - 음원 플랫폼은 표절 여부를 사전에 판단해 법적인 리스크를 최소화하고 퀄리티가 확보된 음원 데이터를 고객에게 제공할 수 있다.
    - 음원 창작자는 자신이 만든 음원을 외부에 공개하기 전 비슷한 음원이 존재하는지 확인 절차를 거칠 수 있다. 즉, 자신이 의도하지 않은 표절 의심 사례가 발생되지 않도록 예방할 수 있다.
## 2. 프로젝트 결과
- 특정 음원 데이터 입력 시, 전체 데이터 내에서 유사도 상위 음원을 세부정보와 함께 시각화해주는 모델 구현
- 전체적인 유사도뿐만 아니라 특정 구간 유사도 정보도 함께 제시해 각 유사 음원에 대한 세부적인 파악 가능
- **에스파 - Next Level 입력 결과** 
<p align="center"><img src="https://user-images.githubusercontent.com/64716178/151362419-82bc0062-07dd-4923-98cf-08957fc43bd1.png" width="600"></p>

## 3. 진행 순서
1. 음원 데이터를 같은 길이로 가공
2. 각 음원 데이터의 특성 추출
3. 추출한 특성 벡터에 대해 유사도 측정
4. 유사도를 잘 표현할 수 있는 Score 고안
5. 클러스터링을 통해 각 음원간 거리 확인
6. 모델 성능 테스트

## 4. 음원 특성 추출
- 음원 데이터
    - 사이디라이트에서 제공받은 아마추어 작곡가 제작 음원 2322곡
        - 해당 음원은 아마추어 작곡가들이 작곡했으며, 2분 이내의 길이다.
    - 음원 사이트에서 다운받은 14곡
        - 해당 음원은 표절로 판정됐거나 표절 의혹이 제기된 곡, 샘플링곡, 리메이크곡 7곡과 이들의 원곡 7곡으로 이뤄져 있다. 목록은 다음과 같다.

|원곡|비교곡|구분|
|------|------|------|
|더더 - It’s you|MC몽 - 너에게 쓰는 편지|표절|
|와이낫 - 파랑새|씨엔블루 - 외톨이야|표절 의혹|
|요시마타 료 - Resolver|FTISLAND - 사랑앓이|표절 의혹|
|TLC - No Scrubs|Ed Sheeran - Shape of you|샘플링|
|Harold Faltermeyer - Axel F|싸이 - 챔피언|샘플링|
|A$ton Wyld - Next Level|에스파 - Next Level|리메이크|
|S.E.S. - Dreams Come True|에스파 - Dreams Come True|리메이크|

- 음원 데이터는 매 20초마다 30초 분량을 잘라내는 방식으로 가공했다.
- 특성 추출에는 librosa 라이브러리를 이용했다.
### 4-1. 오디오 기반 특성 추출
한국저작권위원회 저작권자동상담시스템에 따르면 음악 표절 여부를 따지는 일반적인 기준은 가락(멜로디), 리듬, 화음의 3가지 요소의 실질적 유사성 여부이다.
- Tempo(1차원)
    - 각 음원의 리듬을 비교하기 위해 선정했다.
- Chroma(72차원)
    - 각 음원의 멜로디와 화음을 비교하기 위해 선정했다.
    - librosa는 chroma feature를 추출하는 3가지 방법(STFT(Short-Time Fourier Transform), Constant-Q Transform, Chroma-Energy Normalized)을 제공한다.
        - STFT
![image](https://user-images.githubusercontent.com/64716178/151360460-2ea55079-89d3-4899-945c-50aeed80f775.png)
        - Constant-Q Transform
![image](https://user-images.githubusercontent.com/64716178/151360567-c80acab3-370a-4d0b-bd06-acd2454d47cd.png)
        - Chroma-Energy Normalized
![image](https://user-images.githubusercontent.com/64716178/151360617-ac65158a-4116-436e-bbff-471002e36641.png)
    - 세 방법으로 각각 12차원(12음계) 벡터를 추출한 뒤 각각의 평균과 분산을 계산해 총 72차원의 벡터를 생성했다.
- MFCC(26차원)
    - 각 음원별로 소리의 고유한 특징을 비교하기 위해 선정했다.
    - 1~13차 MFCC의 평균과 분산을 각각 구해 총 26차원의 벡터를 추출했다.
    - 13차까지의 값을 사용한 것은 선행 연구 중 ‘음악 특징점간의 유사도 측정을 이용한 동일음원 인식 방법’(성보경·정명범·고일주, 2008)을 토대로 결정했다.
### 4-2. 이미지 기반 특성 추출
librosa 라이브러리를 이용해 mel-spectrogram을 생성하고, 이를 컨볼루션 신경망에 통과시켜 특징 벡터를 추출한다.

- VGG19
    - block3_conv4/ block5_conv4 두 레이어에서 각각 feature map을 추출한 뒤 GAP(Global Average Pooling)을 통과시켜 특징 벡터로 변환했다.<br>
    - block3까지 통과한 벡터는 256차원, block5까지 통과한 벡터는 512차원이다.
<p align="center"><img src="https://user-images.githubusercontent.com/76440511/151654003-9c49ce83-e706-4857-92b1-84c35e68c147.png" width="700"></p>

- AutoEncoder
    - 딥러닝 비지도 학습을 이용해 각 Mel-Specrtogram에서 잠재벡터(Latent Vector)를 추출했다.
    - 30초 분량 기준 약 6천개의 데이터로 학습 진행 후 모델 예측 결과를 확인했다.
    ![image](https://user-images.githubusercontent.com/76440511/151498920-fa0e9a7d-f44d-4c74-b887-bd0e21fed9f6.png)
    - 학습된 Model의 Encoder부분만 분리한 뒤, Encoder가 출력한 128차원의 잠재벡터를 각 음원의 특성으로 활용했다.

## 5. 유사도 측정
### 5-1. 사용한 유사도 측정 방법
- 유클리디안 거리
- 코사인 유사도
- 피어슨 유사도
### 5-2. Score 계산
Score를 고안할 때는 이미 발매된 표절, 표절의심, 샘플링, 리메이크곡과 그 원곡(총 7쌍)을 이용했다. 7쌍의 곡은 쌍끼리 유사성이 입증됐다고 봤으며, 따라서 서로에 대한 유사도가 얼마나 높게 나오는지 확인함으로써 Score의 신뢰도를 판단했다.
- 1차 Score
    - Score: 3가지 지표를 기반으로 각각 점수를 계산한 후 산술평균
        - 유클리디안 거리 점수 : 샘플음원 대비 유사음원과 비교 시, (-) 변화량 크기
        - 코사인 유사도 점수 : 샘플음원 대비 유사음원과 비교 시, (+) 변화량 크기
        - 피어슨 유사도 점수 : 샘플음원 대비 유사음원과 비교 시, (+) 변화량 크기
    - 일부 음원은 유사 음원과의 Score가 다른 음원들보다 낮게 나오는 경우도 있었다.
- 2차 Score
    - Score: 3가지 지표를 기반으로 아래와 같이 200점 만점으로 산출
![image](https://user-images.githubusercontent.com/64716178/151359330-9007e8df-676c-42ee-9c3e-41077dd05e66.png)
        - 유클리디안 거리 점수: 100 - 유클리디안 거리 (최대 100)
        - 코사인 유사도 점수: 코사인 유사도 + 1.0 (최대 2)
        - 피어슨 유사도 점수: 피어슨 유사도 + 1.0 (최대 2)
    - Score의 만점이 200점이므로 100점 이상을 ‘유사하다’고 봤다.
## 6. 클러스터링
DBSCAN과 K-Means로 각각 음원을 클러스터링했다.

|DBSCAN|K-Means|
|---|---|
|<img src="https://user-images.githubusercontent.com/76440511/151661516-6763ed2c-5bf0-4e18-81e1-7204ee592ad4.png" height="332">|<img src="https://user-images.githubusercontent.com/64716178/151360173-264dc197-2b5e-4ae6-846a-86f0a8e90c46.png" height="350">|
- DBSCAN은 클러스터별 매우 불균형한 분표를 보였다.
- 음원 데이터가 서로 가깝게 분포하고 있어 밀도 기반인 DBSCAN으로는 유의미한 클러스터링 결과를 확인할 수 없다고 판단했다.
- PCA 차원축소 및 음원 데이터 분포 시각화 현황<br>
![image](https://user-images.githubusercontent.com/64716178/151360015-88704396-e9d8-4e2b-90aa-c6c48c2cc025.png)

## 7. 한계점
1. 전처리한 음원 길이
    - 본 프로젝트에서는 음원을 30초 길이로 잘라 진행했다.
    - 추후 더 짧은 길이로 가공한다면 모델 성능 향상이 기대된다.
2. 표절 기준의 모호성
    - 법적으로 표절의 기준이 정확하게 정해져 있지 않다.
    - 본 프로젝트에서 구현한 모델을 사용하더라도 최종 판단은 사람이 듣고 직접 해야 한다.
3. 특정 구간 멜로디 유사도 측정의 한계
    - 본 프로젝트에서는 Chroma 기반 12음계 벡터의 평균, 분산을 산출해 적용함으로써 음의 구성적인 유사도는 판단하였으나, 표절 판단의 주요 기준 중 하나인 특정 구간 멜로디 흐름 자체를 비교하기에는 한계가 있었다.
    - 추후 보컬 추출한 음원을 Chroma 시각화 후, 음계의 흐름이 부각된 이미지의 특성을 추출해 비교한다면 추가 성능 향상이 기대된다.
4. 테스트 데이터 개수
    - 본 프로젝트에서는 표절, 샘플링 등 각 유형별 대표적인 유사 음원(총 14곡)을 선정해 테스트를 진행했다.
    - 추후 유형별로 더 많은 테스트 데이터를 구성해 모델 테스트를 진행한다면 보다 객관적인 성능 향상이 기대된다.
5. 음원간 유사도 비교 시 소요시간 단축 방안
    - 본 프로젝트에서는 동일 음원에 대해 매번 새롭게 유사도 계산하는 것을 방지하기 위해 기존 결과 데이터를 저장해놓고 활용했다.(매번 새롭게 유사도 계산 시, 1곡당 6천여개 음원과 비교하는 데에만 약 7분 소요)
    - 훨씬 더 많은 음원 데이터와 다양한 조건 비교를 고려 시, 클러스터링 또는 기존 DB음원 유사도순 정렬 등을 통한 1차 필터링 절차가 필요하다.

## 8. 참고 문헌
- [Very deep convolutional networks for large-scale image recognition, Karen Simonyan, Andrew Zisserman, arXiv preprint arXiv:1409.1556, 2014](https://arxiv.org/abs/1409.1556)
- [DNN을 이용한 오디오 이벤트 검출 성능 비교, 정석환·정용주, 한국전자통신학회 논문지 v.13 no.3, 571-578, 2018](https://scienceon.kisti.re.kr/srch/selectPORSrchArticle.do?cn=JAKO201819355173292&dbt=NART)
- [음악 특징점간의 유사도 측정을 이용한 동일음원 인식 방법, 성보경·정명범·고일주, 한국컴퓨터정보학회 논문지 v.13 no.3, 99-106, 2008](https://www.kci.go.kr/kciportal/ci/sereArticleSearch/ciSereArtiView.kci?sereArticleSearchBean.artiId=ART001253733)
- [[기고] 홈쇼핑모아의 유사 이미지 검색 시스템, 컴퓨터월드, 2020.10.31,](https://www.comworld.co.kr/news/articleView.html?idxno=50015)
## 9. 프로젝트팀 멤버
|Member|Connect|
|:------:|:------:|
|권라영|[![gmail_logo](https://user-images.githubusercontent.com/64716178/150918268-0bf92ae7-8237-4222-9a90-2650ff3050b6.png)](mailto:kraeyong@gmail.com) [![github_logo](https://user-images.githubusercontent.com/64716178/150918265-60193f86-b2fa-4f53-8137-2668f6d20a23.png)](https://github.com/raunee)|
|유현준|[![gmail_logo](https://user-images.githubusercontent.com/64716178/150918268-0bf92ae7-8237-4222-9a90-2650ff3050b6.png)](mailto:hjunyoo17@gmail.com) [![github_logo](https://user-images.githubusercontent.com/64716178/150918265-60193f86-b2fa-4f53-8137-2668f6d20a23.png)](https://github.com/hyunjuyo)|
