# SGNS with tied weights 

Code for the SGNS model with tied embeddings from the paper [Context Vectors Are Reflections of Word Vectors in Half the Dimensions](https://jair.org/index.php/jair/article/view/11368) (JAIR)

### Requirements
Code is written in Python 3 and requires TensorFlow 1.1+.

### Datasets
Assuming you have cloned the git repository, navigate into this directory. To download the example text and evaluation data:

```shell
curl http://mattmahoney.net/dc/text8.zip > text8.zip
unzip text8.zip
curl https://storage.googleapis.com/google-code-archive-source/v2/code.google.com/word2vec/source-archive.zip > source-archive.zip
unzip -p source-archive.zip  word2vec/trunk/questions-words.txt > questions-words.txt
rm text8.zip source-archive.zip
```

### TensorFlow Ops
You will need to compile the ops as follows (See 
[Adding a New Op to TensorFlow](https://www.tensorflow.org/how_tos/adding_an_op/#building_the_op_library)
for more details).:

```shell
TF_CFLAGS=( $(python -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_compile_flags()))') )
TF_LFLAGS=( $(python -c 'import tensorflow as tf; print(" ".join(tf.sysconfig.get_link_flags()))') )
g++ -std=c++11 -shared word2vec_ops.cc word2vec_kernels.cc -o word2vec_ops.so -fPIC ${TF_CFLAGS[@]} ${TF_LFLAGS[@]} -O2 -D_GLIBCXX_USE_CXX11_ABI=0
```

On Mac, add `-undefined dynamic_lookup` to the g++ command. The flag `-D_GLIBCXX_USE_CXX11_ABI=0` is included to support newer versions of gcc. However, if you compiled TensorFlow from source using gcc 5 or later, you may need to exclude the flag. Specifically, if you get an error similar to the following: `word2vec_ops.so: undefined symbol: _ZN10tensorflow7strings6StrCatERKNS0_8AlphaNumES3_S3_S3_` then you likely need to exclude the flag.

### Folders
Prepare folders for checkpoints, vocabulary, and embeddings:
```shell
mkdir embeddings
mkdir saves
```

### Training
Once you've successfully compiled the ops, run the model as follows:

```shell
python word2vec_wt.py \
  --train_data=text8 \
  --eval_data=questions-words.txt \
  --save_path=saves
  --is_train=True
```

### Embeddings
To fetch the trained embeddings run the model with `--is_train=False` and provide the appropriate checkpoint through the `--checkpoint` flag.
