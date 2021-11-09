# SEGNET

abstract

pixel-wise segmentation 모델을 제안한다.

encoder-decoder로 아키텍처로 encoder는 f.c layer 제외하고 VGG16을 쓰고 학습 파라미터 필요없는 un-maxpooling 이용한 업샘플링 한다.

decoder에서 업샘플링된 피쳐맵은 convolution layer를 통해 dense feature map으로 만든다.

소개

maxpooling, sub-sampling 하다보면 조잡한 피쳐맵이 만들어지게 된다. 이것은 segnet이 하고자하는 pixel-wise prediction을 해야하는 segmentation task에서 정보가 부족해서 좋은 결과를 내지 못한다. 이러한 low resolution feature 로부터 intput 이미지와 동일한 크기로 정확하게 boundary localization이 가능한 아키텍처를 만들고자 제안되었다. 

![SEGNET%20dfafad90f2d040378cc751638f2f24f8/Untitled.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/8d4abf48-e405-4cca-8d1b-cf0b0ce614fd/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T121651Z&X-Amz-Expires=86400&X-Amz-Signature=fb2ebfeb9d6a6ab1ce7b4e7a39340cd2ae8894e28e400ba471b6d5d03452dfef&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

encoder

VGG16에서 fclayer 제외하고 13개 convolution layers와 동일하다. 제외한 이유는 메모리 사용량이나 계산량 등에서 강점을 가지기 위해 제외하였다. max-pooling과정에서 어떤 픽셀이 선택되는지 maximum featrue value의 위치를 저장하는 방식을 선택한다 만약 2x2 사이즈의 pooling window를 쓰면 2비트의 저장공간이 필요하기에 말하지 않아도 사이즈가 줄어든다. 

VGG16

기존 연구에 사용되던 Local Respnse Normalization에 대한 구조를 실험을 통해 확인하면서 깊은 BCDE구조로 깊어지면서 분류 에러가 감소하는것을 관찰했다. 깊어질수록 성능이 좋아진다.

VGG16의 큰 특징은 3x3 필터를 사용한다는 점이다. 작은 필터만으로도 정확도를 개선시켰다. 

![SEGNET%20dfafad90f2d040378cc751638f2f24f8/Untitled%201.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/000c7ef4-95b0-4a66-b46e-1b9624623f27/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T121714Z&X-Amz-Expires=86400&X-Amz-Signature=39d87b8c96fbcd00efc0f85ac26e55007c6fe297818a26a1c178669073a33f92&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

가중치 = 파라미터 개수의 차이가 있다. 3x3 필터가 3개면 27개의 가중치를 가지고 7x7 필터는 49개의 가중치를 가진다. 학습 속도가 빨라지며 층의 개수가 늘어나면서 비선형성을 증가시킨다. 이는 특징 식별성 증가로 이어진다. 

![SEGNET%20dfafad90f2d040378cc751638f2f24f8/Untitled%202.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/000f59fd-2788-47bb-a642-bc3587659068/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T121725Z&X-Amz-Expires=86400&X-Amz-Signature=428f1c6916ab9a7b2a04140256c933cb02e16417800371b57d2888ef07422e15&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

1층(conv1_1) : 64개의 3x3x3 필터 커널로 입력 이미지를 컨볼루션 해주며 zero padding은 1 stride도 1도 해주며 모든층 동일한 설정이다. 이렇게 하고 마지막 층 제외하고 모든 충에 활성화 함수가 적용된다.

2층 : 64개의 3x3x64 필터 커널로 특성맵 컨볼루션한다. 224x224x64생성되고 2x2 maxpooling을 stride2로 적용해서 112x112x64로 줄여준다.

3층 : 128개의 3x3x64 필터 커널로 112x112x128이 나온다.

생략

14층 7x7x512 특성맵을 flatten해준다. 전 층의 출력을 받아 단순하게 1차원 벡터로 펼쳐준다. 7x7x512 = 25088개 뉴런이 되고 1x1사이즈의 4096장의 동일한 사이즈로 만들어준다. 훈련시에는 dropout을 적용해준다.

15층 4096뉴런으로 구성해주고 14층 뉴런과 fully connected되며 dropout적용해준다.

16층 1000개의 뉴런으로 구성되고 15층의 뉴런과 fully connected된다. 출력값은 softmax함수로 활성화되며 1000개의 뉴런이자 클래스로 분류하는 목적 네트워크이다.

decoder

업샘플링 과정에 앞서서 저장했던 pooling indices를 활용하게 된다 아래 그림을 보자면 max pooling 과정에서 선택된 index들의 위치로 값이 이동하는것을 볼 수 있다. 이렇게 하면 0이 많은 sparse feature map이 만들어지는데 여기에 trainable decoder filter를 이후에 진행해 빈 부분을 메꾸는 작업을 한다. 이후 batch normalization layer 거친 후 soft-max classifier통해서 pixel-wise prediction을 수행한다.

![SEGNET%20dfafad90f2d040378cc751638f2f24f8/Untitled%203.png](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/1d1e55b9-6233-4a4b-957c-17ff95ff4cf1/Untitled.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20211109%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20211109T121741Z&X-Amz-Expires=86400&X-Amz-Signature=d56ffa4989fb9f78e3fc964b091bf29783f63c2ce916ecece6f421aa8b166ca3&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22Untitled.png%22)

segnet은 업샘플링에서 max-pooling indices로 추가 학습 없이 진행한다. feature map에 decoder filter를 추가하는 방법을 선택했다. 이를 통해서 boundary에 대한 정확한 기술을 통한 정확도 향상 파라메터 수의 감소를 통한 efficiency향상을 이뤄냈다. 

un-maxpooling

업샘플링을 위한 것이며 deconvolution layer를 쓰지 않기에 별도 학습 파라미터가 없고 전체 모델은 여전히 end-to-end 학습이 가능하며 다른 구조에 응용할 수 있어서 수정이 용이하다. 
