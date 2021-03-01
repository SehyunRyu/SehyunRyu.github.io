---
title: 논문 리뷰-Finding Tiny Faces +느낀점
categories: [Privacy_Preserving_Deep-Learning]
tags:
seo:
date_modified: 2021-03-01 19:00:00 +0900
---

지난 주 상담 때 교수님께서 해당 논문이 연구주제와 연관도 있고, 자료도 잘 되어있으니 실습을 해보라고 말씀해주셨다. 그래서 바로 해당 논문의 내용을 [Pytorch로 구현한 코드](https://github.com/varunagrawal/tiny-faces-pytorch)를 봤는데, 제대로 알 수도 없었고 우선 논문으로 어떤 과정을 구현하는지 알아야하지 않을까 싶었다. 그래서 먼저 논문부터 읽기로 했다.  
<br/>

### Finding Tiny Faces (Peiyun Hu et al.) 요약
[논문 링크](https://arxiv.org/abs/1612.04402)  

![Image Pyramid + Template](/assets/img/post/2021-3-1/image_pyramid.jpg)  
우선 본 논문은 제목과 같이, 어떻게 하면 image 내에 있는 작은 Target Object들을 잘 Detection 해낼 수 있는지에 대한 방법을 제시한 논문이다. 역사적으로 처음에 작은 Target Object를 Detection 할 때에는 (a)의 방식처럼 Single-Scale Template을 사용했다고 한다. 그리고 이를 원본 image를 여러 크기로 늘리고 줄인 Image Pyramid에 대해 Detection을 시켰다고 한다. 그런데 Target Object의 크기에 따라 Resolution이 다를 것이므로, 반대로 하나의 image에 여러 Size와 그에 맞게 Resolution이 바뀐 Template들을 적용시키는 방식이 그 다음으로 나온 (b)이다. 그리고 해당 논문에서는 Extreme Case들까지 다루기 위해 (a)와 (b)의 방식을 합쳐, Coarse한 Image Pyramid를 만들고 그곳에 여러 Size와 Resolution의 Template들을 적용시켰다. 또 (d)를 보면 이에 더하여 Template에 Context까지 포함을 시켰음을 알 수 있다.  
<br/>

##### Different-Size Object Detection에 기여한 바
해당 논문은 크게 세 가지 점에서 기여를 했다 볼 수 있는데, 우선 Object Detection에 있어 주변 배경과 같은 Context가 중요하다는 것을 보였다. 그리고 Target Object의 Size보다는 큰 Template을 사용해 Detection 할 때, 또 Object의 Size가 작고 큰지에 따라 Template의 Resolution이 높고 낮은 것이 각각 좋다는 것을 보였다. Template을 사용한 Object Detection은 image를 전체적으로 훑으며 Template과 유사한 부분을 찾아내 Bounding Box를 만드는 방법이라 볼 수 있다.  
<br/>

![Context](/assets/img/post/2021-3-1/context.jpg)  
1. 이들은 직관적으로 사람이 사물을 인식할 때에도 Context가 포함되었을 때 Image Recognition을 더 정확하게 할 수 있을 것이라 생각했고, 위의 실험 결과와 같이 그 생각이 타당함을 보였다. 특히 Size가 작고 Resolution이 낮은 경우일 수록 Context가 주어짐에 따른 인식률이 크게 차이가 남을 볼 수 있다. 상황을 상상해보면 정말 그럴 것 같다 생각할 수 있다. 여튼 이러한 이유로 그들은 Template에 Context를 추가하였고 이것이 인식률을 높임을 보였다.   
<br/>

![Size](/assets/img/post/2021-3-1/size.jpg)
2. 또 이들은 실험을 통해 Taget Object보다 Template의 Size가 작을 경우 Dtection을 잘 하지 못한다는 것을 보였다. 위의 사진과 같이 Terget Image와 Template 사이의 Size 차이가 너무 크면 또 정확도가 낮아지지만, 일반적으로 Template이 Target Object보다 커야 Detector의 성능이 향상된다. 직관적으로 생각해도, 사람의 코만 봤을 때와 얼굴 전체를 보았을 때, 얼굴 전체가 나온 사진을 얼굴이 어디있는지 알기 더 쉬울 것이다. 더구나 image가 굉장히 작다면, 해당 사진이 그냥 살구색 사각형으로만 보일 것이다.  
<br/>

![Resolution](/assets/img/post/2021-3-1/resolution.jpg)
3. 마지막으로 이들은 Small Face를 인식하는 데에는 해상도를 (2배 만큼)높게 만든 Template으로 학습시킨 결과가 더 좋고, Large Face에는 해상도를 (반만큼) 낮춘 Template을 사용했을 때 결과가 더 좋다는 것을 보였다. Taget Object의 Pixel 크기에 따라 최적에 근접한 해상도를 알 수 있다고 하는데, 정확히 어떻게 알아내는지는 모르겠다(사실 이해할 시간을 투자할 만큼 중요한 내용은 아닌 것 같아서). 궁금하면 해당 논문을 읽어봐도 좋겠고, 여튼 이들은 Taget Object의 Height를 40px과 140px를 기준으로 하여 Resolution을 0.5x, 1x, 2x 만큼 변형시킨 Template을 사용했다고 한다.  
<br/>  

##### Architecture
![Architecture](/assets/img/post/2021-3-1/architecture.jpg)  
그래서 Neural Network의 전체적인 Architecture는 위와 같다. image를 각각 0.5x, 1x, 2x하여 Image pyramid를 만들고, CNN을 통과시킨다. 그리고 서로 다른 Size와 Resolution의 Template들에 대해 detection을 하기 위해, 각 Size의 CNN output들을 A-type과 B-type으로 나누었다. 그 뒤 Detected Face들의 Bounding Box 결과들을 하나로 통합하여 Loss를 평가해 학습하는 방식으로 보인다. 부가적으로 CNN에는 ResNEt101,ResNet50, VGG16을 적용했는데 그 중 가장 결과가 좋은 ResNet101을 택했다고 한다.  
<br/>

### + Object Detection은 어떻게 하지?
Object Detection은 image 속에 있는 Object들에 대한 Multi-Class Classification, 해당 Object가 있는 곳에 대한 Bounding Box Regression의 두 가지 작업이 합쳐진 Task이다. 생각해보니 Object Detection이 기본적으로 어떻게 이루어지는지도 몰랐기에, 추가적으로 짧게 정리해본다.
<br/>

![Stage](/assets/img/post/2021-3-1/stage.jpg)  
Object Detector에는 크게 Many Stages Detector와 One Stage Detector의 두 가지 종류가 있다. 우선 Many Stages Detector는 직관적으로 좀 더 이해하기가 쉽다. 먼저 Bounding Box를 만들 수 있을만한 위치들을 Propose하고, 그 영역 내의 image를 Classification하기에 Stage가 여러 개로 구성된다 생각할 수 있다. 그렇지만 One Stage Detector는 Image Classification과 Bounding Box Regression을 동시에 해낸다.  
<br/>

논문의 References에 있던 가장 처음에 나온 R-CNN은 Two Stage Detector에 해당되며, Object Detection에 CNN을 처음으로 적용했다고 한다. 처음에는 Proposed Bounding Box 내의 image를 찌그러뜨려 Imput Size를 동일하게 맞추고 CNN에 넣어 Classification 했다. 그리고 이에서 파생된 Fast R-CNN은 image를 찌그러뜨리지 않고, ROI Pooling이라는 방식을 통해 Input Size를 동일하게 맞춤으로써 속도를 빠르게 만들었다. 이 ROI Pooling은 Stride의 크기를 다르게 해, 서로 다른 image들을 Pooling 시켜 결국 같은 Output Size를 갖게 하는 방식이라 생각할 수 있다.  
<br/>

그러면 Bounding Box Proposal은 어떻게 할까. 처음에는 Selective Search Algorithm이라는 방식을 사용했는데, 이는 또 Hierarchical Grouping Algorithm을 기반으로 한다고 한다. 직관적으로 간단히 생각했을 때, Random하게 Bounding Box들을 만든 뒤 컬러나 밝기와 같은 그 안의 image 특성이 비슷하며 겹치는 Bounding Box들을 합쳐 새로운 하나로 만드는 방식이라 볼 수 있다. 이후 변형된 Model들에서는 Selective Search 대신 RPN(Region Proposal Network)을 사용해 Bounding Box Proposal을 함으로써 시간을 굉장히 단축시켰다.  
<br/>

### 논문에 대한 의문점 +약간의 코드 분석
![Demo Image](/assets/img/post/2021-3-1/demo.jpg)  
자 위의 내용을 보면 원래 읽던 논문에서의 Detection 방식과는 꽤 차이가 있음을 알 수 있다. 위에서 설명한 R-CNN 계열의 Detector들은 Bounding Box Proposal의 과정이 있지만, 본 논문과 같이 Template을 사용한 Detection은 앞서 언급한대로 image를 훑으며 Similarity를 Matrix로 반환해 일정 역치를 넘는지 확인하는 방식이다. 그러면 이 논문의 Architecture에서는 Bounding Box를 어떻게 만들까? Demo Image를 보면 알 수 있듯이, Bounding Box들의 크기와 형태는 굉장히 다양하다. 그만큼 많은 Template을 사용했다는 얘기일까?  
<br/>

![Template Json](/assets/img/post/2021-3-1/template_json.jpg)  
위에 있는 링크의 해당 논문을 PyTorch로 구현한 코드를 보면, 총 25개의 template이 있음을 알 수 있다. 위의 사진은 Template을 만들기 위한 Dataset image들 중 일부의 Groundtruth Bounding Box Coordinate이라 보면 될 것이다. 
<br/>

![Refinement](/assets/img/post/2021-3-1/what_the.jpg)  
간략하게 코드를 봤을 때, 해당 함수에 의해 Bounding Box의 좌표와 크기가 Refinement 됨을 알 수 있다. 즉 Template의 크기 그대로 Bounding Box를 만드는 것이 아니라 크기에 변형이 생긴다는 것이다. 그런데 갑자기 왠 Exponential이 등장했는지 의문스러웠다.
<br/>

![Log](/assets/img/post/2021-3-1/log.jpg)
아마 [Fast R-CNN RPN논문](https://arxiv.org/pdf/1506.01497.pdf)에 나오는 내용 중, 위의 사진과 같은 Bounding Box Regression의 식에서 나온 Log가 연관이 되어 Exponential이 나온 게 아닐까 싶다. 정확한 의미는 모르겠으나, 우선 더 정확히 보지 않고 넘어갔다.  
<br/>

![Indices](/assets/img/post/2021-3-1/indices.jpg)  
그런데 Bounding Box Regression을 하기 전, Template으로 image를 훑는 과정은 어디에 있는지 의문이 들었다. 그런데 Bounding Box를 만드는 함수 내에서 이미 어디서 'prob_cls > prob_thresh'인지 찾았고, 즉 Probability가 Threshold를 넘는 indices들에 대해 밑부분의 Bounding Box를 만들고 고치는 과정을 진행함을 알 수 있다.  
<br/>

![prob_cls]](/assets/img/post/2021-3-1/prob_cls.jpg)  
이 prob_cls는 score_cls를 Sigmoid(개별 Neuron의 Threshold를 재현한 함수의 일종)에 통과시킨 결과이고, score_cls는 Model에 image를 넣은 output의 결과임을 알 수 있다. 이 말은 Model에 image를 통과시켰을 때에 이미 어느 부분에 얼굴이 있을지 안다는 이야기일 것이다. 즉 CNN을 통과하며 image의 어떤 위치에 얼굴이 있을지 대략적으로 안다는 말 같은데, 이럴 것이면 왜 Template을 만드는지 이해를 할 수가 없다. 굉장히 작은 얼굴을 이미 CNN만으로 Detection 할 수 있다면 Template이 왜 필요한 것인지, 순전히 Bounding Box를 만들기 위해 필요한 것인지 의문이 든다.  
<br/>

이 부분들에 대해서 더 언급을 했다가는 글이 너무 길어질 것 같아 여기서 끊고, 다음 글에서 내용을 이어 작성하려 한다. 다음에는 코드를 보다 자세히 보며 전체적인 흐름이 어떻게 되는지 파악하는 것이 목표이다.  
<br/>

### 느낀점 +연구실 포닥 선배의 조언
진호가 얘기했듯 무엇을 모르는 지에 대해 모르는 것이 정말 큰 문제인데, 최근 내가 정말 그러하다는 것을 느꼈다. 학교에서 하는 공부들은 대개 문제도 주어지고, 풀기 위해 알아야할 것은 대부분 교과서나 가이드라인 안에 있고, 해결하고 난 뒤에는 답도 명료한 형태로 나온다. 그런데 독학을 하는 과정에서는 뭐 하나 깔끔하게 정의된 것이 없는 듯 하다. 내가 정확히 무엇을 어느 정도까지 알아야 하는지 모르니, 처음부터 너무 세부적인 사항에 빠져들어 막막해지는 과정을 반복했다. 배경지식을 알기 위해 찾아본 자료의 또 다른 배경지식으로 들어가다 나오기도 많이 했다. 그렇게 목적과는 멀어져 시간을 쓰며 스스로 참 일머리가 없고 답답하다 느꼈다.  
<br/>

##### 1. 우선 이 일을 하는 목적이 무엇인지 명확히 정의
과한 일반화를 시켜 어떤 일을 하기 전에는 항상, '1. 우선 이 일을 하는 목적이 무엇인지 명확히 정의'해야 하는 것 같다. 논문을 읽다 모르는 개념이 나와서 찾아봤는데, 그 개념과 관련해서 알아보던 중 또 모르는 것이 생겨 내려가고... 이런 과정은 분명 유익하며 그런 가운데 큰 흐름을 느낄 수 있을 것이지만, 현재 목적과는 너무 멀어지는 정도로 디테일에 빠져들 수 있다. 그렇게 환원적으로 계속해 내려가다보면 기본적인 수학 이론들에 닿게 될 텐데, 그 시간은 엄청날 것이다. 한 분야의 전문가가 되기 위해 그런 것이 필요하다 보지만, 당장에 이루어야 할 목적이 있다면 그런 과정은 이후로 미루어두는 것이 적절하지 않은가 생각한다. 무엇이 목적인지 명확히 정의한다면 내가 당장 필요한 정도에서만 무언가를 얻어갈 수 있지 않을까.  
<br/>

##### 2. 일단 큰 흐름을 파악한 뒤 세부사항을 확인하기
학교에서 수업을 들을 때나 책을 읽을 때에는 목차가 깔끔하게 정리되어 있다. 목차를 읽으며 전체적으로 이 배움의 흐름과 목적이 생각하고, 그 다음 세부사항을 알아보는 공부방식이 효율적이라 생각한다. 그런데 혼자 어떤 일을 할 때에는 그런 목차가 정의되어 있지 않고, 큰 흐름을 놓치기 쉬운 것 같다. 그렇기에 자체적으로 큰 흐름을 먼저 파악한 뒤 세부사항을 확인하는 습관을 가져야 하지 않을까 싶다. 무언가를 설계할 때에도 마찬가지가 아닌가. 건축가가 전체적으로 어떤 목적과 형태의 건물을 만들 것인지 보다, 우선 사용할 기둥의 재료와 이것을 구할 가게를 고민하고 있는 모습은 말이 안되지 않은가. 분명 개인마다 학습 및 업무에 대한 철학이 다르겠고 나도 상황마다 다르다고 생각하지만, 경험에 의한 판단으로 대개 Top Down 방식이 효율적이지 않은가 생각한다. 이것을 놓쳐 효율을 잃고 싶지 않다.  
<br/>

더불어 연구실 옆자리에 계신 친절하고 똑똑한 포닥 선배님께서 해주신 조언을 짧게 적어본다.
1. 비슷한 작업을 하는 코드들은 Pipeline이 비슷하니 하나만 확실히 공부하면 다른 것들은 이해가 쉽다.
2. 처음에는 대략적으로 보고, 그 다음 각 함수별 인풋 아웃풋을 명확히 확인해본다.
3. Pycharm과 같은 IDE로 직접 코드를 한 줄씩 실행해보며 어떤 기능을 하는지 확인해본다.  
<br/> 
<br/>   
<br/>  
*개인적인 질문 및 조언과 오류에 대한 지적은 늘 감사합니다!