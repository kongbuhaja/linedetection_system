# ad-4-linedetection-project1
[4기] 자율주행 데브코스 차선인식 온라인 프로젝트 repository

# 프로젝트 기획  
## 프로젝트 목표  
차량에 장착된 카메라를 활용하여 차량이 주행하기위해 필요한 차선인식 기능을 구현해보자.  

## 프로젝트에 사용한 알고리즘

이번 프로젝트는 크게 이미지 전처리와, 차선인식으로 나눌 수 있다.

### **이미지 전처리**

이미지 전처리는 영상에서 보다 더 좋은 데이터를 추출하기위한 환경을 구축하기위해 반드시 필요한 과정이다.

이미지 전처리 순서는 roi영역 추출, noise제거, 영상 명암비 조절, 영상 이진화, lidar영역 제거를 따른다.

**roi영역 추출**: 우리의 목표는 height=400일때 차선의 위치를 찾는 것으로 (0,380), (640,400)으로 설정했다.

**noise제거**: GaussianBlur를 활용하는데 5x5필터를 1번 수행하는 것보다 3x3필터를 2번 수행하는 것은 같은 결과를 가져오지만 연산량을 줄일수 있다.(m x n x 25 > m x n x 9 x 2)

**영상 명암비**: 영상의 평균 밝기 값을 이용해 명암비를 높여 차선인식에 유리한 영상으로 만들었다.
(dst=dst+(dst-mean)*1.0)

**영상 이진화**: 명암비 조절을 거친 영상을 threshold값을 활용해 영상의 이진화를 진행한다.
threshold함수는 특정 임계값보다 작으면 0, 크면 최대값으로 영상을 이진화 한다.

**lidar영역 제거**: 영상 속 lidar의 위치는 바뀌지 않으며 차후 차선 인식을 하는데 있어 장애물이 될 수 있다. 이를 masking작업을 활용해 lidar영역을 배경으로 만들어 준다.
화면의 흔들림으로 인해 lidar의 주변으로 noise가 생기는 현상이 있는데, 이를 한번 더 GaussianBlur함수, threshold함수를 통해 제거하고 다시 이진화를 진행한다.

### **차선인식**

흔히 HoughTransform 알고리즘이 차선인식에 좋은 결과를 가져온다고 알려져 있다.
하지만 이번 프로젝트는 강의 이외의 또 다른 방식으로 차선인식기능을 구현해보고 싶은 생각에 다른 알고리즘을 고려해봤다. 

이에 영상속 연결된 객체를 찾아주는 connectedComponentsWithStats함수를 사용해보기로 했다.

**connectedComponentWithStats**: 이미지내의 연결객체를 판단 해주는 함수이다. 반환값으로 객체의 수를, 매개변수로 객체 bbox의 area, left, top, width, height 정보를 얻을 수 있다.

**차선 판단**

차선을 판단하는 순서는 다음과 같다.
1. 일정 범위의 객체의 크기
2. 화면 기준 객체위치의 좌우 판단
3. 인식된 차선이 여러개일 경우 화면 중앙에 가까운 객체 우선순위 지정

**예상 가능한 장애**

차선을 인식하는데 발생할 수 있는 장애는 다음과 같다.
1. 차선이 한쪽만 인식될 경우
2. 차선이 아예 인식 안될 경우
3. 차선이 한쪽만 제대로 인식 될 경우
4. 차선이 양쪽다 제대로 인식 안될 경우
3, 4번의 경우 차량을 멈추고 영상에서 발견되는 경우가 적으며 차선판단 3번으로 어느정도 해결되기에 고려하지 않기로 했다.
1,2번의 경우 차선이 정상적으로 인식 되었을 때의 정보를 저장하여 상황이 발생했을때 이전 차선위치를 기반으로 차선의 위치를 추정하였다.

1번 경우 차선추정
이전 차선정보를 통해 양측 차선간 간격을 측정하고 그 간격만큼을 이동한 위치에 가상의 차선을 만든다.

2번 경우 차선추정
이전 차선정보를 그대로 현재 차선의 위치로 가정한다.
→ 이전 차선들의 변화량을 통해 다음 차선위치를 예측해야 할 것 같지만 마땅히 알고리즘이 떠오르지 않았다.

# 프로젝트 결과

## 최종 결과

### **Good detection**

**두 차선 인식**

![good_plane.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/20cc311b-b7b7-4b2e-bbf9-9a4515d985bb/good_plane.png)

![good_plane2.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/705950ca-facc-40e7-9a29-8389bd15e8ba/good_plane2.png)

**한 차선 예측**

![good_palne_one_missed.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef660e6a-1117-4ff5-a2c8-865a87306fc1/good_palne_one_missed.png)

![good_curve_one_missed.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1037b3a-aeae-4dbc-b7c5-b51b2eb93931/good_curve_one_missed.png)

**두 차선 예측**

![good_curve_two_missed.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4f75c67b-c04e-478b-aa22-bb19640ec560/good_curve_two_missed.png)

### **Bad detection**

**한 차선 예측**

없는 차선이 아닌 보이지 않는 차선임으로 화면밖에 표시 되야 좋은 예측

![bad_curve_one_missed.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/051829b5-ae67-4ad3-8dce-aa1121eb68c2/bad_curve_one_missed.png)

![bad_curve_one_missed2.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c77555bf-fa33-4c4d-9e63-97df1d1a2b6e/bad_curve_one_missed2.png)

없는 차선이 아닌 보이지 않는 차선임으로 화면밖에 표시 되야 좋은 예측

**완전 틀린 예측**
왼쪽차선은 핑크색 오른쪽차선은 민트색으로 인식되야하나 왼쪽차선이 오른쪽 차선으로 인식 된 상황

![wrong.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58742f0e-b034-4159-8bda-b8e6726dc739/wrong.png)

### **결과**

target: 30 frame마다 예측 위치를 csv파일로 저장

음수의 좌표는 0으로 변환, 640이 넘는 좌표는 640으로 변환 후 비교
정답 csv 파일과 대조했을때 91.66%의 정확도를 가진다.

## 협업 과정

- 슬랙 -  진행 상황 공유
- zoom - 화면공유 (코드 리뷰, 코드 컨벤션)
- github - 코드 관리
- 회의 - 서로 항상 존중해주면서 밝고 재밌는 분위기에서 프로젝트를 진행하였습니다.
- 코드 리뷰 - 왜 이렇게 코드를 작성하였는지 질문하고 의견을 제시하였습니다.
