---
layout: post
title: "TensorBoard 시각화 배우기"
author: 이호종
comments: true
categories: [dev]

---

# TensorBoard: Visualizing Learning

방대한 심층 신경망을 학습하는 것과 같이 텐서플로를 사용한 계산은 복잡하고 난해할 수 있다. 텐서플로 프로그램을 쉽게 이해하고, 디버그하고, 최적화하기 위해 텐서보드(TensorBoard)라는 시각화 툴이 포함되어있다. 텐서보드를 사용하여 텐서플로 그래프를 시각화하고, 그래프의 실행에 대한 양적 측정기준을 그래프로 표현하고, 통과한 이미지와 같은 추가 데이터를 보여줄 수 있다. 텐서보드를 완전히 설정하면 다음과 같은 화면을 볼 수 있다.
![텐서보드 이미지](https://www.tensorflow.org/images/mnist_tensorboard.png)

[![TensorBoard 사용법](http://img.youtube.com/vi/eBbEDRsCmv4/0.jpg)](https://youtu.be/eBbEDRsCmv4?t=0s)

위 영상은 간단한 텐서보드 사용법을 담고 있다. 다른 사용가능할 자원도 담고 있다. [텐서보드 README]는 많은 텐서보드 사용법, 팁, 디버그 정보를 담고 있다.

## 데이터의 직렬화 (Serializing the data)

텐서보드는 텐서플로의 이벤트 파일을 읽어서 동작한다. 이벤트 파일은 텐서플로를 수행할 때 생성된 요약데이터를 가지고 있다. 텐서보드안의 요약데이터에 대한 일반적인 생애주기가 있다.

첫째, 텐서플로 그래프를 그린다. 요약 데이터를 수집하고 [요약 명령어](https://www.tensorflow.org/api_guides/python/summary)에 주석을 달 노드를 결정한다.

예를 들면, MNIST 숫자를 인식하기 위해 컨벌루션 신경망을 학습하는 것을 가정하자. 시간에 따라 학습 속도가 어떻게 달라지는지, 목적 함수가 변화되는지 기록하려 한다. 학습 속도와 손실을 각각 출력하는 노드에 [`tf.summary.scalar`](https://www.tensorflow.org/api_docs/python/tf/summary/scalar) 연산자를 연결하여 수집한다. 그러면, `학습 속도(learning rate)` 또는 `손실함수(loss fucntion)`와 같은 의미있는 `tag`를 `scalar_summary`에 제공한다.

아마도 특정 단계의 활성화 분포 또는 기울기나 가중치의 분포를 시각화하고 싶을 수 있다. 기울기 결과나 변수가 가지는 가중치에 각각 [`tf.summary.histogram`](https://www.tensorflow.org/api_docs/python/tf/summary/histogram) 연산자를 연결하여 이 데이터를 수집한다.

모든 사용가능한 요약 연산자의 자세한 내용은 [summary opertaions](https://www.tensorflow.org/api_guides/python/summary)에서 확인하자.

텐서플로에서 연산자는 실행될때 또는 결과에 의존성있는 연산이 수행될 때까지 아무것도 하지 않는다. 그리고 방금 생성된 요약 노드는 그래프의 주변장치이다. 현재 수행되는 어떤 연산자와도 연관성이 있지 않다. 그래서 요약을 생성하기 위해서 모든 요약 노드를 수행시킬 필요가 있다. 수동으로 요약노드를 관리하는 것은 지루한 일이다. 그래서 모든 요약 데이터를 생성하는 각 연산자를 합치기위해 [`tf.summary.merge_all`](https://www.tensorflow.org/api_docs/python/tf/summary/merge_all)을 사용하자.

그러면 주어진 단계에 요약데이터 전체에 대한 직렬화된 `요약(Summary)` protobuf 객체를 생성할 합쳐진 요약 연산자를 실행할 수 있다. 마지막으로 요약 데이터를 디스크에 쓰기 위해여 요약 protobuf를 [`tf.summary.FileWriter`](https://www.tensorflow.org/api_docs/python/tf/summary/FileWriter)에 전달한다.

`FileWriter`는 로그 디렉토리를 주어 생성한다. 로그 디렉토리는 꽤 중요하다. 모든 이벤트가 쓰여질 장소이다. 또한, `FileWriter`는 선택적으로 `그래프(Graph)`를 생성자에 인자로 사용할 수 있다. `그래프` 객체를 받으면 텐서보드는 텐서 모형 정보에 따라 그래프를 시각화한다. 그래프의 흐름을 통해 더 나은 시각적 정보를 주게된다. [Tensor shape information](https://www.tensorflow.org/get_started/graph_viz#tensor_shape_information)을 참고하자.

그래프를 수정하고 `FileWriter`를 갖게 되어서 네트워크를 실행할 준비가 되었다. 원한다면, 매 단계마다 합쳐진 요약 연산을 수행하고 많은 학습데이터를 기록할 수 있다. 그래도 필요한 것보다 더 많은 데이터가 될 가능성이 있다. 대신에 매 `n` 단계마다 요약 연산을 수행하는 것을 고려하자.

아래의 예제 코드는 [simple MNIST tutorial](https://www.tensorflow.org/get_started/mnist/beginners)을 수정한 것이다. 몇개의 요약 연산을 추가하고 매 10단계마다 수행하도록 수정하였다. 이것을 수행하고 `tensorboard --logdir=/tmp/tensorflow/mnist`를 실행하면 가중치나 정확도가 학습에 따라서 어떻게 변화되는지에 대한 통계를 시각화 할 수 있다. 아래 코드는 발췌된 것이고 전체 소스는 [여기](https://www.github.com/tensorflow/tensorflow/blob/r1.3/tensorflow/examples/tutorials/mnist/mnist_with_summaries.py)에서 확인할 수 있다.

```python
def variable_summaries(var):
  """Attach a lot of summaries to a Tensor (for TensorBoard visualization)."""
  with tf.name_scope('summaries'):
    mean = tf.reduce_mean(var)
    tf.summary.scalar('mean', mean)
    with tf.name_scope('stddev'):
      stddev = tf.sqrt(tf.reduce_mean(tf.square(var - mean)))
    tf.summary.scalar('stddev', stddev)
    tf.summary.scalar('max', tf.reduce_max(var))
    tf.summary.scalar('min', tf.reduce_min(var))
    tf.summary.histogram('histogram', var)

def nn_layer(input_tensor, input_dim, output_dim, layer_name, act=tf.nn.relu):
  """Reusable code for making a simple neural net layer.

  It does a matrix multiply, bias add, and then uses relu to nonlinearize.
  It also sets up name scoping so that the resultant graph is easy to read,
  and adds a number of summary ops.
  """
  # Adding a name scope ensures logical grouping of the layers in the graph.
  with tf.name_scope(layer_name):
    # This Variable will hold the state of the weights for the layer
    with tf.name_scope('weights'):
      weights = weight_variable([input_dim, output_dim])
      variable_summaries(weights)
    with tf.name_scope('biases'):
      biases = bias_variable([output_dim])
      variable_summaries(biases)
    with tf.name_scope('Wx_plus_b'):
      preactivate = tf.matmul(input_tensor, weights) + biases
      tf.summary.histogram('pre_activations', preactivate)
    activations = act(preactivate, name='activation')
    tf.summary.histogram('activations', activations)
    return activations

hidden1 = nn_layer(x, 784, 500, 'layer1')

with tf.name_scope('dropout'):
  keep_prob = tf.placeholder(tf.float32)
  tf.summary.scalar('dropout_keep_probability', keep_prob)
  dropped = tf.nn.dropout(hidden1, keep_prob)

# Do not apply softmax activation yet, see below.
y = nn_layer(dropped, 500, 10, 'layer2', act=tf.identity)

with tf.name_scope('cross_entropy'):
  # The raw formulation of cross-entropy,
  #
  # tf.reduce_mean(-tf.reduce_sum(y_ * tf.log(tf.softmax(y)),
  #                               reduction_indices=[1]))
  #
  # can be numerically unstable.
  #
  # So here we use tf.nn.softmax_cross_entropy_with_logits on the
  # raw outputs of the nn_layer above, and then average across
  # the batch.
  diff = tf.nn.softmax_cross_entropy_with_logits(targets=y_, logits=y)
  with tf.name_scope('total'):
    cross_entropy = tf.reduce_mean(diff)
tf.summary.scalar('cross_entropy', cross_entropy)

with tf.name_scope('train'):
  train_step = tf.train.AdamOptimizer(FLAGS.learning_rate).minimize(
      cross_entropy)

with tf.name_scope('accuracy'):
  with tf.name_scope('correct_prediction'):
    correct_prediction = tf.equal(tf.argmax(y, 1), tf.argmax(y_, 1))
  with tf.name_scope('accuracy'):
    accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
tf.summary.scalar('accuracy', accuracy)

# Merge all the summaries and write them out to /tmp/mnist_logs (by default)
merged = tf.summary.merge_all()
train_writer = tf.summary.FileWriter(FLAGS.summaries_dir + '/train',
                                      sess.graph)
test_writer = tf.summary.FileWriter(FLAGS.summaries_dir + '/test')
tf.global_variables_initializer().run()
```

`FileWriters`를 초기화한 후에 모델을 학습하고 테스트하기 위해 `FileWriters`에 요약들을 추가해야 한다.

```python
# Train the model, and also write summaries.
# Every 10th step, measure test-set accuracy, and write test summaries
# All other steps, run train_step on training data, & add training summaries

def feed_dict(train):
  """Make a TensorFlow feed_dict: maps data onto Tensor placeholders."""
  if train or FLAGS.fake_data:
    xs, ys = mnist.train.next_batch(100, fake_data=FLAGS.fake_data)
    k = FLAGS.dropout
  else:
    xs, ys = mnist.test.images, mnist.test.labels
    k = 1.0
  return {x: xs, y_: ys, keep_prob: k}

for i in range(FLAGS.max_steps):
  if i % 10 == 0:  # Record summaries and test-set accuracy
    summary, acc = sess.run([merged, accuracy], feed_dict=feed_dict(False))
    test_writer.add_summary(summary, i)
    print('Accuracy at step %s: %s' % (i, acc))
  else:  # Record train set summaries, and train
    summary, _ = sess.run([merged, train_step], feed_dict=feed_dict(True))
    train_writer.add_summary(summary, i)
```

이제 텐서보드를 사용하여 데이터를 시각화하기 위해 모든 것을 설정하였다.

> 위 코드만으로는 수행할 수 없다. 전체 소스를 참고하여 실행하도록 하자. 전체 소스를 아무런 인자 없이 돌리면 총 1000번 단계가 수행된 뒤에 프로그램이 끝난다.

## 텐서보드 실행 (Launching TensorBoard)

텐서보드를 실행시키기 위해서 아래 명령어를 사용하면된다. (대안은`python -m tensorflow.tensorboard` 이다.)

```bash
tensorboard --logdir=path/to/log-directory

# 난 아래 명령어를 통해 학습과 테스트를 텐서보드에서 확인하였다.
tensorboard --logdir=/tmp/tensorflow/mnist/logs/mnist_with_summaries/
```

`logdir`은 `FileWriter`가 데이터를 저장한 디렉토리를 가리킨다. `logdir` 디렉토리가 각 실행을 통해 직렬화된 데이터를 포함하는 하위 디렉토리를 포함하면, 텐서보드는 모든 수행을 시각화한다. 텐서보드를 실행하고 웹브라우저에서 `localhost:6006`을 접속하여 텐서보드를 볼 수 있다.

텐서보드를 보면 위-오른쪽 구석에서 네비게이션 탭이 있다. 각 탭은 시각화 할 수 있는 직렬화된 셋을 나타낸다.

그래프를 시각화하기 위하여 *graph*를 사용하는 방법의 자세한 내용은 다음 [TensorBoard: Graph Visualization](https://www.tensorflow.org/get_started/graph_viz)에서 볼 수 있다.

일반적인 텐서보드의 더 많은 사용법은 [TensorBoard README](https://www.github.com/tensorflow/tensorflow/blob/r1.3/tensorflow/tensorboard/README.md)에서 확인하자.
