---
title: Seq2Seq 기반 딥러닝 모델을 사용하여 자동으로 데이터 시각화 Plot 그리기
categories: [Side Project, Capstone]
tags: 
 - Capstone
 - Visualization
 - Recommendation
---

이화여대 2021-1학기 캡스톤디자인프로젝트B 스타트7팀 Ewha Visualization Recommendation Program(ERP) 기술 튜토리얼에 관한 글입니다.
본 포스팅은 시각화 추천 프로그램(Visualization Recommendation Program)을 개발하는 과정 중, 
Seq2Seq 모델을 기반으로 한 **Data2Vis**를 사용하여 **`데이터에 적합한 시각화 Plot을 자동으로 그려주는 딥러닝 모델을 학습`** 시키는 과정에 대하여 작성되었습니다.

---
> Contents 

1. About Our Project
2. Data2Vis
- Model Archi.
3. Our Model
- Dataset
- Model config & Training
- Tensorboard - Loss
4. Web Demo
5. Conclusion

---

# About Our Project...(and goal)

안녕하세요. 논문 리뷰 글이 아닌 프로젝트 관련 글은 오랜만인 것 같습니다.

요즘 캐글, 데이콘 등 데이터 분석 관련 Competition들이 굉장히 많다는 걸 관심있으신 분들은 아실 겁니다.
저도 데이콘 대회에 수상한 경험이 있고, 지금은 예전만큼은 못하지만 한창 Data Science 공부를 많이 할 때가 있었습니다.


처음 데이터 분석을 공부하시는 분들부터 전문적으로 다루시는 분들까지, 데이터 분석 분야에서 지나칠 수 없는 부분은 단연코 `Visualization about DATA` 일 것입니다. 캐글에서 대회 하나가 열리자마자 한시간 내로 아름다운 EDA plot들이 고수님들의 손을 거쳐 그려지는 것을 보실 수 있습니다. (Kaggle에서는 Notebook 분야의 그랜드 마스터분들이 주로 이 역할을 맡죠.)


또한 이러한 데이터 사이언스 대회가 아니더라도,  비즈니스 분석에 있어서 현재 자사/개인이 갖고 있는 데이터가 어떤 형태이고 어떤 의미인지를 파악하는 것은 매우 중요한 일입니다.


이러한 상황(=데이터에 대한 이해가 필요한 상황)에 발맞추어 Data Visualization에 대한 연구 또한 발전되고 있습니다. 


저희 팀은 이 `Data Visualization` 연구에서도 자동으로 plot(chart)을 그려줄 수 있는 **(딥러닝을 이용한) Data Visualization Recommendation**을 프로젝트 주제로 하였습니다. 자세히는 저희가 제공하는 샘플 데이터 또는 사용자가 원하는 데이터셋을 upload 했을 때, 저희의 Visualization Recommendation 모델이 데이터셋을 **해석**하여 여러 개의 plot을 그려주어 사용자에게 추천해주는 것입니다.

<br>

**즉 요약하자면,**
- 데이터셋(e.g., csv, tsv, json)을 `input`으로 하여
- 저희의 딥러닝 모델이 해당 데이터셋에 대한 적절한 `chart(visualization) recommendation`을 k개 리스트업하여 보여주고
- 사용자가 chart를 선택
    - 추가적으로는 차트를 편집하고 저장할 수 있는 기능

의 기능을 구현한 웹 어플리케이션을 구현하는 것이 목표입니다.


본 포스팅에서는 '사용자가 chart를 선택하기 전'까지의 과정, 즉 데이터셋을 받아 딥러닝 모델이 chart recommendation을 몇 가지 보여주는 단계까지 작성되었습니다.

---

# Data2Vis

앞서 Data Visualization에 대한 연구가 활성화되고 있다 하였는데, 이에 따라 당연히 Visualizaiton Recommendation 연구도 상당히 발전하였습니다. 관련하여 여러 개의 논문을 읽어보았고 github 코드 등을 다 살펴본 후 내린 결과, 저희는 [Data2Vis : Automatic Generation of Data Visualizations Using Sequence-to-Sequence Recurrent Neural Networks](https://arxiv.org/abs/1804.03126) 논문에 제시된 모델을 기본적으로 사용하기로 했습니다. **(물론 저희가 사용할 데이터셋에 더 optimize할 예정이므로 모델에 대한 여러 가지 실험들을 방학동안 거칠 예정입니다.)**



`Data2Vis`는 visualization generation 문제를 하나의 language translation problem으로 보았고, 이 문제를 **attention 베이스의 LSTM encoder-decoder 모델**을 사용하여 해결하였습니다.

또한 그래프를 그려내는 grammar로 JSON 포맷의 `Vega-Lite`를 사용합니다.
- 따라서, 학습용 데이터셋도 Vega-Lite 코드!
- 데이터셋에 대한 이야기는 아래 섹션에서 더 자세하게 진행하겠습니다.


Data2Vis 모델에 대해 간략하게 정리해보고 해당 섹션은 마무리합니다. ([이전 게시글](https://yskim0.github.io/paper%20review/2021/05/06/Data2Vis/)에 더 자세히 리뷰되어 있습니다. 아래 부분은 해당 글을 발췌하였습니다.)

## Model Architecture

![](https://user-images.githubusercontent.com/48315997/117335052-adad1280-aed5-11eb-9188-0b30c1cb9533.png)


- the data visualization problem as a **Seq2Seq translation problem**
```
input : dataset (fields, values in JSON format)
output : valid Vega-Lite visualization specification
```
- **encoder-decoder archi.**
> where the encoder reads and encodes a source sequence into a fixed length vector, and a decoder outputs a translation based on this vector.

- **Attention** Mechanism
> Attention mechanisms allow a model to focus on aspects of an input sequence while generating output tokens.

- **Beam Search algorithm**
> The beam search algorithm used in sequence-to-sequence neural translation models keeps track of k most probable output tokens at each step of decoding, where k is known as the beamwidth. This enables the generation of k most likely output sequences for a given input sequence.

- THREE techniques : **bidirectional encoding, differential weighing of context via an attention mechanism, and beam search**

- **character**-based sequence model


---

# Our Model (Dataset, Training-Settings...)


```
📩 input : 1개의 dataset
📬 output : visualization recommendation plot
```

본격적으로 저희 프로젝트에 사용할 모델을 만들겠습니다. 사실 data2vis 깃허브에서 제공하는 pretrained model이 있지만, 어차피 방학에 여러 실험을 할거니까 미리 연습해본다는 생각으로 **처음부터 새로 학습시켰습니다.**

## Dataset

Vega-Lite 그래머 기반의 양질의 plot 데이터셋을 크롤링하는 등 따로 수집해오기 어렵다고 판단한 결과, **data2vis가 학습할 때 사용했던 데이터셋을 그대로** 사용하였습니다. (본 포스팅에서 사용된 데이터셋은 data2vis github에 있습니다.)

### 원본 데이터 예시

- `barley.json`

![data](https://user-images.githubusercontent.com/48315997/118830315-cff15800-b8f9-11eb-8b11-c386ae61f3d1.png)

plot을 그리고자 하는 원본 데이터가 위와 같은 형태를 가진 json 타입의 데이터라 가정하겠습니다.


![image](https://user-images.githubusercontent.com/48315997/118835697-32e4ee00-b8fe-11eb-86db-be0fc6ff7fb3.png)


이는 모델을 학습시킬 때 사용되는 **training data** 중 하나로, 위의 barely.json 데이터에 대한 plot을 그리는 Vega-lite 그래머 기반의 데이터입니다.

즉, Vega-lite 그래머 기반의 plot을 생성하기 위하여 이러한 형식의 데이터셋이 학습용으로 필요한 것입니다.


![plot](https://user-images.githubusercontent.com/48315997/118832412-928dca00-b8fb-11eb-9832-73bbedccd0e9.png)


barley.json 데이터의 일부와 위의 Vega-lite 그래머 기반 코드를 [Vega Editor](http://vega.github.io/editor/)를 통해 그린 plot입니다.

`"mark" : "bar"`로 코딩했듯이 bar 차트이고, `encoding` 파트를 봤을 때 x축은 `yield`, y축은 yield 범위에 따른 `count` aggregation function이 적용되어 그리고자 한 차트가 잘 그려졌음을 확인할 수 있습니다.


**따라서,** 저희가 학습시키고자 하는 모델은 
- `barley.json`과 같은 데이터가 **input**으로 들어왔을 때
- 모델 안에서 Vega-lite 그래머 기반의 **적절한** plot을 그릴 수 있는 코드를 생성해주는 language translation 문제를 해결
하는 것입니다.
    - 이를 위해서 실제 데이터의 필드를 str, num으로 바꾸어 모델에 넣은 후 Vega-lite Spec를 아웃풋으로 받습니다.
- 출력으로 나온 Vega-lite spec에 원본 데이터 필드를 다시 mapping 시켜 최종 Vega-lite spec을 만든 후 그대로 웹에 올려주면 됩니다.




### Training data

![image](https://user-images.githubusercontent.com/48315997/118835445-f87b5100-b8fd-11eb-8b53-61dc15ec90b2.png)

11개의 데이터셋에 대한 **4300여개**의 Vega-Lite code를 학습용 데이터셋으로 사용합니다.
참고로 데이터셋 분포는 `Training : Eval : Test = 0.8 : 0.1 : 0.1` 입니다.





## Model config & Training

### Config

<img height = 500 src="https://user-images.githubusercontent.com/48315997/119250775-6022ec00-bbdd-11eb-9c0a-d066f2c0d55d.png">

학습시키는 모델의 config입니다. 
기존대로 모델은 **`AttentionSeq2Seq`**을 사용하였으나 다른 부분들을 몇 가지 수정해보았습니다.
- LSTM cell -> **GRU cell** 로 변경
- Dropout 수치를 0.5 -> **0.8로 변경**
- epoch 수를 20000 -> **15000으로 변경** (3000마다 save)


### Training

이제 터미널에 가서 가상환경 잘 설정해두고 **아래와 같은 명령어**를 친 후 학습이 완료될 때까지 기다리면 됩니다.

```bash
python3 -m bin.train --config_paths=\
"example_configs/nmt_medium-Copy1.yml,example_configs/train_seq2seq.yml,example_configs/text_metrics_bpe.yml" 
```

[결과]

![스크린샷 2021-05-23 오후 3 51 10](https://user-images.githubusercontent.com/48315997/119250967-b3496e80-bbde-11eb-9f64-44e6c33de0a6.png)

대략 **2일정도** 소요되었습니다.



## Tensorboard - Loss

**tensorboard**를 활용해 train 과정이 어땠는지 모니터링 할 수 있습니다.
다들 잘 아실 것 같습니다만, summary 데이터 파일이 있는 디렉토리로 이동하여 아래와 같은 명령어를 칩니다.

```
tensorboard --logdir .
```

![스크린샷 2021-05-23 오후 3 57 44](https://user-images.githubusercontent.com/48315997/119251097-9d887900-bbdf-11eb-9952-f2fc88d27ccc.png)

**학습이 마무리될 즈음에는 loss가 0.03 정도의 값을 가지고 있었습니다.**

아쉽게도 accuracy같은 정량적인 metric을 사용하기는 어렵기 때문에 직접 결과를 보고 평가해보겠습니다.

---
# Web Demo


<img width="800" alt="스크린샷 2021-05-23 오후 4 04 28" src="https://user-images.githubusercontent.com/48315997/119251228-8f872800-bbe0-11eb-8211-e7cc84d25eb0.png">

웹 데모의 경우 data2vis 깃허브의 `webserver.py` 를 약간 변형하였고, 직접 학습시킨 모델(`model.ckpt-15000`)을 연동시켰습니다. 


**모델에 random한 test data를 불러와서 Inference한 후 Vega-lite spec을 웹에 그린 결과입니다.**

***그럴듯한 plot***이 꽤 나오는 것을 보실 수 있습니다! 


---

# Conclusion

현재 저희는 프로젝트에 맞는 웹 페이지를 구현하는 중이고, 모델 개선과 관련하여 팀 내에서 논의하고 있습니다.

모델 쪽은 저의 일이기 때문에, 제가 생각해둔 바로는
- plot의 **종류를 다양화**한다.
- plot의 **테마**를 다르게 만들 수 있도록 한다.
- 더 많은 **aggregation** 기능을 추가한다.
- (본질적으로는 Vega-lite 말고 ~~**plotly**~~로 가고 싶다는 생각이 있음...) : 다양화시키기 수월해보여서
등이 있습니다.


