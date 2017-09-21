---
layout: post
title: "tensorflow 학습과 검증"
author: "이호종"
categories: [dev]
comments: true

---

이번에는 저번 포스팅에 이어서 학습과 검증 API에 대해서 알아보자
[Getting Started With TensorFlow](https://www.tensorflow.org/get_started/get_started)에서 중간 이후의 내용이다.

## tf.train API

머신러닝의 완전한 논의는 이번 튜토리얼에서 다루지 않는다. 그러나 텐서플로는 손실함수(loss function)을 최소화 하기 위한 각 변수를 천천히 변화시키는 최적화도구(**optimizer**)를 제공한다. 가장 간단한 최적화도구는 **경사하강법(gradient descent)**이다. 경사하강법은 해당 변수를 반영하여 손실의 유도된 정도에 따라 각 변수를 수정한다. 일반적으로 손으로 기호 미분계수(symbolic derivatives)를 계산하는 것은 지루하고 오류가 발생하기 쉽다. 결과적으로 텐서플로는 `tf.gradients` 함수를 사용하여 단지 모델의 설명만 주어지면 자동으로 미분계수를 생성한다. 단순하게 최적화 도구는 미분계수를 계산해준다.

> 코드는 이전 포스팅의 코드와 연결된다.


```python
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)
```

```python
sess.run(init) # reset values to incorrect defaults.
for i in range(1000):
  sess.run(train, {x: [1, 2, 3, 4], y: [0, -1, -2, -3]})

print sess.run([W, b])
```

최종 모델 인자의 결과:
```python
[array([-0.9999969], dtype=float32), array([ 0.99999082], dtype=float32)]
```

이제 실제 머신러닝을 완성했다. 간단한 회귀분석은 큰 텐서플로 코어 코드를 요구하지 않지만, 더 복잡한 모델과 모델에 데이터를 주입하는 메소드는 더 많은 코드를 필요로 한다. 그러므로 텐서플로는 공통 패턴, 구조, 기능에 대한 높은 수준의 추상화를 제공한다. 아래에서 이런 추상화를 어떻게 이용하는지 알아보자.

### Complete program
아래에 완전한 회귀분석모들을 보여주겠다.

```python
import tensorflow as tf

# Model parameters
W = tf.Variable([.3], dtype=tf.float32)
b = tf.Variable([-.3], dtype=tf.float32)
# Model input and output
x = tf.placeholder(tf.float32)
linear_model = W * x + b
y = tf.placeholder(tf.float32)

# loss
loss = tf.reduce_sum(tf.square(linear_model - y)) # sum of the squares
# optimizer
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)

# training data
x_train = [1, 2, 3, 4]
y_train = [0, -1, -2, -3]
# training loop
init = tf.global_variables_initializer()
sess = tf.Session()
sess.run(init) # reset values to wrong
for i in range(1000):
  sess.run(train, {x: x_train, y: y_train})

# evaluate training accuracy
curr_W, curr_b, curr_loss = sess.run([W, b, loss], {x: x_train, y: y_train})
print "W: %s b: %s loss: %s"%(curr_W, curr_b, curr_loss)
```

결과값은 아래와 같다.
```python
W: [-0.9999969] b: [ 0.99999082] loss: 5.69997e-11
```

손실은 매우 작은 숫자이다(0에 근접한). 이 프로그램을 실행하면 모델이 수도랜덤 값으로 초기화되기 때문에 손실이 완전히 같지는 않을 것이다.

더 복잡한 프로그램도 텐서보드를 이용하여 시각화할 수 있다.
![손실함수 모델](https://www.tensorflow.org/images/getting_started_final.png)

## tf.estimator

`tf.estimator`는 머신러닝의 역학을 단순화한 높은 수준의 텐서플로 라이브러리이다. 아래와 같은 구성요소를 담고 있다.

- 학습단계를 수행한다.
- 평가단계를 수행한다.
- 데이터 셋을 관리한다.

tf.estimator는 많은 공통 모델을 정의한다.

### Basic usage

`tf.estimator`를 사용하면 회귀분석 프로그램이 얼마나 단순해질까:

```python
import tensorflow as tf
# NumPy is often used to load, manipulate and preprocess data.
import numpy as np

# Declare list of features. We only have one numeric feature. There are many
# other types of columns that are more complicated and useful.
feature_columns = [tf.feature_column.numeric_column("x", shape=[1])]

# An estimator is the front end to invoke training (fitting) and evaluation
# (inference). There are many predefined types like linear regression,
# linear classification, and many neural network classifiers and regressors.
# The following code provides an estimator that does linear regression.
estimator = tf.estimator.LinearRegressor(feature_columns=feature_columns)

# TensorFlow provides many helper methods to read and set up data sets.
# Here we use two data sets: one for training and one for evaluation
# We have to tell the function how many batches
# of data (num_epochs) we want and how big each batch should be.
x_train = np.array([1., 2., 3., 4.])
y_train = np.array([0., -1., -2., -3.])
x_eval = np.array([2., 5., 8., 1.])
y_eval = np.array([-1.01, -4.1, -7, 0.])
input_fn = tf.estimator.inputs.numpy_input_fn(
    {"x": x_train}, y_train, batch_size=4, num_epochs=None, shuffle=True)
train_input_fn = tf.estimator.inputs.numpy_input_fn(
    {"x": x_train}, y_train, batch_size=4, num_epochs=1000, shuffle=False)
eval_input_fn = tf.estimator.inputs.numpy_input_fn(
    {"x": x_eval}, y_eval, batch_size=4, num_epochs=1000, shuffle=False)

# We can invoke 1000 training steps by invoking the  method and passing the
# training data set.
estimator.train(input_fn=input_fn, steps=1000)

# Here we evaluate how well our model did.
train_metrics = estimator.evaluate(input_fn=train_input_fn)
eval_metrics = estimator.evaluate(input_fn=eval_input_fn)
print "train metrics: %r"% train_metrics
print "eval metrics: %r"% eval_metrics
```

수행하면 다음과 같은 결과가 나온다.
```python
train metrics: {'loss': 1.2712867e-09, 'global_step': 1000}
eval metrics: {'loss': 0.0025279333, 'global_step': 1000}
```

평가데이터가 손실(loss)가 클 수 있지만 여전히 0에 가깝다. 이것은 학습이 적절했다는 것을 뜻한다.

### A custom model

`tf.estimator`은 이미 정의된 모델로 한정하지 않는다. 텐서플로에 내장되지 않은 사용자 모델을 생성하는 것을 가정하자. 여전히 `tf.estimator`의 데이터 셋, 데이터 주입, 학습 등을 높은 수준의 추상화로 남겨둘 수 있다. 설명을 위해, 저수준 텐서플로 API(lower level TensorFlow API)의 지식을 사용하여 `LinearRegressor`와 동등한 자체모델을 어떻게 구현하는지 보여주겠다.

`tf.estimator`에 동작하는 사용자 모델을 정의하기 위해서 `tf.estimator`를 사용해야한다. `tf.estimator.LinearRegressor`는 `tf.estimator.Estimator`의 실제 서브 클래스이다. 서브클래스인 `Estimator` 대신에 `Estimator`에 예측, 학습과정, 손실을 평가할 수 있는 방법을 `tf.estimator`에 알려주는 함수 `model_fn`을 제공하면 된다. 코드는 다음과 같다.

```python
import numpy as np
import tensorflow as tf

# Declare list of features, we only have one real-valued feature
def model_fn(features, labels, mode):
  # Build a linear model and predict values
  W = tf.get_variable("W", [1], dtype=tf.float64)
  b = tf.get_variable("b", [1], dtype=tf.float64)
  y = W * features['x'] + b
  # Loss sub-graph
  loss = tf.reduce_sum(tf.square(y - labels))
  # Training sub-graph
  global_step = tf.train.get_global_step()
  optimizer = tf.train.GradientDescentOptimizer(0.01)
  train = tf.group(optimizer.minimize(loss),
                   tf.assign_add(global_step, 1))
  # EstimatorSpec connects subgraphs we built to the
  # appropriate functionality.
  return tf.estimator.EstimatorSpec(
      mode=mode,
      predictions=y,
      loss=loss,
      train_op=train)

estimator = tf.estimator.Estimator(model_fn=model_fn)
# define our data sets
x_train = np.array([1., 2., 3., 4.])
y_train = np.array([0., -1., -2., -3.])
x_eval = np.array([2., 5., 8., 1.])
y_eval = np.array([-1.01, -4.1, -7, 0.])
input_fn = tf.estimator.inputs.numpy_input_fn(
    {"x": x_train}, y_train, batch_size=4, num_epochs=None, shuffle=True)
train_input_fn = tf.estimator.inputs.numpy_input_fn(
    {"x": x_train}, y_train, batch_size=4, num_epochs=1000, shuffle=False)
eval_input_fn = tf.estimator.inputs.numpy_input_fn(
    {"x": x_eval}, y_eval, batch_size=4, num_epochs=1000, shuffle=False)

# train
estimator.train(input_fn=input_fn, steps=1000)
# Here we evaluate how well our model did.
train_metrics = estimator.evaluate(input_fn=train_input_fn)
eval_metrics = estimator.evaluate(input_fn=eval_input_fn)
print "train metrics: %r"% train_metrics
print "eval metrics: %r"% eval_metrics
```

실행하면 결과는 다음과 같다.

```python
train metrics: {'loss': 1.227995e-11, 'global_step': 1000}
eval metrics: {'loss': 0.01010036, 'global_step': 1000}
```

사용자 `model_fn()` 함수의 내용이 저수준 API의 수동 모델 학습과정과 얼마나 유사한지 주목해야 한다.

---
2번에 걸친 텐서플로의 기본 내용을 포스팅했는데 나처럼 텐서플로에 처음 접하는 사람이 읽고 도움이 되었으면 한다. 다음은 [MNIST For ML Beginners](https://www.tensorflow.org/get_started/mnist/beginners)이다. 이 부분은 추후에 살펴보게 되면 포스팅을 진행하겠다. 다음에는 따라하기가 아닌 내가 해보고 시행착오를 겪은 내용을 추가할 수 있도록 해보겠다.

끝.

