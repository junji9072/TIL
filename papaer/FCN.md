# FCN

Abstract

합성곱 신경망(Convolutional Neural Network; CNN)은 입력 이미지의 특성의 계층을 얻는 강력한 모델이다. 본 논문에서는 CNN만으로 신경망을 구성하였으며, end-to-end, pixel-to-pixel 방식으로 학습을 하였다. 본 논문에서 주의깊게 봐야할 점은 처음부터 끝까지의 모든 layer를 Convolution layer로 구성했다는 점이다. 본 논문에서는 이를 fully convolutional network(FCN)이라고 표현하였다.

이 신경망의 특성은 임의의 사이즈를 갖는 입력 이미지를 사용할 수 있고, 입력 이미지에 대응되는 동일한 크기의 출력 이미지를 효율적인 추론(inference)와 학습(learning)을 할 수 있다. 본 논문은 먼저 FCN space에 대한 상세하게 정의, FCN이 공간 밀도 예측 작업으로의 응용을 설명, 마지막으로 이전에 나왔던 대표적인 신경망(AlexNet, VGG, GoogleNet)을 FCN으로 바꾸고 segmentation을 위해서 fine-tuned를 통해 이 신경망들이 다른 데이터를 통해 배웠던 데이터 셋의 표현을 전이학습하는 방식 진행된다. 

Adapting classifiers for dense prediction

FCN 이전에 유명했던 대표적인 신경망(AlexNet, VGG, GoogleNet)들은 반드시 고정된 입력 이미지의 크기를 입력 받게 되는데 그렇게 되면 위치 정보가 사라지고 크기가 고정된다는 문제가 발생한다. 이러한 출력의 문제는 단순 분류 결과만 나타나게 된다. 예를들어 강아지, 고양이를 분류하기 위해 출력이 단순히 0,1 이렇게만 출력이 된다. 

![FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled.png](FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled.png)

위 그림이 이전 단순 FCL(fully conneted layer)에서 convolutionalization을 통해 convolution layer로 바꾸었다.

이러한 과정을 거치게 된다면 FCL과 달리 위치정보나 class에 대한 정보를 잃지 않게 된다. tabby cat heatmap을 보면 고양이에 대한 정보뿐 아니라 위치 정보도 함께 가지는걸 확인할 수있다. convolution layer는 필터의 크기만 맞다면 어떠한 input이 오더라도 수용이 가능하여 input의 크기에 제약을 받지 않는다.

![FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%201.png](FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%201.png)

FCL 대신에 convolution layer(1x1 conv 레이어)를 추가해준다. 

이 결과물은 결국 각 클래스의 feature map 상에서 segmentation이 된다. 즉, fc 레이어들을 1x1 convolution으로 하면 위치 정보도 남기때문에 heatmap그림에서와 같이 고양이 위치 값이 남아있는걸 확인 할 수 있다.

shift-and-stitch is filter rarefaction

간단히 말하자면 입력 이미지를 조금씩 이동시켜서 출력 영상들을 겹쳐서 출력으로 만드는 것이다. upsampling을 사용하지 않는다면 최종 출력 이미지의 해상도는 낮다. 예를 들어 100 x 100의 입력이미지를 넣는다면 10x10 최종 출력 이미지를 얻게 될 것이다. 하지만 semantic segmentation을 위해선 입력과 출력은 크기가 동일해야 한다. 하지만 출력 이미지 그대로 upsampling 하면 정확하지 않을 것이다. 하지만 최종 출력 이미지를 이동해서 출력을 얻고 이동해서 출력을 얻는것을 반복한다면 여러 출력 이미지를 결합해서 사용하면 될 것이다.

![FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%202.png](FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%202.png)

빨간 박스는 2x2 크기로 max pooling 필터로 stride는 2로 2칸씩 이동해서 3x3 연산 결과

![FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%203.png](FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%203.png)

위와 동일하지면 왼쪽으로 1 픽셀 이동해서 결과도 다르다. 결과 행렬이 오른쪽이 회색인것은 입력 이미지가 왼쪽으로 1 이동해서 오른쪽 회색 필셀 영역을 필요하지 않기 때문이다.

이러한 과정을 반복해서 결과들을 합쳐서 출력물을 얻게 됩니다. 이 알고리즘의 단점은 계산 비용이 큰 것이다. 그래서 upsampling을 위해 트릭을 쓰는데, max pooling을 쓰지 않고 학습가능 가중치로 필터를 채운다. 하지만 여기서는 쓰지 않고 skip connection을 통해서 구현을 하게 된다.

Upsampling is backward strided convolution

낮은 해상도의 이미지를 높은 해상도의 이미지로 upsampling이 목표였다. 이를 위한 대표적인 방법이 보간법(interpolation)이다. 본 논문에서는 backward convolution(deconvolution, transposed convolution)이라는 학습 기반 upsampling을 적용했다. backward convolution과 activation function을 연결하여 nonlinear upsampling을 학습하게 된다.

Patchwise training is loss sampling

Patchwise training는 모델을 학습할 시 전체 이미지를 넣는게 아니라 객체 주변을 분리한 서브 이미지를 사용하는 방법을 말한다. 이미지 부분을 학습하는 patch 학습과 전체 이미지 학습하는 fully convolution training 동일하며 계산적으로 유리하다고 썼다.

Segmentation

본 논문은 FCL FCN으로 변경하고 픽셀별 loss 계산을 위해 upsampling을 했다. fine-tuning을 통해 segmentation을 위한 학습을 진행하고 skip connection을 추가했다. 이를 추가한 네트워크 구조를 skip architecture라 하는데 end-to-end로 학습하게 된다. 

![FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%204.png](FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%204.png)

먼저, FCN-16은 pool4레이어 에서 1x1 합성곱 레이어를 적용하고, 마지막 레이어에서 x2 upsampling을 한 뒤에 합친다. 마지막 레이어에서 x32배 upsampling 하지 않고 x16배 upsampling 하면 입력 이미지의 크기와 동일한 출력 이미지를 얻을 수 있다. FCN-8은 FCN-16에서 x16배 upsampling 하기전 결과에서 x2 upsampling을 한 뒤 pool3 레이어에서 1x1 합성곱 레이어를 적용하여 하나로 합친다. 그리고 x8배 upsampling을 하게 된다 위 과정에서 단순히 마지막 x32배 upsampling 한것은 FCN-32라고 한다. 이러한 결과를 시각화 해서 보자.

![FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%205.png](FCN%20f8dd5d597bcf49d99c5666395a836003/Untitled%205.png)

FCN-32 경우 한번에 x32배 upsampling 하면 많은 정보량이 손실이 있는것을 볼 수 있다. 이전 레이어의 feature map을 써서 skip conection을 사용한다. 마지막 conv 레이어 결과를 2배 upsampling 하고 마지막 pool5 이전 단계 결과와 합쳐준다. 그것을 x16배 upsampling 하면 FCN-16의 결과가 나온다. 

정리

기존의 모델을 semantic segmentation 목적에 맞게 수정해서 전이학습을 했다. 입력과 출력 이미지 크기가 동일 해야 했기에 upsampling을 했지만 이 과정에서 많은 정보 손실이 있었다. 이를 극복하기 위해 skip connection을 통해 정보 손실 방지와 좋은 성능을 낼 수 있도록 하였다.

---

[https://everyday-image-processing.tistory.com/32](https://everyday-image-processing.tistory.com/32)

[https://modulabs-biomedical.github.io/FCN](https://modulabs-biomedical.github.io/FCN)

[https://medium.com/@msmapark2/fcn-논문-리뷰-fully-convolutional-networks-for-semantic-segmentation-81f016d76204](https://medium.com/@msmapark2/fcn-%EB%85%BC%EB%AC%B8-%EB%A6%AC%EB%B7%B0-fully-convolutional-networks-for-semantic-segmentation-81f016d76204)

[https://woans0104.tistory.com/2](https://woans0104.tistory.com/2)