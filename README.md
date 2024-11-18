
# 포인트 클라우드를 활용한 방 꼭짓점 보정 알고리즘

아이폰 Lidar 센서로 추출된 3차원 depth값을 point cloud 좌표로 바꿔 방의 꼭짓점을 자동으로 보정하는 알고리즘입니다.
2차원 RGB이미지에서 찾은 keypoint의 정확도를 확보하기 위해 3차원 Point cloud 기반 알고리즘을 설계하였습니다.

## 프로젝트 개요

### 기존 연구 진행 상황
- Hrnet을 transfer-learning하여 특정 카테고리에서 특정 point를 찾는 keypoint detection
- 아이폰의 Lidar 센서로 추출한 3차원 depth 값과 2차원 keypoint의 좌표 매칭  
- 두 키포인트의 3차원 좌표로 실제 길이 측정

### 문제 상황
1. 비대면 도배 견적 플랫폼의 방 치수 자동 측정 솔루션의 도입을 위해 방의 가로, 세로, 높이를 정확하게 측정해야 하는 상황
2. 아이폰의 Lidar 센서의 특성 상 256*192의 depth map 형성 -> 방의 꼭짓점이 point cloud에 반영되지 않는 경우 발생
3. 방 특성상 길이가 큼 -> 2차원에서 찾은 keypoint의 약간의 오차가 3차원 측정에서는 큰 오차로 발생

## 해결 방안
### 알고리즘 개요
- 방의 꼭짓점은 실제로 3개의 평면이 만나서 생기는 점이라는 것에서 기안
- 2차원에서 찾은 keypoint를 기반으로 3차원 point cloud 좌표에서 꼭짓점을 찾아서 보정

## 세부 개선 사항
### 1. 기존 진행 결과 확인
![Example Image](https://github.com/MustreeAI/fix-coner-algorithm/blob/main/public/room_depth_image.png)

Transfer learned Hrnet Keypoint detection model과 3차원 depth값을 활용한 방 높이 자동 측정 결과이다.
실제 높이가 2.45m인 것을 감안하면, keypoint detection으로 찾은 Keypoint와 길이는 정확해보인다.

실제 3차원 point cloud상에서 keypoint 하나를 자세히 시각화한 결과다.
    ![Example Image](https://github.com/MustreeAI/fix-coner-algorithm/blob/main/public/visualize_3d.png)

빨간색 점이 2차원 이미지에서 찾은 keypoint이고, 이 점이 정확히 모서리 부분에 있지도 않아
모서리 부분이 point cloud상에서 잘 표현되지 않고 있다.

### 2. RANSAC 알고리즘 기반 Keypoint 보정 알고리즘
앞선 시각화 자료에서 알 수 있듯이 2차원 RGB이미지에서 정확히 3차원 상의 정확한 keypoint를 찾지 못한다. 

그리하여 RANSAC 알고리즘으로 keypoint 주변 3개의 평면을 찾고, 이 3개의 평면이 만나는 점을 모서리로 생각하는 알고리즘을 고안하였고 테스를 진행하였다
#### 1. 2차원에서 찾은 keypoint 기준으로 3개의 평면 찾기
- 일단 첫번째로 2차원에서 찾은 keypoint를 기준으로 1000개의 주변 점들을 찾는다.
- 이 때 주변 점의 개수는 아이폰의 depth map의 크기가 (192,256)으로 고정인 점을 감안하여 1000개로 설정하였다.
- 그리고 RANSAC 알고리즘으로 1000개의 점에서 3개의 평면을 찾는다.

    ![Example Image](https://github.com/MustreeAI/fix-coner-algorithm/blob/main/public/3d_planes.png)
- 방의 높이를 재려고 찾은 두 keypoint를 주변으로 3개의 평면을 찾아 시각화하였다.

#### 2. 평면의 교차점(꼭짓점) 보정
- keypoint로 찾은 평면 3개의 평면 방정식을 도출하고, 방정식의 해를 찾아 교점을 구한다
- 교점의 위치로 keypoint를 수정한다.

    ![Example Image](https://github.com/MustreeAI/fix-coner-algorithm/blob/main/public/fixed_keypoint.png)
#### 3. 예외 사항
- 직교하는 3개의 평면을 잘못 찾은 경우
    - 평면에 구조물이 있는 경우
    - 평평하지 않은 평면이 있는 경우 

    ![Example Image](https://github.com/MustreeAI/fix-coner-algorithm/blob/main/public/wrong_correction_image.png)
    - 구조물에 의해 point cloud가 제대로 생성되지 않은 부분이 평면으로 인식 x

    ![Example Image](https://github.com/MustreeAI/fix-coner-algorithm/blob/main/public/wrong_correction_keypoint.png)
    - 아래에 찍힌 보라색의 점이 3 평면의 교점이자 보정된 keypoint
    - 강건성을 확보할 후처리 알고리즘 필요
## 업데이트 상황

