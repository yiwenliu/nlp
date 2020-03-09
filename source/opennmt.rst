opennmt
==============
安装opennmt
----------------
Requirements
^^^^^^^^^^^^^^^^
OpenNMT-tf requires:

Python >= 2.7

TensorFlow >= 1.4, < 2.0

python虚拟环境
^^^^^^^^^^^^^^^^^^^
在gpu ubuntu pc上，当在terminal中运行python时，会寻找到anaconda的python，因为把/home/anaconda/bin添加到了PATH的最前面，shell在找到anaconda的python后，就不会继续寻找unbuntu系统自带的python了。

查找以前的笔记，使用命令

.. code-block:: none
	:linenos:

	$conda env list #在anaconda中创建了一个虚拟环境，名为tf3.5。
	$source activate tf3.5#激活这个环境

在上面的环境中，opennmt运行quickstart失败，所以新建了一个环境

.. code-block:: none
	:linenos:

	$conda create -n opennmt_py2 python=2.7
	$source activate opennmt_py2

安装tf
^^^^^^^^^^^^^
因为新建了python虚拟环境，所以要重新安装tf，选择1.4版本，因为要和cuda和cudnn的版本匹配。

$pip install tensorflow-gpu==1.4 #下载tf1.4时，耗时太长，从terminal的提示链接中下载了.whl

(opennmt_py2)$pip install tensorflow_gpu-1.4xxx.whl --default-timeout=10000000000
#--default-timeout：下载了.whl还是要用这个option是因为在安装tf的过程中，还要下载安装别的组件

查看tensorflow版本
^^^^^^^^^^^^^^^^^^^
.. code-block:: none
	:linenos:

	$python
	>>>import tensorflow as tf
	>>>tf.__version__
	'1.4.0' #满足opennmt的要求

install via pip
^^^^^^^^^^^^^^^^^^^
(xx)superben@NewsAI:pip install OpenNMT-tf

安装路径~/anaconda3/envs/opennmt_py2/lib/python2.7/site-packages/opennmt

File format
----------------
1. 文本

- sentences are separated by a newline.
- src-train.txt等都需要utf-8编码。ubuntu中有一个命令可以把我在windows机器上的gbk文本转换成utf-8

.. code-block:: none
	:linenos:

	$iconv -f gbk -t utf-8 1.txt > 2.txt


2. Vocabulary

训练可视化
--------------
在安装tensorflow时，已经安装了tensorboard。执行如下命令，

.. code-block:: none
	:linenos:

	(xx)superben@NewsAI:tensorboard --logdir="run"
	>>>http://NewsAI:6006

分词
--------
采用的方案
^^^^^^^^^^^^^^
因为boost的原因，放弃使用opennmt的pyonmttok，而是用“数据服务平台的分词服务”，生成分好词的（用空格）的src-train.txt和tgt-train.txt

- 使用的命令

.. code-block:: none
	:linenos:

	$onmt-build-vocab --save_vocab src-vocab.txt src-train.txt --tokenizer SpaceTokenizer

- 使用配置文件

配置文件中只需要一行：

.. code-block:: none
	:linenos:

	mode: space

tokenizer配置文件
^^^^^^^^^^^^^^^^^^^^^^^^^^^
1. 文件格式

http://zh.opennmt.net/OpenNMT-tf/tokenization.html#configuration-files

2. 文件中参数的意义

https://github.com/OpenNMT/Tokenizer/blob/master/docs/options.md

在命令中使用tokenizer配置文件
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
.. code-block:: none
	:linenos:

	$onmt-build-vocab -h查看帮助
	$onmt-build-vocab  --save_vocab src-vocab.txt --tokenizer_config tokenizer.yaml src-train.txt

运行$onmt-build-vocab出错
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
1.
~/anaconda3/envs/opennmt_py2/lib/python2.7/site-packages/opennmt/tokenizers/tokenizer.py   line47

.. code-block:: none
	:linenos:

	-yaml.load(conf_file)
	+yaml.load(conf_file, Loader=yaml.SafeLoader)

2. pyonmttok调用boost时出错，"did not match C++ signature"

原以为是boost和python2.7的版本不匹配，使用$conda install -c anaconda boost在虚拟环境下安装了boost，但是还是报错。

这个问题无法解决，放弃使用opennmt的pyonmttok，而是用“数据服务平台的分词服务”。

使用opennmt
-------------
参考：http://opennmt.net/OpenNMT-tf/quickstart.html

2019.3月在gpu机器上，安装了环境后，跑了一把，发现标题和原文不符合，原本以为是数据量不够，整理了一个30w+的训练对后，再次运行，发现还是一样的结果。

.. code-block:: none
	:linenos:

	$source activate opennmt_py2
	#把训练数据放在~/opennmt_data/all目录下
	$cd opennmt_data/all
	#build the source and target word vocabularies from the training files
	$onmt-build-vocab --size 50000 --save_vocab src-vocab.txt src-train.txt
	$onmt-build-vocab --size 50000 --save_vocab tgt-vocab.txt tgt-train.txt
	#在~/opennmt_data/all目录下配置文件data.yml
	model_dir: run/

	data:
	  train_features_file: src-train.txt
	  train_labels_file: tgt-train.txt
	  eval_features_file: src-val.txt
	  eval_labels_file: tgt-val.txt
	  source_words_vocabulary: src-vocab.txt
	  target_words_vocabulary: tgt-vocab.txt

	#train the model
	$onmt-main train_and_eval --model_type NMTSmall --auto_config --config data.yml
	#translate
	$onmt-main infer --auto_config --config data.yml --features_file src-test.txt