---
title: 논문 리뷰-Privacy-Protection Drone Patrol System based on Face Anonymization +앞으로의 학습 계획
categories: [Privacy_Preserving_Deep-Learning]
tags:
seo:
date_modified: 2021-02-10 11:59:00 +0900
---

### 연구참여를 하게 되었고, 주제가 정해졌다!
지도교수님과 컨택을 하여 이번학기 부터 올해 말까지 POSTECH Advanced Information Systems Lab에서 연구참여를 하게 되었다. 줌을 통해 화상으로 교수님과 면담을 하게 되었고, 여름방학 때 학교 Computer Vision Lab에서 연구참여를 하며 Stanford University의 CS231n 수업을 듣고 몇 편의 논문을 읽었던 것들이 배경이 되어 Privacy Preserving Deep-Learning이라는 주제에 대해 알아보기로 했다. 교수님께서 연구의 요점을 간략히 말씀해주셨는데 semantic segmentation, GAN, Normalization 등 그래도 computer vision 관련 개념들을 공부했기에 쉽게 이해가 되었다. 나름 참 뿌듯했고, 교수님께서 기한을 주고 성과물을 요구하실 거라는 말씀에 많은 것을 배울 수 있을 것이라 기대가 됐다. 또 서류처리를 하며 연구비 지원을 위해 국가연구자번호도 발급 받으니 정말 열심히 해야겠다는 생각이 들었다. 글을 쓰는 것이 생각보다 귀찮은 일이지만, 개인적인 생각을 남기는 차원에서라도 해당 주제 관련 공부 내용들을 Privacy_Preserving_Deep-Learning이라는 태그로 글을 조금 써보려고 한다.  
  
우선 방학이 끝나기 전 교수님께서 object detection이나 GAN 모델을 코드로 실습해보라 말씀해주셨다. CV Lab에 있을 때 이론만 공부했지 코드는 대략 보기만 했을 뿐 실제로 구현해본 적은 없었다. 그래서 CS231n의 assignment를 풀어보려 했는데, 이게 맞나 하는 생각이 들었다.  
첫째로 알아먹을 수가 없었다. [numpy](https://numpy.org/doc/stable/user/whatisnumpy.html), [matplotlib.pyplot](https://matplotlib.org/3.1.1/api/_as_gen/matplotlib.pyplot.html), [pytorch](https://pytorch.org/docs/stable/index.html)와 같은 라이브러리들을 사용하는 데, 대체 뭐가 정확히 어떻게 되어먹은지 알 수 없었다. 물론 긴 document를 한참 보면 알 수 있을 것이고 당연히 알면 좋겠지만, 내가 우선적으로 해야할 게 이것인지 의문이 들었다. 어느 정도까지 알아야 하고, 어느 수준에서 코드를 수정해야 하는지 감이 안 왔다. 내가 state-of-the-art 수준의 segmentation을 만들어야 한다던지는 아닐테니까... 해당 기술들을 사용하기 위해 어느 수준까지 알아야 하는 것인지 의문이 들었다. 그래서 우선 연구실의 논문을 찾아 읽고 감을 잡아야겠다 생각했다.  
  
혹시 이 글을 읽어주시는 분들 중 GAN, loss function 등의 내용이 뭔지 답답하시다면, 기본적인 내용은 검색이나 youtube에서 CS231n 수업을 들어보시는 것도 좋을 것 같습니다. 당연히 논문을 읽으셔도 되겠지만요ㅎㅎ 저도 잘 모르는 만큼 배우려 하는 중이고, 나중에 그런 내용들도 정리할 날이 오지 않을까 싶기는 하지만요..! 더 자세히 아시는 분들은 지적 및 알려주시면 감사하고요ㅎㅎ  
  
### Privacy-Protection Drone Patrol System based on Face Anonymization (Harim Lee et al.) 요약
[논문 링크](https://arxiv.org/abs/2005.14390)  
![Fig 2.](/assets/img/post/2021-2-10/Fig2.jpg)  
Abstract의 시작 부분에서는 로봇 시장이 커져가고 있으며, 그에 따라 드론에 찍힌 사람의 얼굴이 노출되는 등의 사생활 침해 위협이 있다고 말한다. 그리고 해당 논문은 그런 사생활 침해 문제를 해결하기 위하여, robot perception을 유지하는 face-anonymizing algorithm을 개발했다. Anonymizing의 큰 그림은 Photorealistic한 image를 Segmenatation Learning Part를 거쳐 Segmentation image로, 그리고 Synthesis Learning Part를 거쳐 다시 Photorealistic한 image로 바꾸는 것이다. 이를 통해 10번 refrence인 Jason et al.의 Algorithm은 원본 image의 특성을 담아낸다는 문제를 해결하였다! 이를 위해서는 Segmentation과 Synthesis를 수행하는 어떠한 [GAN](https://arxiv.org/abs/1406.2661) framework든 사용할 수 있지만, 해당 논문에서는 각각의 과정에 작성 시점의 두 가지 state-of-the-art인 [CycleGAN](https://arxiv.org/abs/1703.10593)과 [GauGAN](https://arxiv.org/abs/1903.07291)의 Loss function을 수정하여 사용했다.  
결과적으로는 [Siamese Network](https://www.cs.cmu.edu/~rsalakhu/papers/oneshot1.pdf)를 통해 계산한 두 image 사이의 Euclidean distance가 동일 인물의 image 사이 값보다 크기에 Privacy-Preserving이 효과적임을 보이고, 드론과 같은 로봇이 물체를 인식하고 주변 환경에서 본인의 위치를 파악하는 데에도 문제가 발생하지 않음을 [ORB-SLAM2](https://arxiv.org/abs/1610.06475)를 통해 확인하였다.
  
### 솔직히 모르겠는 Loss function 수정 내용
![Fig 3.](/assets/img/post/2021-2-10/Fig3.jpg)  
논문의 4~7p 즈음에 걸쳐 Loss function을 정확히 어떻게 수정하였는지에 대해 내용이 나온다. 그런데 수식의 내용이 복잡해 보이고, 무엇보다 CycleGAN과 GauGAN이 Loss function을 어떻게 만들었는지 알아야 수정한 것을 알 수 있기에 저 논문들을 확인해봐야 할 것이다. 어찌보면 논문의 핵심은 중간의 GAN Framework가 변경되어 상관이 없는 것이 아닌가 싶기도 하고, 불가능한 완벽을 추구하기 보다는 우선 기록을 남기고 더 학습하자는 취지로 확실히 이해하지 못한 채 이 글을 마치려 한다. 다른 두 논문을 읽고 이해가 되면 올리는 쪽이 더 나을 것 같다..!  
  
### 논문 리뷰 및 앞으로의 학습 계획
해당 논문에서는 두 가지의 GAN Framework를 합쳐 하나의 Architecture를 구성하고 Loss function을 변경하였지만, 개별 Network의 Architecture를 크게 변경하지는 않았다. 연구실의 해당 주제 논문을 읽고나니, 내가 앞으로 해야할 일은 세부적인 Network의 수정이라기 보다 state-of-the-art 수준의 Neural Network Architecture를 활용하는 쪽이라는 느낌이 확실히 온다. 어떤 식으로 Privacy를 보호할지 idea developing이 확실히 핵심적이겠지..!  
다만 기술 및 지식적으로 Loss function을 정확히 어떤 식으로 구성하였는지, epoch 설정 및 normalization을 정확히 어떻게 하는지 등이 기억이 잘 안 난다.(이거 참 심각한 수준이지만!) 그래서 훗날 네트워크를 용도에 맞게 변경하기 위해서라도 이 내용들을 다시 학습해야 할 것으로 보인다. 또 위에서 언급한 CycleGAN과 GauGAN 논문을 읽고 해당 논문의 Loss function 수정 부분을 다시 읽어보면 좋을 것 같다. 그리고 핵심적 내용은 아니지만 Siamese Network가 무엇이고 어떻게 작동하는지도 논문을 읽어봐야 알 것 같다.  
그리고 직접 코드를 만져봐야겠고! 어떤 내용을 어느 정도부터 실습을 해야할지 참 고민이 됐는데, 지금 생각했을 때는 [FCN Semantic Segmentation Network](https://arxiv.org/abs/1411.4038) 코드를 보고 수정해서 돌려보며 앞서 말한 라이브러리들에도 좀 익숙해지면 적절한 수준이 아닐까 싶다. 제 상황에 더 좋아보이는 게 있다면 알려주세요, 약간은 맨땅에 헤딩하는 기분이에요...ㅋㅋ  
정리하자면 우선, 'Loss function/epoch/normalization 등 기본개념 학습 -> FCN Semantic Segmentation Network 코드 실습 -> CycleGAN/GauGAN 등 논문 읽고 다시 이해해보기'의 순서를 거치면 되지 않을까 싶다!  
  
  
*개인적인 질문 및 조언과 오류에 대한 지적은 늘 감사합니다!