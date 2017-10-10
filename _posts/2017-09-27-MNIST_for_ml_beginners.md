---
layout: post
title: MNIST For ML Beginners
author: 이호종
categories: [dev]
comments: true

---

> [MNIST For ML Beginners](https://www.tensorflow.org/get_started/mnist/beginners)의 내용을 옮겼습니다.

> 이번 포스팅에는 확률과 통계에 관한 많은 수학식이 나옵니다. 저도 전체를 아직 이해하지는 못했습니다. 읽어주시는 분들이 오류라던지 고쳐야 할 부분에 대해서 알려주시면 그에 맞춰서 수정하도록 하겠습니다. 읽어주셔서 감사합니다.

# MNIST For ML Beginners

이 튜토리얼을 머신러닝과 텐서플로 모두를 처음 접하는 사람들을 위한 것이다. 만약 MNIST가 무엇인지, 소프트맥스(multinomial logistic) 회귀가 무엇인지 알고 있다면 [다음장](https://www.tensorflow.org/get_started/mnist/pros)을 참고하는 편이 좋다. 이번 튜토리얼을 시작하기 전에 [텐서플로 설치](https://www.tensorflow.org/install/index)는 완료되어 있어야 한다.

프로그래밍을 배우기 시작하면 처음으로 하는 전통적인 것은 "Hello World!"를 출력하는 것이다. 프로그래밍의 Hello World와 같이 머신러닝에서는 MNIST가 있다.

MNIST는 간단한 컴퓨터 비전 데이터 셋이다. 아래 그림과 같은 손으로 쓴 숫자 이미지로 구성되어있다.

![MNIST dataset example](https://www.tensorflow.org/images/MNIST.png){:width="50%"}

각 이미지가 어떤 숫자인지 알려주는 레이블도 포함되어있다. 예를들면 위 이미지에 대한 레이블은 5, 0, 4, 1 이다.

튜토리얼에서 이미지를 보고 어떤 숫자인지 예측하는 모델을 학습한다. 목표는 최고 성능을 가지는 정교한 모델을 학습하는 것이 아니라 (나중에 코드를 주겠지만) 텐서플로를 사용하여 한 발 내딛게 하는 것이다. 소프트맥스 회기(Softmax Regression)이라 불리는 아주 간단한 모델로 시작하겠다.

이 튜토리얼의 실제 코드는 매우 짧다. 흥미있는 내용이 있는 전부는 단지 세줄이다. 그러나 그 뒤에 숨겨진 생각(텐서플로의 동작과 핵심 머신러닝 컨셉)을 이해하는 것은 아주 중요하다. 이것 때문에 코드를 통해서 유심히 살펴봐야한다.

## About this tutorial

이 튜토리얼은 [mnist_softmax.py](https://www.github.com/tensorflow/tensorflow/blob/r1.3/tensorflow/examples/tutorials/mnist/mnist_softmax.py) 코드에서 발생하는 것을 한줄, 한줄 설명한다.

이 튜토리얼을 다음을 포함하여 몇 가지 다른 방법으로 사용할 수 있다.

- 파이썬 환경에 코드를 한줄, 한줄 복사/붙여넣기 하여 각 줄의 설명을 통해 익히는 것
- 설명을 읽은 전, 후에 전체 `mnist_softmax.py` 파이썬 파일을 실행시켜라. 그리고 명확하지 않은 코드를 이해하기 위해 이 튜토리얼을 사용하는 것

이 튜토리얼에서 성취할 것:

- MNIST 데이터와 softmax 회귀에 대하여 학습한다.
- 이미지의 모든 픽셀을 확인하는 것을 기초로하여 숫자 인식에 대한 모델 함수를 생성한다.
- 수천의 예제를 "보는 것"에 의해 숫자를 인식하는 모델을 학습하기 위해 텐서플로를 사용한다.(그리고 첫번째 텐서플로 세션을 실행한다.)
- 테스트 데이터 셋에 대한 모델의 정확도를 확인한다.

## The MNIST Data

MNIST 데이터는 [Yann LeCun's website](http://yann.lecun.com/exdb/mnist/)에 올라와있다. 이 튜토리얼에서 코드 안에 복사/붙여넣기 하려면 아래 두 줄로 시작하면 된다. 자동으로 다운로드하고 읽을 것이다.

```python
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets("MNIST_data/", one_hot=True)
```

> 위 두 줄은 `mnist_softmax.py`에 이미 포함되어 있다. `mnist_softmax.py`를 실행시키면 자동으로 다운로드 받는다. 단, 한줄씩 넣어서 확인하는 경우 위 두 라인을 넣으면 다운로드 받는다.

MNIST 데이터는 세 부분으로 나뉘어 있다. 55,000 개의 학습 데이터 (`mnist.train`), 10,000 개의 테스트 데이터(`minst.test`), 5,000개의 검증 데이터(`mnist.validation`). 이 분류는 매우 중요하다. 학습한 내용이 실제로 일반화 되어있는지 학습하지 않은 데이터로 검증하도록 데이터를 분리하는 것은 머신러닝에서 필수적이다.

앞에 말한 것과 같이 MNIST 데이터는 두 부분으로 되어있다. 손으로 쓴 숫자 이미지와 그 레이블. 이미지를 "x", 레이블을 "y"라고 하겠다. 학습 셋과 테스트 셋 모두는 이미지와 그 레이블을 포함하고 있다. 예를 들면 학습 이미지는 `mnist.train.images` 학습 레이블은 `mnist.train.labels` 이다.

각 이미지는 28x28 픽셀이다. 이것을 숫자의 배열로 해석할 수 있다.
![숫자 이미지의 배열화 예제](https://www.tensorflow.org/images/MNIST-Matrix.png){:width="50%"}

이 배열을 28x28=784 숫자의 벡터로 펼칠 수 있다. 이미지끼리 일관된 방법이라면 배열을 펼치는 방법은 중요한 것이 아니다. 이런 관점에서 MNIST 이미지는 [매우 풍부한 구조](https://colah.github.io/posts/2014-10-Visualizing-MNIST/)(경고: 계산 집약적 시각화)와 함께 784차원 벡터 공간에 있는 점의 묶음일 뿐이다.

데이터를 펼치면 이미지의 2D 구조에 대한 정보가 사라진다. 이것이 안 좋은 것인가? 최고의 컴퓨터 비전 방법은 이 구조를 활용하는 것인다. 뒤의 튜토리얼에서 다룰 예정이다. 그러나 여기서 사용할 간단한 방법, softmax regression(아래에 정의)는 그렇지 않다.

결과는 `mnist.train.images`는 `[55000, 784]` 형태의 텐서(n차원 배열)이다. 처음 차원은 이미지 리스트의 인덱스이고 두번째 차원은 각 이미지의 픽셀 인덱스이다. 텐서 안의 각 요소는 개별 이미지안의 개별 픽셀에 대한 0과 1사이의 픽셀 채도 정보이다.

![mnist.train.xs](https://www.tensorflow.org/images/mnist-train-xs.png){:width="50%"}

MNIST 안의 각 이미지는 이미지에 그려진 숫자를 나타내는 0과 9 사이의 숫자 레이블이 있다.

이 튜토리얼의 목적을 위해 "ont-hot 벡터" 로서 레이블을 윈한다. one-hot 벡터는 대부분 차원에서 0이고, 한 차원에서 1인 벡터이다. 이 경우에 n번째 숫자는 n번째 차원이 1인 벡터로 표현된다. 예를 들면, 3은 $$[0,0,0,1,0,0,0,0,0,0]$$ 이다. 결과적으로, `mnist.train.labels`는 부동소수점 `[55000,10]`차원의 배열이다.

![mnist.train.ys](https://www.tensorflow.org/images/mnist-train-ys.png){:width="50%"}

이제 모델을 만들 준비가 되었다!

## Softmax Regressions

MNIST에 있는 모든 이미지는 손으로 쓰여진 0과 9 사이의 숫자라는 거승ㄹ 안다. 그래서 주어진 이미지는 단지 10개의 가능한 경우가 있다. 이미지를 모고 각 숫자에 대한 확률을 줄 수 있다. 예를 들면 우리 모델을 숫자 9의 사진을 보고 그것이 9임을 80%의 확률로 확신한다. 그러나 100% 확신이 아니기 때문에 8일 확률은 5%(위쪽 원때문에)이고 모든 다른 수에 대한 아주 작은 확률을 줄 수 있다.

이것은 softmax regression이 자연스럽고 단순한 모델인 전형적인 사례이다. 여러가지 중에 하나인 객체에 확률을 할당하면 softmax는 0과 1사이의 값의 리스트를 제공하기 때문에 1을 더한다. 심지어 나중에 더 복잡한 모델을 학습할때, 마지막 단계는 softmax의 계층일 것이다.

softmax regression은 두 단계를 가진다. 첫번째는 특정 클래스에서 입력이 있다는 증거를 더하고 그 다음에 증거를 확률로 변환한다.

주어진 이미지가 특정 클래스에 있다는 증거를 계산하기 위해 픽셀 채도 정보에 대해 가중치 합을 한다. 가중치는 높은 채도 정보를 가지는 픽셀이 이미지가 해당 클래스에 있다는 것에 반하는 증거라면 음수이고 증거가 알맞다면 양수이다.

아래 도형은 가중치를 각 클래스에 대해 학습한 모델에 나타낸 것이다. 빨간색은 음의 가중치, 파란색은 양의 가중치를 나타낸다.

![weight model](https://www.tensorflow.org/images/softmax-weights.png){:width="50%"}

또한 편향(bias)라 불리는 추가 정보를 더한다. 기본적으로 어떤 것은 더 입력값에 의존한다고 말할 수 있다. 입력값 $$x$$가 주어졌을때 클래스에 대한 증거 $$i$$는 다음과 같다.

$$
evidence_i = \sum_{j}{W_{i,j}x_{j} + b_{i}}
$$

$$W_i$$는 가중치이고 $$b_i$$는 클래스 $$i$$에 대한 편향(bias)이고 $$j$$는 입력 이미지 $$x$$의 픽셀을 합한것에 대한 인덱스이다. 그다음 "softmax" 함수를 사용하여 증거 집계표를 예측 확률 $$y$$로 변환한다.

$$
y = softmax(evidence)
$$

여기서 softmax는 "활성화" 또는 "연결" 함수로써 수행하여 선형함수의 출력을 원하는 형식으로 만든다. 이 경우 10가지 경우에 대한 확률분포이다. 증거 집계표를 입력값이 각 클래스에 속하는 확률로 변환하는 것을 생각해 볼 수 있다. 정의는 다음과 같다.

$$
softmax(x) = normalize(exp(x))
$$

이 수식을 확장하면 다음과 같다.

$$
softmax(x)_i = \frac{exp(x_i)}{\sum_{j}{exp(x_j)}}
$$

그러나 softmax를 처음 방법으로 생각하는 것이 종종 더 유용하다. 입력을 지수화 한 다음 정규화한다. 지수화는 하나 이상의 증거 단위가 모든 가설에 주어진 가중치를 곱셈 증가시키는 것을 의미한다. 반대로 하나 적은 증거 단위를 갖는 것은 가설이 초기 가중치의 일부를 얻는 것을 의미한다. 가설은 0이나 음의 가중치를 결코 갖지 않는다. 그다음 softmax는 이 가중치를 정규화하고 최대 1을 더하여 유효한 확률 분포를 구성한다.(softmax 함수에 대한 더 많은 설명을 보려면 Michael Nielsen의 책에서 [complete with an interactive visualization 부분](http://neuralnetworksanddeeplearning.com/chap3.html#softmax)을 확인하자)

softmax regression는 많은 $$x$$가 있더라고 아래와 같이 보일 수 있다. 각 출력에 대하여 $$x$$의 가중치 합을 계산하고 편향(bias)를 더한다음 softmax를 적용한다.

![softmax example](https://www.tensorflow.org/images/softmax-regression-scalargraph.png){:width="50%"}

수식으로 표현하면 다음과 같다.

![softmax equatioin](https://www.tensorflow.org/images/softmax-regression-scalarequation.png){:width="50%"}

이 절차를 행렬곱과 벡터를 더함으로 "벡터화" 할 수 있다. 이것은 계산 효율화에 도움이 된다.(생각할 수 있는 유용한 방법이다.)

![softmax matrix](https://www.tensorflow.org/images/softmax-regression-vectorequation.png){:width="50%"}

더 간단하게 표현하면 아래 식으로 쓸 수 있다.

$$
y = softmax(Wx+b)
$$

이제 텐서플로를 사용할 수 있는 형태로 변환하자.

## Implementing the Regression

파이썬에서 숫자 계산을 효율적으로 하기 위해 행렬곱과 같은 고비용 연산을 파이썬 외부에서 다른 언어로 작성된 높은 효율 코드를 사용한 [NumPy](http://www.numpy.org/)같은 라이브러리를 일반적으로 사용한다. 모든 연산을 파이썬으로 다시 전환하면 많은 부하가 발생할 수 있다. GPU나 분산환경에서 계산을 수행한다면 데이터를 전송하는데 높은 비용이 있어 특히 안 좋다.

텐서플로는 파이썬 밖에서도 힘든 작업을 수행한다. 그러나 부하를 피하기 위해 한걸음 더 나아간다. 텐서플로는 파이썬에 독립적으로 단일 고비용 연산을 수행하는 것 대신에 파이썬 외부에서 전체가 실행되는 상호작용 연산 그래프를 설명한다. (이런 접근은 몇가지 머신러닝 라이브러리에서 볼 수 있다.)

텐서플로를 사용하려면 처음에 다음을 추가해야 한다.

```python
import tensorflow as tf
```

심볼릭 변수를 조작하여 상호작용연산을 설명한다. 하나를 생성해보자.

```python
x = tf.placeholder(tf.float32, [None, 784])
```

`x`는 특정 값이 아니다. 텐서플로에 계산 실행을 요구할때 입력값을 주는 `placeholder`이다. 입력값으로 무수히 많은 MNIST 이미지를 입력할 수 있기를 원한다. 각 이미지는 784 차원의 벡터로 펼쳐져있다. 이것을 모양이 `[None, 784]`인 부동소수점 숫자의 2차원 텐서로 표현한다.(여기서 `None`는 아무 길이나 될 수 있는 1차원을 의미한다.)

또한 모델에 대한 가중치와 편향(bias)가 필요하다. 추가 입력값과 같이 가중치와 편향(bias)를 다루는 것을 상상할 수 있지만 텐서플로는 더 나은 방법(`Variable`)을 가지고 있다. `Variable`은 상호작용 연산의 텐서플로 그래프 안에 존재하는 수정 가능한 텐서이다. 계산에 따라 사용과 수정이 가능하다. 머신러닝 응용프로그램에서는 일반적으로 모델 인자가 `Variable`이다.

```python
W = tf.Variable(tf.zeros([784, 10]))
b = tf.Variable(tf.zeros([10]))
```

`tf.Variable`에 `Variable`의 초기값을 주는 방식으로 `Varible`을 생성한다. 이 경우 `W`와 `b`를 전체가 0인 텐서로 초기화한다. `W`와 `b`를 학습할 예정이기 때문에 초기값이 무엇인지는 중요하지 않다.

다른 클래스들에 대하여 증거(evidence)의 10차원 벡테를 생성하는 것에 의해 784차원 이미지 벡터를 곱하기때문에 `W`는 [784,10]의 형태를 가진다. `b`는 [10]의 형태를 가지고 이것을 출력값에 더할 수 있다.

이제 모델을 구현할 수 있다. 정의하는데 단지 한 줄이면 된다.

```python
y = tf.nn.softmax(tf.matmul(x, W) + b)
```

첫째, 식 `tf.matmul(x,W)`을 이용하여 `x`와 `W`를 곱한다. 이것은 방정식에서 $$Wx$$가 있는 방정식을 곱했을 때 뒤집힌다. `x`를 다양한 입력을 갖는 2차원 텐서로 다루는 것은 작은 트릭이다. `b`를 더하고 마지막으로 `tf.nn.softmax`를 적용한다.

단지 한 줄로 모델을 정의 하고, 설정을 위해 몇 줄을 작성한다. 텐서플로가 softmax regression을 꽤 쉽게 만들었기 때문이다. 머신러닝 모델에서부터 물리학 실험까지 다양한 종류의 수치계산을 설명하는 매우 유연한 방법 일뿐이다. 그리고 한번 정의되면 모델은 다른 기기에서도 실행할 수 있다. 컴퓨터의 CPU, GPU, 심지어 스마트폰에서도!

## Training

모델을 학습하기 위해서 모델이 좋아진다는 것에 대한 정의를 할 필요가 있다. 사실, 머신러닝에서는 일반적으로 모델이 나쁜 것이 무엇인지 정의한다. 이것을 비용 또는 손실이라 부른다. 그리고  모델이 원하는 결과로부터 얼마나 떨어져 있는지 표현한다. 에러를 최소화하도록 시도하고 에러가 작아질수록 모델이 좋아진다.

하나의 매우 일반적인, "cross-entropy"라 불리는 모델의 손실을 정하기 위한 매우 좋은 함수가 있다. cross-entropy는 정보 이론에서 정보 압축 코드에 대해 생각하는 것으로 발생한다. 그러나 도박에서부터 머신러닝에 이르기까지 많은 분야에서 중요한 아이디어가 되고 있다. 정의하면 다음과 같다.

$$
H_{y'}(y) = -\sum_{i}{y'_{i}\log(y_i)}
$$

$$y$$는 예측 확률 분포이고, $$y'$$은 실제 분포이다(숫자 레이블이 있는 one-hot 벡터). 대략적으로 cross-entropy는 사실을 설명하는데 있어서 예측이 얼마나 비효율적인지 측정한다. cross-entropy에 대하여 자세한 내용을 다루는 것은 이 튜토리얼의 범위를 넘어선다. 그러나 [이해](https://colah.github.io/posts/2015-09-Visual-Information) 할 가치가 있다.

cross-entropy를 구현하기 위해 우선 정답을 입력하기 위해 새 placeholer를 추가해야 한다.

```python
y_ = tf.placeholder(tf.float32, [None,10])
```

그런다음 cross-entropy 함수 $$-\sum{y'\log(y_i)}$$를 구현 할 수 있다.

```python
cross_entropy = tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(y), reduction_indices=[1]))
```

처음에 `tf.log`는 각 `y`의 요소에 대하여 로그를 계산한다. 다음 `tf.log(y)`의 연계된 요소에 `y_`의 각 요소를 곱한다. 그 다음, `reduction_indices[1]` 파라메터에 따라 `tf.reduce_sum`은 y의 두번째 차원에 있는 요소를 더한다. 마지막으로 `tf.reduce_mean`은 배치 안의 모든 예제에 대하여 평균을 계산한다.

소스 코드를 보면, 수치적으로 불안정하기 때문에 이 식을 사용하지 않았다. 대신에 정규화되지 않은 로짓에  `tf.nn.softmax_cross_entropy_with_logits`를 적용하였다(예를들면, `tf.matmul(x,W)+b`에서 `softmax_cross_entropy_with_logits`를 호출한다). 더 수치적으로 안정한 함수가 softmax 활성화를 내부적으로 계산한다. 코드를 작성할때, `tf.nn.softmax_cross_entropy_with_logits`를 사용하는 것을 고려하라.

모델이 하길 원하는 것을 알고 있으므로 텐서플로가 그렇게 하도록 훈련시키는 것은 매우 쉽다. 텐서플로는 계산의 전체 그래프를 알기 때문에 [역전파 알고리즘(backpropagation algorithm)](https://colah.github.io/posts/2015-08-Backprop)을 자동으로 사용할 수 있다. 역전파 알고리즘은 최소화할 손실에 변수가 어떻게 영향을 주는지 효율적으로 결정한다. 그런 다음, 변수를 수정하고 손실을 최소화하는 최적화 알고리즘을 선택하여 적용할 수 있다.

```python
train_step = tf.train.GradientDescentOptimizer(0.5).minimize(cross_entropy)
```

이 경우, 텐서플로에게 0.5의 학습속도를 가진 [경사하강법(gradient descent algorithm)](https://en.wikipedia.org/wiki/Gradient_descent)을 사용하여 `cross_entropy`를 최소화 하길 요구한다. 경사하강법은 텐서플로가 단순히 각 변수를 비용을 줄이는 방향으로 조금 움직이는 간단한 절차이다. 그러나 텐서플로는 [많은 다른 최적화 알고리즘](https://www.tensorflow.org/api_guides/python/train#Optimizers)을 제공한다. 하나를 사용하는 것은 한줄을 수정하는 것처럼 간단하다.

여기서 배후에 있는 텐서플로가 실제로 하는 일은 역전파와 경사하강법을 구현하는 새 연산을 그래프에 추가하는 것이다. 그런 다음 실행하면 하나의 작업을 되돌려 주어 경사하강 학습 단계를 수행하고 변수를 약간 조정하여 손실을 줄인다.

이제 `InteractiveSession`에 모델을 실행할 수 있다.

```python
sess = tf.InteractiveSession()
```

처음으로 생성한 변수를 초기화하는 작업을 생성한다.

```python
tf.global_variables_initializer().run()
```

학습을 시작하자. 1000번 학습단계를 수행할 예정이다.

```python
for _ in range(1000):
  batch_xs, batch_ys = mnist.train.next_batch(100)
  sess.run(train_step, feed_dict={x: batch_xs, y_: batch_ys})
```

각 단계에서 학습 셋에서 100개의 랜덤 데이터의 "배치(batch)"를 가져온다. `placeholder`를 배치에 있는 데이터로 바꾸어 `train_step`을 수행한다.

랜덤 데이터의 작은 배치를 사용하는 것은 확률론적 학습으로 불린다. 이 경우 확률론적 경사 하강법이다. 이론상으로 무엇을 해야 하는지 더 잘 알려주기 때문에 모든 학습단계에서 모든 데이터를 사용하고 싶지만, 그것은 매우 높은 비용이 드는 일이다. 그래서 대신에 매번 다른 부분집합을 사용한다. 이렇게 하는 것이 비용이 싸고 많은 이점이 있다.

## Evaluating Our Model

우리 모델이 얼마나 잘 동작하는가?

우선 올바른 레이블을 어디에서 예측했는지 알아보자. `tf.argmax`는 특정 축을 따라 텐서에 가장 높은 엔트리의 위치를 제공하는 꽤 유용한 함수이다. 예를 들면, `tf.argmax(y,1)`은 우리 모델이 각 입력에 대하여 가장 가능성이 있다고 생각하는 레이블이다. 반면에 `tf.argmax(y_,1)`은 정답 레이블이다. 예측이 정답과 같은지 확인하기 위하여 `tf.equal`을 사용할 수 있다.

```python
correct_prediction = tf.equal(tf.argmax(y,1), tf.argmax(y_,1))
```

위 함수는 불리안 리스트를 반환한다. 어느 부분이 맞는지 확인하기 위해 부동소수점 숫자로 변환한 후 평균을 계산한다. 예를들면 `[True, False, True, True]`는 `[1,0,1,1]`이 되고 평균은 `0.75`가 된다.

```python
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
```

마지막으로 테스트 데이터에 대하여 정확도을 구하기 위해 다음을 사용한다.

```python
print(sess.run(accuracy, feed_dict={x: mnist.test.images, y_: mnist.test.labels}))
```

정확도는 약 92%이다.

이것이 좋은 정확도인가? 아마 아닐 것이다. 사실 꽤 나쁘다. 왜냐하면 매우 단순한 모델을 사용하기 때문이다. 작은 변화를 주어 정확도를 97%까지 올릴 수 있다. 제일 좋은 모델은 정확도를 99.7% 이상으로 올릴 수 있다. (더 많은 정보는 [여기](https://rodrigob.github.io/are_we_there_yet/build/classification_datasets_results)에서 확인 할 수 있다.)

중요한 것은 이 모델에서 배웠다는 것이다. 그래도 이런 결과로 감이 잡힌다면 다음 튜토리얼에서 더 잘 살펴보고 텐서플로를 사용하여 복잡한 모델을 만드는 법을 배워보아라.

끝.