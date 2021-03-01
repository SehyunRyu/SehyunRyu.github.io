---
title: 코드 실행 실패로 깨달은 가상환경의 중요성 +Numpy, Pytorch 등 학습 유튜브 영상
categories: [Privacy_Preserving_Deep-Learning]
tags:
seo:
date_modified: 2021-02-25 00:22:00 +0900
---

이전에 계획했던 FCN-Segmentation의 구현된 코드를 우선 동작시킨 뒤, 코드가 어떻게 구성되어 돌아가는지 이해를 하고자 했다. 그런데 코드를 제대로 읽어보기도 전에 구동시키는 것 자체로 많은 오류들을 마주치며, 개발환경의 구성이 정말 중요함을 느끼게 된 계기가 되었다.  
<br/>

### PIP 환경의 구축
[코드 Github 링크](https://github.com/pochih/FCN-pytorch)  
원래는 특별히 노트북에 무언가를 설치할 필요없이 사용할 수 있는 Google의 [Colaboratory](https://colab.research.google.com/notebooks/intro.ipynb)를 사용하려 했다. 그러나 [Papers With Code](https://paperswithcode.com/)에서 찾은 FCN-Segmentation의 논문 코드를 Github에서 다운받으니 단순 Python파일과 같이 확장자가 .py였다. 그런데 Colaboratory에서는 [Jupyter Notebook](https://jupyter.org/)과 동일한 확장자의 .ipynb 파일만을 사용할 수 있었다. 그래서 우선 노트북에서 실행이 되는지 확인을 하고, 되면 Colab에서 .py 파일을 .ipynb 파일로 변경하여 실행을 시키고자 했다.  
<br/>

![Python on Microsoft Store](/assets/img/post/2021-2-25/python_store.jpg)  
Python이 설치되어있지 않을 경우, cmd에 python 이라고만 치면 Microsoft sotre가 뜨며 Python을 다운로드 받을 수 있다. 그 뒤 아래와 같은 명령어를 cmd에 입력하여 PIP를 다운로드 받을 수 있을 것이다.  
```
python get-pip.py
```  
<br/>

![Fig 3.](/assets/img/post/2021-2-25/pip_list.jpg) 
PIP는 Python으로 작성된 패키지 소프트웨어를 설치하거나 관리하는 시스템이다. 간단하게 Python으로 개발을 할 때 필요한 다른 라이브러리 등을 쉽게 설치 및 업데이트를 하여 관리할 수 있도록 해주는 프로그램이라 볼 수 있을 것이다. 개인 컴퓨터에서 파이썬 프로그램을 구동시키기 위해서는 PIP가 편리할 것이기에 이를 사용했다. PIP를 이용하면 아래와 같은 명령어로 numpy, Pytorch, Tensorflow, Pandas 등 많은 library들을 설치할 수 있다.  
```
pip install (library 이름)
```
![If yoy success download](/assets/img/post/2021-2-25/pytorch_complete.jpg) 
설치가 완료되어 잘 실행되는지는 위와 같이 import가 되는지 확인해보면 된다.  
<br/>

![Too Long Directory](/assets/img/post/2021-2-25/Errno2.jpg) 
그런데 나의 경우는 다운로드 중 위와 같은 에러가 떴었다. 검색해서 찾은 [Stackoverflow의 글](https://stackoverflow.com/questions/54778630/could-not-install-packages-due-to-an-environmenterror-errno-2-no-such-file-or)을 보면 이것은 Directory가 너무 길어서 그렇다. 혹시 이런 오류가 뜬다면 링크의 내용과 긴 Directory를 사용할 수 있도록 설정하여 해결하면 될 것이다. 그리고 환경변수를 바꿀 때에는 '윈도우 버튼'+r 을 누르고 'sysdm.cpl ,3'를 입력하면 된다.  
<br/>

### Python 및 library 버전으로 인한 오류, 가상환경의 중요성
![Fixed Attribution Error](/assets/img/post/2021-2-25/pilmode.jpg)  
그렇게 해서 다운로드 받은 코드 파일을 실행시키고자 했는데, imageio library에서 Attribution Error가 발생했다. 검색해보니 library가 업데이트 됨에 따라 해당 함수를 imread로 변경하고 내부 argument도 pilmode로 변경해야 했다. 그렇게 바꾸고 나니 다운로드 받았던 CityScapes Dataset을 코드에 넣고 돌릴 수 있도록 npy파일로 Parsing하는 과정이 시작됐다.  
<br/>

![Image Parsing](/assets/img/post/2021-2-25/image_parsing.jpg)  
그런데 이 작업은 굉장히 오래걸렸다. 2월 20일 오후 4시 쯤 실행을 시작해서 2월 21일 오후 8시 쯤이 되어서야 끝났다. Training을 하면 이보다 시간이 훨씬 더 많이 걸릴 텐데, 노트북으로 실행시키는 것은 무리일 거란 생각이들어 어떻게든 Colab에서 실행을 해야겠다 생각했다. 그리고 그 이전에 우선 Training이 정상적으로 실행되는지 확인을 하고자 했다. 그런데 위와 같이 Attribution Error가 발생했고, 진호가 같이 봐줬는데 이렇게 해봐야 다른 곳에서 다시 발생하고 해결되면 그저 운이 좋은 것일 뿐이라 알려줬다. 코드를 다운받은 Github 페이지를 보니 Pytorch 0.2.0 version을 기준으로 작성이 된 코드였고, 윈도우에서는 해당 버전을 사용할 수 없었다. 그리고 Colab은 Python의 version을 바꿀 수 없음을 알게 되었다.  
<br/>

그래서 왜 Anaconda와 같은 가상환경을 사용해야 하는지 깨닫게 되었다. Virtual Machine과 Anaconda, Docker 가 정확히 어떤 차이들이 있는지 등을 구분은 아직 할 수 없다. 그렇지만 Anaconda나 Docker 등을 통해 코드 별로 실행에 필요한 환경을 따로 구성해줘야지, PIP는 컴퓨터 전체를 하나의 환경으로 사용한다 볼 수 있으므로 일례로 Ptroch 0.2.0 version이 필요한 코드와 1.7.1 version이 필요한 코드를 모두 돌릴 수 없을 것이다. 그래서 개발환경을 나누어(?) 조성하는 것이 필수적임을 깨닫고, 연구실에서는 서버에서 Docker를 사용한다는 것을 알게 되었다. 연구실 선배께서 pytty와 tigervnc를 통해 원격접속 모니터도 사용할 수 있게 해주셨는데, 앞으로 Docker에 대해서도 사용법 등을 알아봐야 할 것이다.  
<br/>

교수님과 상담을 통해 우선 [Tiny Face Recognition](https://openaccess.thecvf.com/content_cvpr_2017/papers/Hu_Finding_Tiny_Faces_CVPR_2017_paper.pdf) 논문을 읽고 코드를 구현해보며 학습하게 되었다. 본격적으로 구현하기 전에 이전에 막막하게만 느껴졌던 Numpy나 Pytorch 등에 관해 정보를 조금 찾다가 유튜브 영상을 봤는데, [이수안 컴퓨터연구소](https://www.youtube.com/channel/UCFfALXX0DOx7zv6VeR5U_Bg)라는 채널에서 상세하고도 간결하게 알려주셔서 기본적인 느낌을 갖는데 큰 도움이 된 것 같다. 결국 Numpy는 Vector, Matrix, Tensor 등의 Array들을 만들고, 형태를 바꾸고, 여러 연산을 하기 쉽게 해주는 library였다. 그리고 Pytroch는 Numpy의 기본적인 기능들에 더해 Neural Network를 만들고 Learning을 하기 위해 Gradient를 구하는 등의 Powerful한 기능들을 추가적으로 제공하는 library라 이해할 수 있을 것 같다.  
<br/>

앞으로는 크게 우선 Tiny Face Recognition 논문 구현을 해보고, 연구실 선배가 말씀해주신 [Kaggle](https://www.kaggle.com/)을 통해 코딩 연습을 좀 하고, 교수님께서 말씀해주신 Activity Recognition에 대해 공부하고 구현을 해봐야하지 않을까 싶다. 학기중이라 쉽지 않겠지만, 화이팅!  
<br/> 
<br/>   
<br/>  
*개인적인 질문 및 조언과 오류에 대한 지적은 늘 감사합니다!