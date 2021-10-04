# computer vision

### 1. Classification? Detection? Segmentation?

![computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled.png](computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled.png)

컴퓨터 비전은 크게 classification, localization, object detection, segmentation으로 나뉜다.

**classification**란 영상 이미지에서 특정 대상이 나오면 확인하는 기술이다. 고양이 사진이 있다면 output으로 cat을 분류하는 것이다. 

**detection**은 **classification**과 달리 특정한 대상이 있는지 여부와 함께 위치 정보도 같이 바운딩박스로 해서 알려준다. 

우리가 해야할 과제인 **semantic segmentation**은 주어진 이미지에 어느 특정 클래스에 해당하는 사물이 어느 위치에 있는지 픽셀 단위로 분할하는 모델을 만든다. 사물을 바운딩하는 **detection** 보다 자세하게 위치를 표시해야 하기에 어려운 문제에 해당한다. 전체 이미지에 라벨 하나가 붙여지는 대신 입력 이미지의 각 픽셀에 대한 범주 레이블을 지정하는 것이  **semantic segmentation**이다. 

### 2. Image segmentation

우리가 해내야할 과제, Image segmentaiton에 대해 알아보자. 근본적으로 Image segmentation이란 무엇인가? 이미지의 영역을 분할해서 각 object에 맞게 합쳐주는 것을 말한다. 

![computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%201.png](computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%201.png)

image segmentation의 대표적인 두 가지 방식

위 그림에서 보이듯이 Semantic segmentation은 object segmentation을 하되 같은 class인 object들은 같은 영역이나 혹은 색으로 분할을 하는 과정이다. 반대로 instance segmentation은 같은 class이지만 서로 다른 instance로 구분을 해주는 것이다. 따라서 object가 겹칠 시 하나의 object로 구분해버리는 semantic segmentation에서의 문제를 instance segmentation을 통해서 해결 가능하다.

### 3.  semantic segmentation VS instance segmentation

semantic segmentation은 각 픽셀 별로 어떤 class에 속하는지 label을 구해줘야 한다.

![computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%202.png](computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%202.png)

one-hot encoding을 통해 각 class에 대해 class개수 만큼 출력 채널을 만들어준다.

![computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%203.png](computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%203.png)

각 class별로 출력 채널을 만든 후 argmax한다.

semantic segmentation 각 픽셀들이 class에 따라 binary하게 포함 되는지 여부만 가려낸다. 강아지 output channel에서 각 픽셀들에 대해 강아지에 포함되는 픽셀인지 아닌지 0과 1로 binary하게 값을 찾아준다. 하지만 그렇게 되면 같은 class의 object들에 대해 서로 구분지을 수 없단 단점이 존재한다.

![computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%204.png](computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%204.png)

실제 우리에게 주어진 라벨링된 데이터 차들이 한데 뭉퉁그려져서 분류되어 있다.

instance segmentation는 각 픽셀별로 어떤 카테고리에 속하는지 계산하는게 아니라 각 픽셀별로 object가 있는지 여부를 계산한다. 일반적으로 mask R-CNN 같이 2-stage detector에서 먼저 object들을 바운딩박스를 해서 localization시킨다. 그 위에 class별로 output 채널을 만들것 같이 localize된 ROI 마다 class 개수 만큼 binary mask를 씌워준다. 이미지 사이즈 크기로 class 개수만큼 output 채널이 존재하지 않고 ROI 별로 class 개수만큼 output 채널이 존재하고 동일 class라더라도 서로 다른 instance부분만 value를 가진다.

![computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%205.png](computer%20vision%20ccf95c579f3d436381bda0dd6c5ef1f3/Untitled%205.png)

### 4. 결론

class label이 10개 존재한다면...

 semantic segmentation에서는 각 픽셀이 어떤 class에 포함되는지 아닌지를 10개의 class에 대해서만 각각 binary하게 계산을 한다.

instance segmentation는 localization을 수행하고 box가 focus하고 있는 instance의 픽셀을 각 박스에 대해 분류를 하는데 동일 class여도 서로 다른 instance면 value를 가지지 않는다.