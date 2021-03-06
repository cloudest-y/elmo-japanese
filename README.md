# elmo-japanese
Tensorflow implementation of bidirectional language models (biLM) used to compute ELMo representations
from ["Deep contextualized word representations"](http://arxiv.org/abs/1802.05365).

This codebase is based on [bilm-tf](https://github.com/allenai/bilm-tf) and deals with Japanese.

This repository supports both training biLMs and using pre-trained models for prediction.


## Installation
- CPU
```
conda create -n elmo-jp python=3.6 anaconda
source activate elmo-jp
pip install tensorflow==1.10 h5py
git clone https://github.com/cl-tohoku/elmo-japanese.git
```
- GPU
```
conda create -n elmo-jp python=3.6 anaconda
source activate elmo-jp
pip install tensorflow-gpu==1.10 h5py
git clone https://github.com/cl-tohoku/elmo-japanese.git
```

## Getting started
- Training ELMo
```
python src/run_train.py \
    --option_file data/config.json \
    --save_dir checkpoint \
    --word_file data/vocab.sample.jp.wakati.txt \
    --char_file data/vocab.sample.jp.space.txt \
    --train_prefix data/sample.jp.wakati.txt
```

- Computing representations from the trained biLM

The following command outputs the ELMo representations (elmo.hdf5) for the text (sample.jp.wakati.txt) in the checkpoint directory (save_dir).

```
python src/run_elmo.py \
    --option_file checkpoint/options.json \
    --weight_file checkpoint/weight.hdf5 \
    --word_file data/vocab.sample.jp.wakati.txt \
    --char_file data/vocab.sample.jp.space.txt \
    --data_file data/sample.jp.wakati.txt \
    --output_file elmo.hdf5
```

The following command prints out the information of the elmo.hdf5, such as the number of sentences, words and dimensions.

```
python scripts/view_hdf5.py elmo.hdf5
```


## Computing sentence representations
- Save sentence-level ELMo representations
```
python src/run_elmo.py \
    --option_file checkpoint/options.json \
    --weight_file checkpoint/weight.hdf5 \
    --word_file data/vocab.sample.jp.wakati.txt \
    --char_file data/vocab.sample.jp.space.txt \
    --data_file data/sample.jp.wakati.txt \
    --output_file elmo.hdf5 \
    --sent_vec
```

- View sentence similarities
```
python scripts/view_sent_sim.py \
    --data data/sample.jp.wakati.txt \
    --elmo elmo.hdf5
```


## Training ELMo on a new corpus
- Making a token vocab file
```
python scripts/make_vocab_file.py \
    --input_fn data/sample.jp.wakati.txt \
    --output_fn data/vocab.sample.jp.wakati.txt
```

- Making a character vocab file
```
python scripts/space_split.py \
    --input_fn data/sample.jp.wakati.txt \
    --output_fn data/sample.jp.space.txt
```
```
python scripts/make_vocab.py \
    --input_fn data/sample.jp.space.txt \
    --output_fn data/vocab.sample.jp.space.txt
```

- Training ELMo
```
python src/run_train.py \\
    --train_prefix data/sample.jp.wakati.txt \
    --word_file data/vocab.sample.jp.wakati.txt \
    --char_file data/vocab.sample.jp.space.txt \
    --config_file data/config.json
    --save_dir checkpoint
```

- Retraining the trained ELMo
```
python src/run_train.py \
    --train_prefix data/sample.jp.wakati.txt \
    --word_file data/vocab.sample.jp.wakati.txt \
    --char_file data/vocab.sample.jp.space.txt \
    --save_dir checkpoint \
    --restart
```

- Computing token representations from the ELMo

```
python src/run_elmo.py \
    --test_prefix data/sample.jp.wakati.txt \
    --word_file data/vocab.sample.jp.wakati.txt \
    --char_file data/vocab.sample.jp.space.txt \
    --save_dir checkpoint
```


## Using the ELMo trained on Wikipedia
- Download: [checkpoint](https://drive.google.com/open?id=11tsu7cXV6KRS8aYnxoquEQ0xOp_i9mfa), [vocab tokens](https://drive.google.com/open?id=193JOeZcU6nSpGjJH9IP4Qn_UWiJXtRnp), [vocab characters](https://drive.google.com/open?id=15D8F3XRCm3oEdLBbl978KaJG_AAJDW4v)

- Computing sentence representations
```
python src/run_elmo.py \
    --option_file data/checkpoint_wiki-wakati-cleaned_token-10_epoch-10/options.json \
    --weight_file data/checkpoint_wiki-wakati-cleaned_token-10_epoch-10/weight.hdf5 \
    --word_file data/vocab.token.wiki_wakati.cleaned.min-10.txt \
    --char_file data/vocab.char.wiki_wakati.cleaned.min-0.txt \
    --data_file data/sample.jp.wakati.txt \
    --output_file elmo.hdf5 \
    --sent_vec
```

- Retraining the pre-trained ELMo on your corpus
```
python src/run_train.py \
    --train_prefix PATH_TO_YOUR_CORPUS \
    --word_file data/vocab.token.wiki_wakati.cleaned.min-10.txt \
    --char_file data/vocab.char.wiki_wakati.cleaned.min-0.txt \
    --save_dir checkpoint_wiki-wakati-cleaned_token-10_epoch-10 \
    --restart
```


## Checking performance in text classification
- Making a dataset for text classification
```
cd data
./make_data.sh
```

- Computing sentence representations
```
python src/run_elmo.py \
    --option_file data/checkpoint_wiki-wakati-cleaned_token-10_epoch-10/options.json \
    --weight_file data/checkpoint_wiki-wakati-cleaned_token-10_epoch-10/weight.hdf5 \
    --word_file data/vocab.token.wiki_wakati.cleaned.min-10.txt \
    --char_file data/vocab.char.wiki_wakati.cleaned.min-0.txt \
    --data_file data/dataset.wakati.txt \
    --output_file elmo.hdf5 \
    --sent_vec
```

- Predicting nearest neighbors
```
python src/knn.py \
    --data data/dataset.wakati-label.txt \
    --elmo elmo.hdf5
```

## LICENCE
MIT Licence