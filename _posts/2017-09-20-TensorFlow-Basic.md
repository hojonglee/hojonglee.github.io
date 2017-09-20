---
layout: post
title: "tensorflow 기본개념"
author: "이호종"
categories: [dev]
comments: true

---

## Tensorflow
텐서플로는 데이터 흐름 그래프를 이용하여 계산을 하는 오픈소스 라이브러리이다. 구글 브레인 팀에서 개발을 하였고 지금 머신러닝 분야에서 엄청나게 사용 중이다. 그래서 나도 텐서플로를 공부하고 처음에 접하면서 필요하다고 생각했던 것을 글로 남기려고 한다.

나는 지금 잔카를로 자코네가 쓰고, 김창엽씨가 옮긴 [텐서플로 입문(예제로 배우는 텐서플로)](http://book.daum.net/detail/book.do?bookid=KOR9788960779198)를 읽으면서 공부 중인데 초반부터 자주 등장하는 tf.constant(), tf.placeholder(), tf.Variable() 이 세가지 함수 또는 클래스가 어떤 역할을 하는지 이해가 잘 안되어 이 부분에 대하여 알아보겠다.
아래 내용은 [Getting Started With TensorFlow](https://www.tensorflow.org/get_started/get_started)를 참고하여 작성하였다.

참고로 난 텐서플로의 python 버전으로 공부를 하고 있다.

## APIs

텐서플로는 여러 API를 제공하는데 저레벨 API(TensorFlow Core)는 연구자나 자체 모델을 컨트롤 하는 경우 사용하고 일반적인 개발자는 좀 더 사용하기 쉬운 고레벨 API를 사용하는 것을 권장한다. 고레벨 API는 내부에서 TensorFlow Core를 이용하여 개발되어있고 사용하기에 편리하다.

## Tensor

텐서플로에서 다루는 데이터의 단위는 **tensor** 이다. 텐서는 여러 차원의 배열의 형태를 한 숫자값의 모음으로 구성되어 있다. 텐서의 **rank**는 차원의 수이다. 아래 텐서의 예제를 확인해보자

```python
3 # a rank 0 tensor; this is a scalar with shape []
[1., 2., 3.] # a rank 1 tensor; this is a vector with shape [3]
[[1., 2., 3.], [4., 5., 6.]] # a rank 2 tensor; a matrix with shape [2, 3]
[[[1., 2., 3.]], [[7., 8., 9.]]] # a rank 3 tensor with shape [2, 1, 3]
```

### Importing TensorFlow

python을 사용하는 경우 아래와 같은 방법으로 텐서플로를 사용할 수 있다.

```python
import tensorflow as tf
```

가이드 문서의 모든 코드에는 위 코드가 모두 포함된 것으로 가정한다.

### The Computational Graph

TensorFlow Core 프로그래밍은 아래의 두 단계로 생각 할 수 있다.

1. computational graph 구성
2. computational graph 실행

계산그래프(computational graph)는 일련의 텐서플로 연산자를 노드 그래프로 배열한 것이다. 작은 계산 그래프를 만들어보자. 각 노드는 0개 이상의 텐서를 입력값으로 받아서 텐서를 출력값으로 생산한다. 노드의 한 유형은 상수이다. 모든 텐서플로 상수와 같이 입력값이 없고 내부적으로 저장하는 값을 출력한다. 아래 예제는 두개의 부동소수점 텐서 `node1`과 `node2`를 생성한다.

```python
node1 = tf.constant(3.0, dtype=tf.float32)
node2 = tf.constant(4.0) # 암묵적으로 tf.float32이다.
print node1, node2
```

마지막 프린트문은 결과로

```python
Tensor("Const:0", shape=(), dtype=float32) Tensor("Const_1:0", shape=(), dtype=float32)
```

을 생성한다.
예상했다시피 결과는 `3.0`, `4.0`이 아니다. 대신에 계산될때, 노드는 3.0과 4.0을 생산한다. 실제 노드를 계산하기 위해서 **session**을 통해 계산 그래프를 수행해야 한다. 세션은 텐서플로 런타임의 제어와 상태를 캡슐화한다.

아래 코드는 `Session` 객체를 생성하고 `run` 메소드를 수행하여 `node1`과 `node2`를 계산하기 위한 계산그래프를 실행한다.

```python
sess = tf.Session()
print sess.run([node1, node2])
```

예상한 값인 3.0과 4.0을 확인 할 수 있다.

```python
[3.0, 4.0]
```

복잡한 계산을 위해서 `텐서` 노드와 연산자를 함께 사용여 구성할 수 있다(연산자도 노드이다).

```python
node3 = tf.add(node1, node2)
print "node3:", node3
print "sess.run(node3):", sess.run(node3)
```

결과는
```python
node3: Tensor("Add:0", shape=(), dtype=float32)
sess.run(node3): 7.0
```
이다.

텐서플로는 텐서보드라 불리는 계산 그래프의 그림을 보여주는 도구를 제공한다. 아래 이미지는 텐서보드에서 보여주는 화면이다.

![텐서보드 예제](https://www.tensorflow.org/images/getting_started_add.png)

위를 보면 항상 상수를 결과값으로 주기 때문에 흥미롭지는 않다. 계산 그래프는 외부에서 **placeholders**로 알려져 있는 외부 입력값을 받아서 파라메터화 할 수 있다. **placeholder**는 나중에 값을 제공하겠다는 약속이다.

```python
a = tf.placeholder(tf.float32)
b = tf.placeholder(tf.float32)
adder_node = a + b  # + provides a shortcut for tf.add(a, b)
```

위 세줄은 두개의 입력 파라메터를 정의하고 그것을 계산하는 작은 함수 혹은 람다이다. run 메소드에 대한 feed\_dict 인자를 사용하여 구체적인 값을 placeholder에 넘겨줌으로써 다중 입력으로 이 그래프를 측정 할 수 있다.

```python
print sess.run(adder_node, {a:3, b:4.5})
print sess.run(adder_node, {a:[1,3], b:[2,4]})
```

결과는

```python
7.5
[ 3.  7.]
```

텐서보드에서 계산그래프는 다음과 같다.
![adder_node](https://www.tensorflow.org/images/getting_started_adder.png)

> placeholder는 텐서플로를 처음 접하면서 가장 헷갈렸던 것 중에 하나이다. 위 내용을 보면 `tf.placeholder()`를 통해 변수를 생성하면 텐서플로는 런타임에 값을 넣어서 계산한다는 것으로 이해할 수 있다.

머신러닝에서는 대개 임의의 입력값을 받는 모델을 원한다. 모델을 학습할 수 있게 만드려면 그래프가 같은 입력값에 대하여 새 출력값을 얻을수 있게 수정 할 수 있어야 한다. **Variable**은 그래프에 학습가능한 파라메터를 더할 수 있게 해준다. **Variable**은 타입과 초기값을 주어 생성한다.

```python
W = tf.Variable([.3], dtype=tf.float32)
b = tf.Variable([-.3], dtype=tf.float32)
x = tf.placeholder(tf.float32)
linear_model = W * x + b
```

상수는 `tf.constant`를 호출하여 초기화 하고 그 값은 절대 변경되지 않는다. 대조적으로, 변수(Variable)는 `tf.Variable`을 호출할때 초기화하지 않는다. 텐서플로 프로그램에서 모든 변수를 초기화하기 위하여 아래의 특별한 함수를 명시적으로 호출해야 한다.

```python
init = tf.global_variables_initializer()
sess.run(init)
```

`init`이 모든 전역변수를 초기화하는 텐서플로의 하위 그래프의 핸들임을 인지하는 것이 중요한다. `sess.run`을 호출하기 전에 모든 변수는 초기화되지 않는다.

아래와 같이 `x`는 placeholder이기 때문에 `linear_model`을 `x`의 여러 값으로 동시에 계산 할 수 있다.

```python
print sess.run(linear_model, {x:[1,2,3,4]})
```

결과는
```python
[ 0.          0.30000001  0.60000002  0.90000004]
```
이다.

모델을 생성했지만 얼마나 좋은지는 아직 알지 못한다. 학습데이터로 모델을 평가하기 위해 원하는 값을 제공하기 위한 `y` placeholder가 필요하다. 그리고 손실함수(loss function)를 작성할 필요가 있다.

손실함수는 현재 모델이 제공된 데이터로부터 얼마나 차이가 나는지 측정한다. 현재 모델과 제공된 데이터의 차이를 제곱하여 합하는 선형 회귀(linear regression)을 위해 표준 손실모델을 사용하겠다. `linear_model -y`는 각 요소가 상응하는 예제의 에러 차이를 나타내는 벡터를 생성한다. `tf.square`는 에러를 제곱하기위해 호출한다. `tf.reduce_sum`을 사용하여 모든 예제의 에러를 요약하는 단일 스칼라를 생성하기 위해 모든 에러의 제곱을 더한다.

```python
y = tf.placeholder(tf.float32)
squared_deltas = tf.square(linear_model - y)
loss = tf.reduce_sum(squared_deltas)
print sess.run(loss, {x: [1, 2, 3, 4], y: [0, -1, -2, -3]})
```

생성된 손실 값은
```python
23.66
```
이다.

`W`와 `b`에 완전한 값인 -1과 1로 재할당하여 손실함수를 수동으로 개선할 수 있다. 변수는 `tf.Variable`이 제공하는 값으로 초기화되지만 `tf.assign`과 같은 함수를 사용하여 값을 변경 할 수 있다. 예를들면, `W=-1`과 `b=1`은 이 모델에서 최적 인자이다. `W`와 `b`를 아래와 같이 바꿀 수 있다.

```python
fixW = tf.assign(W, [-1.])
fixb = tf.assign(b, [1.])
sess.run([fixW, fixb])
print sess.run(loss, {x: [1, 2, 3, 4], y: [0, -1, -2, -3]})
```

마지막 print의 결과는 손실이 0임을 보여준다.
```python
0.0
```

`W`와 `b`의 "완전한" 값을 추측할 수 있지만 머신러닝의 전체 관점은 정확한 모델 인자를 자동으로 찾아내는 것이다. 다음 섹션에서 어떻게 달성하는지 보여줄 것이다.

> 이번 포스팅은 여기까지로 하자. 원래 목적은 tf.constant, tf.Variable, tf.placeholder가 텐서플로에서 어떤 기능을 하는지 알아보는 것이었다.
> 1. tf.constant: 텐서플로 상수를 생성, 초기화 후 변경되지 않음
> 2. tf.placeholder: 런타임에 값을 할당할 수 있는 텐서
> 3. tf.Variable: init 함수를 호출하는 시점에 초기화 됨, 수행 중에 다른 값을 할당할 수 있음

텐서플로를 처음 공부하면서 작성했습니다. 틀린 것이 있으면 댓글로 알려주시면 확인 후 수정하겠습니다. 이 뒷부분은 다음 포스팅에 작성하겠습니다.

끝.
