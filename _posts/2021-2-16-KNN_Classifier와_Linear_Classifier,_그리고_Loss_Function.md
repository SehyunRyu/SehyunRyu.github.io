---
title: KNN Classifier와 Linear Classifier, 그리고 Loss Function
categories: [Privacy_Preserving_Deep-Learning]
tags:
seo:
date_modified: 2021-02-16 14:15:00 +0900
---

이번 post에서는 지난 번 언급한 학습의 과정에서, 'Loss function/epoch/normalization 등 기본개념 학습'에 해당하는 부분을 개인적으로 수행하고 정리해보고자 한다. 전에 CS231n 강의와 자료를 봤는데 제대로 기억을 못하니, 혼자 보기 위해서도 남겨 놓아야겠다. 자료의 기본적인 내용 및 출처는 모두 [CS231n](https://cs231n.github.io/)을 기반으로 하여 작성한다..!

### k-Nearest Neighbor Classification
Nearest Neighbor는 pixel-wise 값들의 차이의 정도를 따진다는 점에서 Similarity에 대해 생각할 때 가장 간단히 생각할 수 있는 개념일 것이다. 실제 가장 먼저 Image Calssification에 사용된 Algorithm이라는 것 같다.  
  
Nearest Neighbor Calssification은 간단히 말해 pixel-wise 값의 차이가 적은 Training Data Set 상 image의 Category로 분류가 되는 방식이다. 그리고 k-Nearest Neighbor은 pixel-wise 가까운 상위 k개 image를 고려하여 Category를 분류하는 방식이다. 보다 정확히는, [Wikipedia에서 보면](https://en.wikipedia.org/wiki/K-nearest_neighbors_algorithm) k개 image Category의 inverse distance weighted average된 값이 가장 큰 Category로 결정하는 것으로 보인다.  
  
Test Set의 모든 image에 대하여 Training Set의 '모든' image들과 pixel-wise calculation을 해야한다는 것은, 굉장히 Computationally Expensive 함을 의미한다. 그리고 직관적으로 Pixe-wise 유사도를 따지는 것이므로, image에 확대, 회전, 일부가 가려짐 등의 변화가 있을 경우 제대로 Classification을 해내지 못할 것이라 생각할 수 있다.  
  

### Cross-Validation
![Cross-Validation Folds](/assets/img/post/2021-2-10/cross_validation_folds.jpg)
Learning에 큰 영향을 주는 설정값인 Hyper Parameter를 잘 설정하기 위한 방법으로 Tarining Data Set 중 일부를 Validation set으로 놓고 Accuracy를 확인하여, 어떤 Hyper Parameter 값이 Learning에 적합한지 알아볼 수 있다. 그러나 Validation Set을 어떻게 잡는지에 의존적인 결과가 나오게 될 것이므로, 이런 의존성을 줄이기 위해 Cross-Validation을 사용할 수 있다.  
  
Cross-Validation은 Training Set을 몇 개의 fold로 나눈 뒤 각 fold들을 돌아가며 Validation Set으로 놓고 나머지는 Training Set으로 설정하여, Accuracy가 가장 좋은 Hyepr Parameter의 설정값들의 평균을 사용하는 방식이다. 일반적으로는 Training Set을 3, 5, 10개 정도의 fold로 나누어 실행한다고 한다. 그런데 딱봐도 연산을 많이 해야하는만큼 Computationally Expensive하다는 단점이 있다.
  
  
### Linear Classification
![Cross-Validation Folds](/assets/img/post/2021-2-10/liner_classifier.jpg)  
k-NN Algorithm의 경우 모든 image를 pixel-wise 비교할 Training Set의 모든 image들을 계속해서 기억하고 있어야 하고 또 연산을 해야 한다는 비효율성을 지니고 있다. 이러한 단점을 해결할 수 있는 model이 Linear Classification이다. 위의 그림과 같이 vector로 변환한 image에 Weight을 곱하여(bias vector를 더하는 것도 Weight에 포함시킬 수 있다.) Score가 가장 높은 Category로 image를 Classification하는 방식이다. 위에서는 단순화시키기 위해 Color Channel을 고려하지 않았지만, 실제로는 Red/Green/Blue라는 3개의 Channel마다의 Score의 weighted sum을 최종 Score로 계산하는 것으로 보인다.  
  
![Cross-Validation Folds](/assets/img/post/2021-2-10/liner_interpretation.jpg)  
이 때 W의 각 row가 image vector와 곱해져 해당 Category의 Score를 만듦을 알 수 있는데, 직관적으로 해당 row와 column의 inner product를 상대적으로 크게 만들기 위해서는 image vector의 값 분포가 row와 유사해야 할 것이라 생각할 수 있을 것이다. 그래서인지 흥미롭게도 W의 각 row를 시각화해보면 위와 같이 Category의 대략적인 형태를 담아냄을 볼 수 있고, 각 row를 Training Set image들의 prototype이라 생각할 수 있다 말한다. k-NN에서는 모든 image와 비교했다면, Linear Classification에서는 모든 image의 대략적인 feature를 뽑아낸 하나의 prototype과 비교하는 셈이다.
  
  
### Loss Function
Linear Classification에서 결과로 나온 Score값의 예측이 잘 맞는지, 아니라면 얼마나 잘 못 되었는지 점수를 매겨야 할 것이다. 그 역할을 하는 것이 Loss function이다. 그리고 이 Loss의 값을 줄이는 쪽으로 Learning이 될 것이다. Classifier의 Loss function을 정하는 방식 중에는 가장 많이 쓰이는 두 가지가 있다. 이걸 알아보는게 학습의 큰 목적 중 하나였고!  
  
##### Multicalss Support Vector Machine(SVM) Classifier - Hinge Loss/Max-Margin Loss
![Cross-Validation Folds](/assets/img/post/2021-2-10/SVM.jpg)  
SVM의 경우 어떤 fixed margin Δ에 대하여, 정답인 Category의 Score와 다른 Score 사이에 최소한 Δ만큼의 차이가 있도록 만드는 방식이라 볼 수 있다. 차이가 Δ보다 작으면 Loss가 0보다 크게 될 것이기 때문이다. Hyper Parameter인 Δ의 값은 Cross-Validation을 통해 정할 수 있겠지만, 경험적으로 1.0으로 설정하면 안정적이라고 한다.  
  
![Regularization](/assets/img/post/2021-2-10/regularization.jpg)  
그런데 단순히 위에서 언급한 바와 같이 Loss function을 정의하여 Loss를 0이 되게 만족시키는 W가 있다하면, λW (λ>1) 또한 Loss를 0으로 만들 것이다. 결과적으로 Score들을 포함하는 벡터가 λ된다 생각할 수 있기 때문이다. 즉 W가 유일하지 않게 되고, 단순히 W의 원소값들을 키우는 방향으로만 Learning이 될 수도 있다는 것이다. 그렇기에 Regularization Loss를 더해주며, 이것은 Hyepr Parameter인 'λ'와 'W의 각 원소들의 제곱의 합'의 곱으로 이루어진다.(그리고 이로 인해 W=0이 아니기에 Loss가 0이 될 수는 없게 된다.)  
  
##### Softmax Classifier - Cross-Entropy Loss
![Regularization](/assets/img/post/2021-2-10/soft_max.jpg)  
가장 많이 쓰이는 Classifier 두 가지 중 SVM을 제외한 다른 하나는 Softmax Classifier이다. 이 Calssifier는 보다 직관적이고 확률적인 해석이 가능한 결과값을 나타내는 Cross-Entropy Loss를 사용한다. 위와 같이 정의가 되며, fj는 Score를 담고 있는 결과 Vecotr의 j번째 값이다. 하나의 image xi에 대하여 '(한 Category Score의 Exponential)/(Category별 Score Exponential의 합)'을 모두 더하면 1이 될 것이고, 이러한 점에서 위 사진의 정답인 Category yi에 대한 값이 확률적으로 해석될 수 있다는 것으로 보인다.  
  
![Information Theorical View](/assets/img/post/2021-2-10/KL.jpg)  
Information Theory에서 Cross-Entropy는 위와 같이 정의된다고 한다. 이것은 p라는 Distribution과 q라는 Distribution이 일치할수록 값이 작아지는 [KL-Dviergence](https://en.wikipedia.org/wiki/Kullback%E2%80%93Leibler_divergence)가 p가 'True Distribution'인 경우에 정의된 것으로 이해할 수 있다. Classify 작업의 경우 정답인 Category에 대해서 값이 1이고, 나머지 오답인 Category에 대해서는 값이 0이므로 위와 같이 정의된 Loss를 줄이는 것이 Learning Output의 Distribution을 정답 Distribution에 근사시키는 과정이라 이해할 수 있을 것이다.  
  
![Numeric Stability](/assets/img/post/2021-2-10/numeric.jpg)  
이 때 Exponential로 인해 수가 너무 커지기에 계산이 불안정해지므로, 임의의 상수 C를 더해 계산을 안정적으로 바꿀 수 있다고 말한다. 그리고 통상적으로 logC=−maxjfj가 되도록 값을 설정한다고 한다.  
  
  
### +Norm이란? L1 & L2
![p-norm](/assets/img/post/2021-2-10/p-norm.jpg)  
Norm은 Vector space에서 nonnegative real number로 가는 함수라고 한다. 그 중 p-norm은 위와 같은 형태로 생겼고, 이를 통해 원점으로 부터 특정 vector의 distance를 나타낼 수 있다고 한다. 이 중 L1 norm(1-norm)과 L2 norm(2-norm)를 Loss function의 정의에 많이 사용하는 것으로 보인다. 특성은 L1이 L2보다 robust하고, L2가 L1보다 stable하다고 한다. 위에 나왔던 내용 중 Regualarization Loss가 L2 꼴임을 알 수 있고, L2 norm의 특성 상 값을 difuse하게 만들어 overfitting을 방지할 수 있다고 한다. 
  
  
  
*개인적인 질문 및 조언과 오류에 대한 지적은 늘 감사합니다!