# 지능형자동차 인식시스템 개발동향 (특집2) : 자동차용 스테레오 비전 개발동향 

## 1. 서론

최근 트랜드 : Lidar + 모노 카메라 --> 스테레오 비젼 
- eg: 독일Daimler회사 103km 완전 자율 주행 수행 

- 이유 
    - 센서의높은가격
    - 구동장치로인한약한내구성
    - 차량외관을 심하게 변형시켜야 하는 센서 부착 방식 
    
## 2. Stereo Matching

- 거리 정보를 획득 절차 
    1. 두 카메라의 내/외부 변수를 추정하는 카메라 **칼리브레이션** 과정
    2. 두 카메라에서 획득된 영상 상에서 서로 대응되는 위치를 찾는 **스테레오 매칭** 과정


### 2.1 카메라 칼리브레이션

- Offline으로비교적간단하게수행될수있기때문에오랜기간구동하면서 발생하는 카메라의 작은 변화를 Online으로 보정하는 것이 주요 연구 대상이다. 

### 2.2 스테레오 매칭

- 다양한 방법들이 존재하며 많은연산량을필요로한다. 

- 계산된 조밀한 스테레오 매칭 결과를 **Dense Disparity Map**이라고부른다.

- 가장 선호되는 알고리즘 : SGM(Semi-Global Matching)
    - Benz S-class에 적용 

> SGM은 픽셀단위의 매칭Cost와 다수의 1D Constraint로근사화된 2D Smoothness를 최소화함으로써 스테레오 매칭을 수행하는 방법


![](https://i.imgur.com/3eCTQuu.png)

## 3. General Obstacle Detection (GOD)

- 정의 : 일반적인장애물을검출하는방식

- 분류 
    - 모노 비젼 : 물체의 **외형을 학습**하는 방법으로만 물체를 검출할 수 있다. 
    - 스테레오 비전 : 조밀한 거리 정보를 제공하기 때문에 물체의 유형과 관계없이 **거리 정보를 기반**으로 장애물을인식할 수 있다

- GOD는 중간수준표현(Medium-level Representation)이다. 
    - 저수준 정보(Color Image, Dense Disparity Map)를 바로 사용하면 정보의 양이 너무 많다. 

### 3.1 GOD 혹은 중간수준표현 단계 수행

![](https://i.imgur.com/6XymEF3.png)

먼저 GOD 혹은 중간수준표현 단계를 효과적으로 수행하기 위해서는 전방 도로면의 형태를 추정하는 과정이 필요하다

도로면은 일반적으로 3차원 곡면이다. 따라서 평면, 2차 곡면, B-spline 곡면 등의 3차원곡면 모델을 사용하여 추정할 수 있다. 

하지만 일반적인 포장도로 상황에서는 좌/우 방향의 기울기는 무시가능하기때문에대부분의경우2차원곡선모델을사용하여 V-disparity 혹은 YZ-plane 도메인에서추정한다. 

이러한 근사화는 도로면 추정 연산량을 줄여주며, 스테레오 매칭 오류와 장애물에서 생성된 3차원 점들에대한강인성을확보해줄수있다는장점을갖는다.

도로면 모델링에 사용되는 2차원 곡선 모델로는 1, 2차 다항식(Polynomial), 구분적 선형(PiecewiseLinear) 함수가있으며, 이를일반화한형태인B-spline 함수도있다. 

이러한모델들중Cubic B-spline함수가 높은 자유도로 인해 다양한 형태의 도로면을 정확하게 표현할 수 있다고 알려져있다. 

하지만 이는 높은 자유도로 인해 스테레오 매칭 오류와 장애물에서 생성된 3차원점들에 강인하게 추정되기 힘들다는 한계를갖기 때문에 이에 대한 추가 연구가 필요한상황이다.

### 3.2 GOD 혹은 중간수준표현으로 사용 되는대표적인방법은세가지가있다

![](https://i.imgur.com/A4LStCI.png)

#### A. Occupancy Grid 기반 방법

이는 전방 주행 환경을 격자로 표현한후 격자를 구성하는 각 칸이 점유되었을 확률을 스테레오 비전 센서를 기반으로 계산하여 전방 상황을 표현하는 방법으로 로보틱스 분야에서 오랜 기간 사용되어온 방법이다.

#### B. Digital Elevation Map 기반방법

이는 주행 환경을 격자로 표현한 후, 격자를 구성하는 각 칸에 존재하는장애물의 높이를 계산하여 이를 기반으로전방 상황을 표현하는 방법으로 주로 루마니아의S. Nedevschi 교수에의해연구되고있다.

#### C. Stixel 기반 방법

이는장애물과 도로면의 경계와 장애물의 높이를추정하여 이를 이어주는 다수의 막대들로전방 상황을 표현하는 방법으로 Stixel은Stick과 Pixel의 합성어이다. 

이 방법은 주로Daimler의U.Franke 팀에의해연구되고있다

### 3.2 가장 선호되는 GOD 알고리즘 : Stixel 

- 장점 : 매우 많은 정보량을 갖는 Dense Disparity Map을 위치, 높이, 움직임 정보를 갖는 Stixel이라불리는 소수의 막대로 표현하는 방법으로 정보 축약 측면에서 매우 효율적

- 수행 절차 
    1. 이 방법은 전방 주행 상황에 존재하는 장애물이 도로면에수직하게서있다는 가정하에이를 고정픽셀 너비의 Stixel로 모델링한다. 
    2. 그 후 SGM을 통해 생성된 Dense Disparity Map을 사용하여 Stixel의 위치 및 높이를 추정한다. 
    3. 검출된 Stixel의 움직임은 Dense Optical Flow와 Kalman Filter를 기반으로 추정되며, 이와 같은 움직임 정보를포함하는Stixel을Dynamic Stixel이라고부른다. 
    4. 마지막으로 Dynamic Stixel들은 위치, 높이, 움직임의 유사성을 기반으로 구분되어 실세계에서 서로 같은 물체를 구성하는 Stixel들끼리 묶여지게된다.
    
    
![](https://i.imgur.com/blbcT1B.png)
```
<그림 5>는 Daimler에서 진행하고 있는 Stixel 처리방법의예를보여준다.
```

## 4. Classifier-based Object Detection (COD)


