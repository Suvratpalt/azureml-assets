# Description
Falcon-40B is a large language model (LLM) developed by the Technology Innovation Institute (TII) with 40 billion parameters. It is a causal decoder-only model trained on 1 trillion tokens from the RefinedWeb dataset, enhanced with curated corpora. Falcon-40B supports English, German, Spanish, and French languages, with limited capabilities in Italian, Portuguese, Polish, Dutch, Romanian, Czech, and Swedish. It is available under the Apache 2.0 license.

Falcon-40B is considered the best open-source model currently available, optimized for inference with features such as FlashAttention and multiquery. However, it is recommended to fine-tune the model for specific use cases.

The training of Falcon-40B involved using 384 A100 40GB GPUs and took two months. The model carries biases and stereotypes encountered online and requires appropriate precautions for production use. It is suggested to finetune the model for specific tasks and consider guardrails. The technical specifications, training details, and evaluation results are provided in the summary.

> The above summary was generated using ChatGPT. Review the <a href="https://huggingface.co/tiiuae/falcon-40b" target="_blank">original model card</a> to understand the data used to train the model, evaluation metrics, license, intended uses, limitations and bias before using the model.

## Training Details

### Training Data

Falcon-40B was trained on 1,000B tokens of [RefinedWeb](https://huggingface.co/datasets/tiiuae/falcon-refinedweb), a high-quality filtered and deduplicated web dataset which we enhanced with curated corpora. Significant components from our curated copora were inspired by The Pile ([Gao et al., 2020](https://arxiv.org/abs/2101.00027)). 

| **Data source**    | **Fraction** | **Tokens** | **Sources**                       |
|--------------------|--------------|------------|-----------------------------------|
| [RefinedWeb-English](https://huggingface.co/datasets/tiiuae/falcon-refinedweb) | 75%          | 750B     | massive web crawl                 |
| RefinedWeb-Europe              | 7%           | 70B       | European massive web crawl                                   |
| Books  | 6%           | 60B        |                  |
| Conversations      | 5%           | 50B        | Reddit, StackOverflow, HackerNews |
| Code               | 5%           | 50B        |                                   |
| Technical          | 2%           | 20B        | arXiv, PubMed, USPTO, etc.        |

RefinedWeb-Europe is made of the following languages:

| **Language** | **Fraction of multilingual data** | **Tokens** |
|--------------|-----------------------------------|------------|
| German       | 26%                               | 18B        |
| Spanish      | 24%                               | 17B        |
| French       | 23%                               | 16B        |
| _Italian_    | 7%                                | 5B         |
| _Portuguese_ | 4%                                | 3B         |
| _Polish_     | 4%                                | 3B         |
| _Dutch_      | 4%                                | 3B         |
| _Romanian_   | 3%                                | 2B         |
| _Czech_      | 3%                                | 2B         |
| _Swedish_    | 2%                                | 1B         |


The data was tokenized with the Falcon-[7B](https://huggingface.co/tiiuae/falcon-7b)/[40B](https://huggingface.co/tiiuae/falcon-40b) tokenizer.

### Training Procedure 

Falcon-40B was trained on 384 A100 40GB GPUs, using a 3D parallelism strategy (TP=8, PP=4, DP=12) combined with ZeRO.

#### Training Hyperparameters

| **Hyperparameter** | **Value**  | **Comment**                               |
|--------------------|------------|-------------------------------------------|
| Precision          | `bfloat16` |                                           |
| Optimizer          | AdamW      |                                           |
| Learning rate      | 1.85e-4       | 4B tokens warm-up, cosine decay to 1.85e-5 |
| Weight decay       | 1e-1       |                                           |
| Z-loss       | 1e-4       |                                           |
| Batch size         | 1152        | 100B tokens ramp-up                         |


#### Speeds, Sizes, Times

Training started in December 2022 and took two months. 


## Evaluation

*Paper coming soon.*

See the [OpenLLM Leaderboard](https://huggingface.co/spaces/HuggingFaceH4/open_llm_leaderboard) for early results.


## Technical Specifications 

### Model Architecture and Objective

Falcon-40B is a causal decoder-only model trained on a causal language modeling task (i.e., predict the next token).

The architecture is broadly adapted from the GPT-3 paper ([Brown et al., 2020](https://arxiv.org/abs/2005.14165)), with the following differences:

* **Positionnal embeddings:** rotary ([Su et al., 2021](https://arxiv.org/abs/2104.09864));
* **Attention:** multiquery ([Shazeer et al., 2019](https://arxiv.org/abs/1911.02150)) and FlashAttention ([Dao et al., 2022](https://arxiv.org/abs/2205.14135));
* **Decoder-block:** parallel attention/MLP with a two layer norms.

For multiquery, we are using an internal variant which uses independent key and values per tensor parallel degree.

| **Hyperparameter** | **Value** | **Comment**                            |
|--------------------|-----------|----------------------------------------|
| Layers             | 60        |                                        |
| `d_model`          | 8192      |                                        |
| `head_dim`         | 64        | Reduced to optimise for FlashAttention |
| Vocabulary         | 65024     |                                        |
| Sequence length    | 2048      |                                        |

### Compute Infrastructure

#### Hardware

Falcon-40B was trained on AWS SageMaker, on 384 A100 40GB GPUs in P4d instances. 

#### Software

Falcon-40B was trained a custom distributed training codebase, Gigatron. It uses a 3D parallelism approach combined with ZeRO and high-performance Triton kernels (FlashAttention, etc.)

#### License

Falcon-40B is made available under the Apache 2.0 license.

# Inference samples

Inference type|Python sample (Notebook)|CLI with YAML
|--|--|--|
Real time|<a href="https://aka.ms/azureml-infer-online-sdk-text-generation-dolly" target="_blank">text-generation-online-endpoint-dolly.ipynb</a>|<a href="https://aka.ms/azureml-infer-online-cli-text-generation-dolly" target="_blank">text-generation-online-endpoint-dolly.sh</a>
Batch |<a href="https://aka.ms/azureml-infer-batch-sdk-text-generation" target="_blank">text-generation-batch-endpoint.ipynb</a>| coming soon

### Model Evaluation Sample

Task| Use case| Dataset| Python sample (Notebook)| CLI with YAML
|--|--|--|--|--|
Text generation | Text generation | <a href="https://huggingface.co/datasets/cnn_dailymail" target="_blank"> cnn_dailymail </a> | <a href="https://aka.ms/azureml-eval-sdk-text-generation/" target="_blank">evaluate-model-text-generation.ipynb</a> | <a href="https://aka.ms/azureml-eval-cli-text-generation/" target="_blank">evaluate-model-text-generation.yml</a>

## Sample input (for real-time inference)

```json
{
  "input_data": {
      "input_string":["The meaning of the life is"]
  }
}
```

## Sample output
```json
[
  {
    "0": "The meaning of the life is to find your gift. The purpose of life is to give it away"
  }
]
```
