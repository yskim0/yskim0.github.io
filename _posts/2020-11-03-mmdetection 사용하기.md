---
title: MMDetection 사용하기
categories:
 - Vision
tags: 
 - Deep Learning
 - Computer Vision
 - MMdetection
 - Mask R-CNN
---


Dacon의 K-Fashion AI 경진대회의 baseline 설명에 따라 **MMdetection** toolkit을 설치해보고 학습까지 진행해보겠습니다.

---


저는 오늘부터 시작된 [K-Fashion AI 경진대회](https://dacon.io/competitions/official/235672/overview/)에 참가합니다.


이 대회는 제가 관심있는 분야인 ***컴퓨터 비전*** 대회라 관심을 가지게 되었고, 실제로 CV 관련 대회는 처음이니 배운다는 마음가짐으로 열심히 공부해보려 합니다.

# MMdectection?

이 대회의 베이스라인을 참고하면서 `mmdetection` 이라는 것을 처음 알게 되었습니다. 
논문도 있으니 시간 남을 때 봐야겠네요

깃허브는 [여기](https://github.com/open-mmlab/mmdetection)에 들어가면 되고, 리드미 파일에 mmdetection이 어떤 건지 나와있습니다.
짧게 요약하자면, 

> **MMDetection** is an open source object detection toolbox based on PyTorch. 

이라 하네요.

object detection에서 다루는 다양한 모델들을 한 곳에 모아뒀다고 하니 정말 간편해보입니다. (이런 걸 이제 알게된 게 아쉬울 정도로...) 

[여기](https://github.com/open-mmlab/mmdetection#benchmark-and-model-zoo) 에 support 하는 모델들이 나열되어 있습니다.

또한 버전 업데이트 및 유지보수도 굉장히 잘 되고 있다고 합니다!

---


# Installation

데이콘 베이스 라인을 보고 따라하는 것이므로 버전도 맞춰서 해봅니다.
우선 mmdetection 깃허브 링크에 가서 브랜치를 `v2.3.0` 으로 이동합니다.

[바로 가기](https://github.com/open-mmlab/mmdetection/tree/v2.3.0)
![스크린샷 2020-11-03 오후 3 34 25](https://user-images.githubusercontent.com/48315997/97955665-0f3ada00-1dea-11eb-93d6-a233996a32f8.png)


README.md 를 조금 내리다 보면 `install.md` 로 넘어갈 수 있는 하이퍼링크가 있습니다.
그걸 눌러 install.md로 이동합니다.

![image](https://user-images.githubusercontent.com/48315997/97955738-3db8b500-1dea-11eb-9988-2a6afd19ec79.png)

![image](https://user-images.githubusercontent.com/48315997/97955982-dbac7f80-1dea-11eb-9aa4-e7ff1a77c779.png)

**Requirements** 잘 확인하시고, install mmdetection 순서대로 진행합니다.

### 1. 가상환경 생성

```
conda create -n 가상환경이름 python=3.7 -y
conda activate 가상환경이름 
```

이렇게 가상 환경 생성하고 activate 시킵니다.

### 2. pytorch, torchvision 설치 (버전 주의)

베이스라인 따라가려면 (즉, mmdetection version==2.3.0) torch version == `1.5.0` 이어야 합니다.
pytorch를 원하는 이전 버전으로 설치해야 할 때 공식홈페이지에 커맨드가 다 나와있습니다. [link](https://pytorch.org/get-started/previous-versions/)

```
conda install pytorch==1.5.0 torchvision==0.6.0 -c pytorch
```

### 3. MMdetection repo. clone

mmdetection 레포지터리를 clone 해야 합니다.
이 때도 이 도큐먼트와 다르게 주의해야 할 점이 있습니다. 앞서 말했듯이 저는 베이스라인 따라 branch == v2.3.0으로 이동했습니다. 이 브랜치에 해당되는 버전으로 git clone 해야 합니다.

```
git clone --branch v2.3.0 https://github.com/open-mmlab/mmdetection.git
```

### 4. Requirements 설치

우선 클론한 디렉토리로 갑니다. 
```
cd ./mmdetection
```

저와 같이 다른 사람들과 같이 쓰는 서버를 쓰시는 환경이라면 마음대로 패키지/라이버리를 다운 받았을 때 충돌될 가능성이 있습니다. 

**그래서 이 requirements 들을 가상 환경 내에 설치해야 합니다.**
현재 가상환경의 절대 주소를 모르시면, 터미널에

```
conda env list
```

를 치시면 가상환경 이름 뒤에 경로가 나옵니다.
그 path를 copy 해두세요.
가상환경의 경로를 {PATH}라고 가정하고 적어보겠습니다.
```
{PATH}/bin/pip install -r requirements/build.txt
```

이렇게 하면 requirements가 잘 설치될 것입니다.

### 5. Set up

```
pip install -v -e .  # or "python setup.py develop"
```

앞에 명령어 해보고 안되면 `python setup.py develop` 하시면 됩니다.

(저 같은 경우는 `pip install -v -e .` 했을 때 안됐어서 `python setup.py develop` 명령어 해서 정상 설치 됐습니다.)

---


# Training 하기 전 준비 해야 할 것들

### 1. Dataset 관련

`mmdetection/mmdet/datasets` 에서 데이터셋에 대한 기본 설정을 할 수 있습니다.
`custom.py` 로 직접 personal 한 설정을 할 수 있겠지만 이 대회는 `coco.py` 에서 설정된 것에서 크게 벗어나지 않아 이대로 사용해도 된다고 합니다.

그래서 `coco.py` 에서 미리 설정된 CLASSES 들을 이 대회에서 쓰일 클래스로 바꾸겠습니다.

![image](https://user-images.githubusercontent.com/48315997/97957150-c71db680-1ded-11eb-88e2-3b503374e352.png)


또한, mmdetection의 경우 train/test 폴더가 한 곳에 flatten 하게 존재해야 한다고 합니다.

`/train/a/*.jpg` 이런 형태가 아닌 `/train/a_*.jpg` 이런 식으로요.

이건 파이썬 코드로 쉽게 수정 가능하니 잊지 않고 하시길 바립니다.

### 2. training options

트레이닝할 때 필요한 기본 옵션들을 `mmdetection/config/_base_/default_runtime.py` 에서 가능합니다.

이 config는 우선 베이스라인 코드에서 제공한 것을 붙여 써서 지금 제 환경에 맞게 조금씩 고쳤습니다. (data root, gpu 개수, epoch 등)

자세한 건 [여기](https://github.com/dacon-ai/K-fashion-baseline/blob/master/configs/_base_/default_runtime.py)를 참고해주세요.

train/test set 경로 등등 잘 설정하고 나면 바로 학습 시작 가능합니다.

---

# Training !

```
python tools/train.py configs/_base_/default_runtime.py
```

이걸 치시면 config로 해놨던 설정들이 주루룩 뜨면서 학습이 시작됩니다.

![image](https://user-images.githubusercontent.com/48315997/97957533-bf124680-1dee-11eb-93b9-a3b3921adbe6.png)

---

# Troubleshooting

### AttributeError: 'COCO' object has no attribute 'get_cat_ids'

`pycocotools` 관련 오류입니다.

저같은 경우에는 

```
{PATH}/bin/pip install "git+https://github.com/open-mmlab/

cocoapi.git#subdirectory=pycocotools"
```

로 해결했습니다.

### ModuleNotFoundError: No module named 'mmcv._ext'

mmcv가 제대로 설치되지 않은 경우입니다.

```
{PATH}/bin pip uninstall mmcv
{PATH}/bin pip install mmcv-full==1.0.5
```

저는 이렇게 해결했습니다.

---

# Reference 

[데이콘 베이스라인](https://www.youtube.com/watch?v=UEu73ew7mSY&feature=youtu.be&ab_channel=%EB%8D%B0%EC%9D%B4%EC%BD%98)