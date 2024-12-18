---
title: Attention Is All You Need
author: Yonsoo Kim
categories:
 - Paper Review

tags: 
 - Deep Learning
 - NLP
 - Transformer
---

Attention Is All You Need 를 읽고 정리한 글입니다.

---

# Attention Is All You Need

## Overview
(You should include contents of summary and introduction.)

**We propose a new simple network architecture, the Transformer, based solely on attention mechanisms, dispensing with recurrence and convolutions entirely.**

- recurrent, convolution 을 사용하지 않고 `Attention`을 이용해서 더 빠른 성능에 도달



## Related work (Basic concepts)

- layer normalization
- RNN
- Attention



## Methods
(Explain one of the methods that the thesis used.)

### Architecture

<img width="480" alt="스크린샷 2020-08-10 오후 12 03 51" src="https://user-images.githubusercontent.com/48315997/89760462-69980780-db27-11ea-81a1-77e7881cf135.png">

- the encoder maps an input sequence of symbol representations (x1,...,xn) to a sequence of continuous representations z = (z1,...,zn)

- **Encoder**
    - 6개의 identicla layer로 구성되어 있음.
        - each layer has 2 sub-layers.
            - *The first is a multi-head self-attention mechanism,* and *the second is a simple, position- wise fully connected feed-forward network.*
    - Add : residual connection을 의미함. 
    - Layer Norm. : `LayerNorm(x + Sublayer(x))`
    - produce outputs of dimension d_model = `512`

- **Decoder**
    - Encoder와 동일하게 6개의 동일한 레이어로 구성됨.
    - Masked Multi-Head Attention
    > We also modify the self-attention sub-layer in the decoder stack to prevent positions from attending to subsequent positions. **This masking, combined with fact that the output embeddings are offset by one position, ensures that the predictions for position i can depend only on the known outputs at positions less than i.**

### Attention 

<img width="680" alt="스크린샷 2020-08-10 오후 12 11 26" src="https://user-images.githubusercontent.com/48315997/89760471-6dc42500-db27-11ea-98ad-348e14e7cd30.png">

- **Scaled Dot-Product Attention**
    - Input : Q(Queries), K(Keys), V(Values)
    - **compute the dot products query with all keys**, divide each by root(dk), and apply a softmax function to obtain the weights on the values.
    <img width="332" alt="스크린샷 2020-08-10 오후 12 13 01" src="https://user-images.githubusercontent.com/48315997/89760492-7b79aa80-db27-11ea-8bb9-0b99805583e1.png">

    > We implement this inside of scaled dot-product attention by masking out (setting to −∞) all values in the input of the softmax which correspond to illegal connections.

- **Multi-Head Attention**
    > linearly project the queries, keys and values h times with different, learned linear projections to dk, dk and dv dimensions, respectively. 
    - perform the attention function in parallel yielding dv -dimensional output values.
    - 위의 과정이 완료된 후에는 `concatenate` 시킴
    <img width="487" alt="스크린샷 2020-08-10 오후 12 14 53" src="https://user-images.githubusercontent.com/48315997/89760505-83d1e580-db27-11ea-9402-fb704b743ab5.png">



### Position-wise Feed-Forward Networks

<img width="303" alt="스크린샷 2020-08-10 오후 12 15 47" src="https://user-images.githubusercontent.com/48315997/89760526-8c2a2080-db27-11ea-971a-627a7fe808a4.png">

### Positional Encoding

Transformer에서는 `시간의 연속성`을 모델의 핵심부에서 다루지 않음. -> 그러나 시간의 순서는 실제 언어에서 중요하므로 단어의 위치 정보를 포함시키기 위해 Positional Encoding을 사용

<img width="342" alt="스크린샷 2020-08-10 오후 12 19 13" src="https://user-images.githubusercontent.com/48315997/89760545-951af200-db27-11ea-8b7d-07338663b53d.png">

<img width="695" alt="스크린샷 2020-08-10 오후 12 19 18" src="https://user-images.githubusercontent.com/48315997/89760549-96e4b580-db27-11ea-817b-da7c7a09f7d9.png">


### Why Self-Attention

1. Total computational complexity per layer

2. The amount of computation that can be **parallelized, as measured by the minimum number of sequential operations required.**

3. The path length between long-range dependencies in the network.


A self-attention layer connects all positions with **a constant number** of sequentially executed operations, whereas a recurrent layer requires O(n) sequential operations.

### Training 

논문 참조.


## Additional studies
(If you have some parts that cannot understand, you have to do additional studies for them. It’s optional.)


[1] Jimmy Lei Ba, Jamie Ryan Kiros, and Geoffrey E Hinton. Layer normalization. arXiv preprint arXiv:1607.06450, 2016.

[2] BERT



## References
(References for your additional studies)

[https://www.youtube.com/watch?v=EyXehqvkfF0](https://www.youtube.com/watch?v=EyXehqvkfF0)

[https://www.youtube.com/watch?v=mxGCEWOxfe8](https://www.youtube.com/watch?v=mxGCEWOxfe8)
