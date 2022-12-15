# Knowledge Distillation
This repository provides implementations of our original BERT distillation technique, [Distilling Task-Specific Knowledge from BERT into Simple Neural Networks](https://github.com/castorini/d-bert), and our more recent text generation-based method, [Natural Language Generation for Effective Knowledge Distillation](https://github.com/castorini/d-bert).
The latter is more effective than the former, albeit at the cost of computational efficiency, requiring multiple GPUs to fine-tune Transformer-XL or GPT-2 for constructing the transfer set.
Thus, we will henceforth refer to them as `d-lite` and `d-heavy`, respectively.

## Transfer Set Construction

Our first task is to construct a transfer set.
The two papers differ for this step only.

### Instructions for `d-lite`
1. Install the dependencies using `pip install -r requirements_init.txt`.

2. Build the transfer set by running `python -m dbert.distill.run.augment_data --dataset_file (the TSV dataset file) > (output file)` or `python -m dbert.distill.run.augment_paired_data --task (the task) --dataset_file (the TSV dataset file) > (output file)`.

These follow the GLUE datasets' formats.

### Instructions for `d-heavy`
1. Install the dependencies using `pip install -r requirements.txt`.

At the time of the experiments, [`transformers`](https://github.com/huggingface/transformers) was still `pytorch_pretrained_bert`, with no support for GPT-2 345M, so we had to add that manually.
We provide the configuration file in `confs/345m-config.json`.

2. Build a cache dataset using `python -m dbert.generate.cache_datasets --data-dir (directory) --output-file (cache file)`.

The data directory should contain `train.tsv`, `dev.tsv`, and `test.tsv`, as in [GLUE](https://gluebenchmark.com). For sentence-pair datasets, append `--dataset-type pair-sentence`.

3. Fine-tune Transformer-XL using `python -m dbert.generate.finetune_transfoxl --save (checkpoint file) --cache-file (the cache file) --train-batch-size (# that'll fit)`.

You can also use `generate.finetune_gpt` for fine-tuning GPT-2. In our paper, we used a batch size of 48, which might be too much for your system to handle. You can probably reduce it without much change in the final quality.

4. Build a prefix sampler for the transformer-dataset pair with `python -m dbert.generate.build_sampler --cache-file (the cache file) --save (the prefix sampler output file).

For GPT-2, add `--model-type gpt2`.

5. Sample from the Transformer-XL using `python -m dbert.generate.sample_transfoxl --prefix-file (the prefix sampler) > (output file)`.

For sentence-pair sampling, append `--paired`.