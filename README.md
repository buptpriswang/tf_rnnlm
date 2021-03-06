# Recurrent Neural Network Language Model using TensorFlow

## Content
* [Original Work](#orig)
* [Motivations](#motivations)
* [Quickstart](#quickstart)
* [Continue Training](#continue)
* [Getting a text perplexity with regard to the LM](#test)
* [Line by line loglikes](#loglikes)
* [Results](#results)


---

## [Original Work](#orig)

Our work is based on the [RNN LM tutorial on tensorflow.org](https://www.tensorflow.org/versions/r0.11/tutorials/recurrent/index.html#recurrent-neural-networks) following the paper from [Zaremba et al., 2014](https://arxiv.org/abs/1409.2329).

The tutorial uses the PTB dataset ([tgz](http://www.fit.vutbr.cz/~imikolov/rnnlm/simple-examples.tgz)). We intend to work with various dataset, that's why we made names more generic by removing several "PTB" prefix initially present in the code.

**Original sources:** [TensorFlow v0.11 - PTB](https://github.com/tensorflow/tensorflow/tree/282823b877f173e6a33bbc9d4b9ad7dd8413ada6/tensorflow/models/rnn/ptb)

**See also:** 
- [Nelken's "tf" repo](https://github.com/nelken/tf) inspired our work by the way it implements features we are interested in. 
- [Benoit Favre's tf rnn lm](https://gitlab.lif.univ-mrs.fr/benoit.favre/tf_lm/blob/200645ab5aa446b72cf30c14355126062070f676/tf_lm.py)


## [Motivations](#motivations)
* Getting started with TensorFlow
* Make RNN LM manipulation easy in practice (easily look/edit configs, cancel/resume training, multiple outputs...)
* Train RNN LM for ASR using Kaldi (especially using `loglikes` mode)

## [Quickstart](#quickstart)

```shell
git clone https://github.com/pltrdy/tf_rnnlm
cd tf_rnnlm
./train.py --help
```

**Downloading PTB dataset:**
```shell
chmod +x tools/get_ptb.sh
./tools/get_ptb.sh
```

**Training small model:**
```shell
mkdir small_model
./train.py --data_path ./simple-examples/data --model_dir=./small_model --config small
```
**Training custom model:**
```shell
mkdir custom_model

# Generating new config file
chmod +x gen_config.py
./gen_config.py small custom_model/config

# Edit whatever you want
vi custom_model/config

# Train it. (it will automatically look for 'config' file in the model directory as no --config is set).
# It will look for 'train.txt', 'test.txt' and 'valid.txt' in --data_path
# These files must be present.
./train.py --data_path=./simple-examples/data --model_dir=./custom_model
```
**note:** data files are expected to be called `train.txt`, `test.txt` and `valid.txt`. Note that `get_ptb.sh` creates symlinks for that purpose

## Training all models and reporting results
```shell
./run.sh
```
Yes, that's all. It will train `small`, `medium` and `large` then generate a report like [this](results/template.md) using [report.sh](../tools/report.sh).   
**Feel free to share the report with us!**

## [Continue Training](#continue)
One can continue an interrupted training with the following command:
```shell
./train.py --data_path=./simple-examples/data --model_dir=./model
```
Where `./model` must contain `config`, `word_to_id`, `checkpoint` and the corresponding `.cktp` files.

## [Getting a text perplexity with regard to the LM](#test)
```shell
# Compute and outputs the perplexity of ./simple-examples/data/test.txt using LM in ./model
./test.py --data_path=./simple-examples/data --model_dir=./model
```

## [Line by line loglikes](#loglikes)
Running the model on each `stdin` line and returning its 'loglikes' (i.e. `-costs/log(10)`).

**Note:** in particular, it is meant to be used for Kaldi's rescoring.
```shell
cat ./data/test.txt | ./loglikes.py --model_dir=./model
```

## [Text generation](#generate)
Not documented yet
 

## [Results](#results)
Configuration `small` `medium` and `large` are defined in `config.py` and are the same as in [tensorflow.models.rnn.ptb.ptb_word_lm.py:200](https://github.com/tensorflow/tensorflow/blob/e2d51a87f0727f8537b46048d8241aeebb6e48d6/tensorflow/models/rnn/ptb/ptb_word_lm.py#L200)
### Baseline
Quoting [tensorflow.models.rnn.ptb.ptb_word_lm.py:22](https://github.com/tensorflow/tensorflow/blob/e2d51a87f0727f8537b46048d8241aeebb6e48d6/tensorflow/models/rnn/ptb/ptb_word_lm.py#L22):
> There are 3 supported model configurations:
> 
> | config | epochs | train | valid  | test  |
> |--------|--------|-------|--------|-------|
> | small  | 13     | 37.99 | 121.39 | 115.91|
> | medium | 39     | 48.45 |  86.16 |  82.07|
> | large  | 55     | 37.87 |  82.62 |  78.29|
> The exact results may vary depending on the random initialization.

### Our RNN LM working on sentences (see [docs/dynamic.md](docs/dynamic.md))
(based on commit [fff...f5d6](https://github.com/pltrdy/tf_rnnlm/commit/fff44942340b1881f1ebbb3be72044da41b8f5d6), training using `sampledsoftmax` loss function, `num_samples=1024` and `batch_size=64`; 1x GPU GTX1080)


| config | epochs | train | valid  | test  |  speed   | training_time | testing time |
|--------|--------|-------|--------|-------|----------|---------------|--------------|
| small  | 13     | 29.06 | 96.84  | 94.13 | ~36kWPS  |    6m21s      |     27s      |
| medium | 39     | 24.61 |  75.35 | 72.83 | ~11kWPS  |    59m7s      |     28s      |
| large  | 55     | 15.23 |  72.8  | 69.80 |  ~4kWPS  |    224m23s    |     1m5s     |

**kWPS:** processing speed, i.e. thousands word per seconds.    
**Reported time** are `real` times (see [What do 'real', 'user' and 'sys' mean in the output of time(1)?](http://stackoverflow.com/a/556411/5903959)   
**Testing** is done using softmax on transposed weights. ([docs/transpose.md](docs/dynamic.md))    
**For faster results** increasing `batch_size` should speed up the process, with a small perplexity increase as a side effect and an increased GPU Memory consumption. (which can fire Out Of Memory exception)

## Contributing
Please do!   
**Fork** the repo -> **edit** the code -> **commit** with descriptive commit message -> open a **pull request**   
You can also open an issue for any discussion about bugs, performance or results.   
Please also share your results with us! (see [sharing your results](docs/share_results.md))
