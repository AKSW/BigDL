## **Loading a tensorflow model from tensorflow model file**

If you have a pre-trained tensorflow model saved in a ".pb" file. You load it
into BigDL using `Model.load_tensorflow` api. 

For more information on how to generate
the ".pb" file, you can refer to [A Tool Developer's Guide to TensorFlow Model Files](https://www.tensorflow.org/extend/tool_developers/).
Specifically, you should generate a model definition file and a set of checkpoints, then use the [freeze_graph](https://github.com/tensorflow/tensorflow/blob/v1.0.0/tensorflow/python/tools/freeze_graph.py)
script to freeze the graph definition and weights in checkpoints into a single file.

**Generate model definition file and checkpoints in python**
```python
import tensorflow as tf
xs = tf.placeholder(tf.float32, [None, 1])
W1 = tf.Variable(tf.zeros([1,10])+0.2)
b1 = tf.Variable(tf.zeros([10])+0.1)
Wx_plus_b1 = tf.nn.bias_add(tf.matmul(xs,W1), b1)
output = tf.nn.tanh(Wx_plus_b1, name="output")

saver = tf.train.Saver()
with tf.Session() as sess:
    init = tf.global_variables_initializer()
    sess.run(init)
    checkpointpath = saver.save(sess, '/tmp/model/test.chkp')
    tf.train.write_graph(sess.graph, '/tmp/model', 'test.pbtxt')
```

**Freeze graph definition and checkpoints into a single ".pb" file in shell**
```shell
wget https://raw.githubusercontent.com/tensorflow/tensorflow/v1.0.0/tensorflow/python/tools/freeze_graph.py
python freeze_graph.py --input_graph /tmp/model/test.pbtxt --input_checkpoint /tmp/model/test.chkp --output_node_names=output --output_graph "/tmp/model/test.pb"
```

**Load tensorflow model in Scala**
```scala
import com.intel.analytics.bigdl.utils._
val path = "/tmp/model/test.pb"
val inputs = Seq("Placeholder")
val outputs = Seq("output")
val model = TensorflowLoader.load(path, Seq("Placeholder"), Seq("output"), ByteOrder.LITTLE_ENDIAN)
```

**Load tensorflow model in python**
```python
path = "/tmp/model/test.pb"
inputs = ["Placeholder"]
outputs = ["output"]
model = Model.load_tensorflow(path, inputs, outputs, byte_order = "little_endian", bigdl_type="float")
```
---

## **Saving a BigDL Graph model to tensorflow model file**

You can also save a graph model to protobuf files so that it can be used in tensorflow inference.

When saving the model, placeholders will be added to the tf model as input nodes. So
you need to pass in the names and shapes of the placeholders. BigDL model doesn't have
such information. The order of the placeholder information should be same as the inputs
of the graph model.

**Scala**
```scala
import com.intel.analytics.bigdl.nn._
import com.intel.analytics.bigdl.utils.tf.TensorflowSaver
import com.intel.analytics.bigdl.tensor.TensorNumericMath.TensorNumeric.NumericFloat
// create a graph model
val linear = Linear(10, 2).inputs()
val sigmoid = Sigmoid().inputs(linear)
val softmax = SoftMax().inputs(sigmoid)
val model = Graph(Seq[linear], Seq[softmax])

// save it to tensorflow model file
TensorflowSaver.saveGraph(model, Seq(("input", Seq(4, 10))), "/tmp/model.pb")
```

**Python**
```python
from bigdl.nn.layer import *
from bigdl.optim.optimizer import *
from bigdl.util.common import *

# create a graph model
linear = Linear(10, 2)()
sigmoid = Sigmoid()(linear)
softmax = SoftMax()(sigmoid)
model_original = Model([linear], [softmax])

# save it to tensorflow model file
Model.save_tensorflow(model_original, [("input", [4, 10])], "/tmp/model.pb")
```


---
## **Build model using tensorflow and train with BigDL**

You can construct your BigDL model directly from the input and output nodes of
tensorflow model. That is to say, you can use tensorflow to define
a model and use BigDL to run it.

**Python:**
```python
import tensorflow as tf
import numpy as np

tf.set_random_seed(1234)
input = tf.placeholder(tf.float32, [None, 5])
weight = tf.Variable(tf.random_uniform([5, 10]))
bias = tf.Variable(tf.random_uniform([10]))
middle = tf.nn.bias_add(tf.matmul(input, weight), bias)
output = tf.nn.tanh(middle)

tensor = np.random.rand(5, 5)
# construct BigDL model and get the result form 
bigdl_model = Model(input, output, model_type="tensorflow")
```