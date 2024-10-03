---
layout: single
title: "[MLB2024] - Gemma Sprint"
---

# Gemma Sprint Overview

As the final project of the Google Machine Learning Bootcamp 2024, Gemma Sprint leverages the Gemma LLM (Large Language Model).
This is the first Gemma Sprint organized by Google for the Developers Machine Learning Bootcamp, with the primary goal of fine-tuning the Gemma LLM model and uploading it to Hugging Face Hub.

## Challenges in Project Execution

I discovered that even if I have a solid idea and the techniques to fine-tune an LLM model, lacking a suitable dataset makes it challenging to achieve a performant fine-tuned model. Therefore, I explored various possible scenarios and narrowed down the cases based on three critical aspects.

### Possibility Analysis

| Idea | Technique | Data |
| ---- | --------- | ---- |
| X    | X         | X    |
| X    | X         | O    |
| X    | O         | X    |
| X    | O         | O    |
| O    | X         | X    |
| O    | X         | O    |
| O    | O         | X    |
| O    | O         | O    |



## Detailed Aspects

### Technical Aspect - Fine-Tuning Modeling: Types, Techiques, Pretraind Models

This involves exploring various types, techniques, and pretrained models. A deep understanding of fine-tuning methodologies is essential, as selecting the appropriate approach based on the characteristics of each model is crucial.

Types
- Sentiment Classification
- Question-Anwsering
- Text Summarization
- Translation
- Creative Writing

Techniques
- unsloth
- alpaca
- RoLA, QRoLA
- SFT, RLHF, CPT

Pretrain Models
- Gemma2 2B, 9B

### Idea Aspect - Selecting Ideas: Fields

I brainstormed ideas applicable across various fields. There are 5 main fields where we can leverage LLMs more effectively.

I narrowed down to 3 cases each with specified use cases and benefits under the goal of each idea needing to be feasible and leverage the strengths of the LLM model effectively.

#### Potential Fields of Application

| | Use Cases | Benefits
|---|---|---|
|Finance|- Financial documentation analysis<br/>- Sentiment Analysis of market trends<br/>- Personalized investment advice|- Enhance data analysis capabilities<br/>- Provide real-time insights<br/>- Streamline process<br/>- Improved decision-making
|Legal |- Document review and analysis<br/>- Contract Generation<br/>- Legal Research Assistance|- Save time on tedious tasks<br/>- Improve accuracy in legal documentation<br/>- Easy and quickly to find relevant case law or statutes|
| Medicine| - Patient data analysis<br/>- Medical disease prediction<br/>- Chatbot assistance for patient inqueries|- Provide quick access to medical information<br/>- Improve patient engagement through chatbot<br/>- Support clinical decision-making with data-driven insights|
| Commerce |- Customer service automation<br/>- Product recommendation system<br/>- Market trend analysis|- Chatbot interacting with customers<br/>- Personlized shopping experience<br/>- Analyze customer behavior for maketing strategies|
| Publilc & Academic Sector |- Question-answering<br/>- Citizen engagement platforms: finding Korean Restaurants in Paris<br/>- Automated report generation|- Facilitate citizen inquiries<br/>- streamline writing process<br/>- Foster better communication between with users|


By focusing on these fields, I explored innovative solutions that not only utilize the capabilities of Gemma LLM but also address the real-world challenges.

Each idea is evaluated for feasibility and potential impact, ensuring the final applications are both practical and benefitial to users.


### Data Aspect - Suitable Datasets

Identifying datasets that are appropriate for specific subjects and fine-tuning techniques is vital. This includes considerations of data types, collection methods, formats, preprocessing, and transformation. Recognizing that acquiring the right dataset directly impacts model performance was a key insight.


Types: Structured, unstructured, sequential, serial

Collection
- [AIHUB (kr)](https://www.aihub.or.kr/)
- Hugging Face datasets
- Kaggle datasets

Format
- json

```json
{
	"Description": "A brief description of the item.",
	"Atmosphere": "The ambiance or mood of the place.",
	"Location": "The geographical area or address.",
	"Price Range": "The cost range of the item.",
	"Rating": "User ratings for the item.",
	"Main Menu": "The main offerings available."
}
```
- Hugging Face datasets: Input-output mapping

```json
{
	"text":""
	"subclass":""
}
```


# Sprint Review

## Part0 - Elaborating Chosen Idea
For this sprint, I focused on classification of mutation cancer types, a topic I have been actively working on. It is to classify a certain cancer type based on a patient gene mutation status.
Given the complexity and the vast amount of data associated with cancer mutations, I recognize that I could leverage capabilities of Gemma LLM to enhance the classification process.

## Part1 - Preparing Custom Dataset

The dataset I initialiy had was insufficient for fine-tuning, and due to its high dimensionality, it presented challenges in precessing.

So, I performed the following steps:

- Collecting additional external data (e.g., Depmap data) to complement the existing dataset and  transforming the dataset into text format that LLM can easily understand. [Colab - depmap data](https://colab.research.google.com/drive/15hzj7VQtIqETbiKUy7ejOtLfrxAPwu5Q#scrollTo=b3vpmJ9Z7VK8)

- This external data was preprocessed to ensure it conformed to the required format for the model. [Colab - dataset mapping](https://colab.research.google.com/drive/1ZhTkWjcAmQbQinMj19r1-hxQ6HBHCyNm#scrollTo=eVRmp4GS4li9)

- After transforming and preprocessing the datasets, I uploaded them to Hugging Face Hub

## Part2 - Fine-tune Gemma Model

### Model Configuration
With the custom datasets prepared, I proceeded to configure the model.
Here, I employed the `unsloth` library with `PEFT` method to make faster the fine-tuning process.

```python
from unsloth import FastLanguageModel
import torch

max_seq_length = 2048
dtype = torch.float16 
load_in_4bit = True   # Use 4bit quantization to reduce memory usage.

model_name = "unsloth/gemma-2-2b-bnb-4bit",
# model_name = "jmcheon/mutation_cancer_classification"

model, tokenizer = FastLanguageModel.from_pretrained(
    model_name = model_name,
    max_seq_length = max_seq_length,
    dtype = dtype,
    load_in_4bit = load_in_4bit,
)
```

The fine-tuning process has been done in `Colab` with `one 15G of GPU T4`, with this constraint, I leveraged `RoLA` technique to reduce the training time and computational resources.

```python
model = FastLanguageModel.get_peft_model(
    model,
    r = 16,
    target_modules = ["q_proj", "k_proj", "v_proj", "o_proj",
                      "gate_proj", "up_proj", "down_proj","embed_tokens",],
    lora_alpha = 32,
    lora_dropout = 0,
    bias = "none",
    use_gradient_checkpointing = "unsloth", # "unsloth" for very long context
    random_state = 3407,
    use_rslora = True,       # Rank stabilized LoRA
    loftq_config = None,
)
```

I ranged the `max_steps` parameter, which is for epoch, to check the training process and continued fine-tuning using the same model.

```python
from transformers import TrainingArguments
from unsloth import is_bfloat16_supported
from unsloth import UnslothTrainer, UnslothTrainingArguments

trainer = UnslothTrainer(
    model = model,
    tokenizer = tokenizer,
    train_dataset = dataset,
    dataset_text_field = "text",
    max_seq_length = max_seq_length,
    dataset_num_proc = 8,
    packing=False,

    args = UnslothTrainingArguments(
        per_device_train_batch_size = 2,
        gradient_accumulation_steps = 8,
        max_steps = 120,         # epoch

        # Select a 2 to 10x smaller learning rate for the embedding matrices!
        learning_rate = 5e-5,           # fine tuning learning rate
        embedding_learning_rate = 1e-5,

        fp16 = True, #  not is_bfloat16_supported()
        bf16 = is_bfloat16_supported(),
        logging_steps = 1,
        optim = "adamw_8bit",
        weight_decay = 0.00,
        lr_scheduler_type = "linear",
        seed = 3407,
        output_dir = "outputs",
    ),
)
```

### Prompt
```python
prompt = """This is mutation status infomation of a patient in some genes and its corresponding cancer type, 
 classify the paitent to a type of cancer based on the mutations.

 only 26 cancer types valid: ["KIPAN", "SARC", "SKCM", "KIRC", "GBMLGG", "STES", "BRCA", "THCA", "LIHC", "HNSC", "PAAD", "OV", "PRAD", "UCEC", "LAML", "COAD", "ACC" , "LGG" , "LUSC", "LUAD", "CESC", "PCPG", "THYM", "BLCA", "TGCT", "DLBC"]

### mutations:
{}

### SUBCLASS:
{}"""
```

## Part3 - Deploying fine-tuned model to Hugging Face Hub

After successfully fine-tuning the Gemma LLM, I uploaded the final model to Hugging Face Hub.

```python
base_model = "unsloth/gemma-2-2b-bnb-4bit"
huggingface_repo = "jmcheon/mutation_cancer_classification"
save_method = (
    "merged_16bit"
)
```

I had an issue with the uploaded tokenizer when loading it, so I had to upload it separately.

```python
model.push_to_hub_merged(
    huggingface_repo,
    save_method=save_method,
    token=hf_token
)

tokenizer.push_to_hub("jmcheon/mutation_cancer_classification")
```

## Limitations
- datasets
- GPU

# References
- [효율 최강 파인튜닝 솔루션 Unsloth (ft. Continued Pre-Training) (kr)](https://devocean.sk.com/blog/techBoardDetail.do?ID=166285&boardType=techBlog)
- [LoRA 개념을 쉽게 설명해드립니다 (생성형 AI, LLM, 딥러닝) (kr)](https://www.youtube.com/watch?v=0lf3CUlUQtA&list=LL&index=2)
- [depmap data (en)](https://depmap.org/portal/data_page/?tab=overview)
- [oncotree (en)](https://oncotree.mskcc.org/#/home)

# Remaining Tasks
- app
- continuous training
- collecting more datasets
- apply other techniques
- other ideas into applications


[#GemmaSprint](https://www.google.com/search?q=google+gemma+sprint&sca_esv=1c128b9a547ab16f&sxsrf=ADLYWIIHQBhBOdUb74DUUiPmZb1IF1IIfw%3A1727989963338&ei=ywj_ZpCyFKXhkdUPxp_E6QY&ved=0ahUKEwjQw8KfkPOIAxWlcKQEHcYPMW0Q4dUDCA8&uact=5&oq=google+gemma+sprint&gs_lp=Egxnd3Mtd2l6LXNlcnAaAhgCIhNnb29nbGUgZ2VtbWEgc3ByaW50MgUQIRigATIFECEYoAFIlh1QrAdYgRxwA3gBkAEAmAGrAqABnRGqAQYxMi4zLjS4AQPIAQD4AQGYAhagAvoSwgIKEAAYsAMY1gQYR8ICBBAjGCfCAgoQIxiABBgnGIoFwgIOEAAYgAQYkQIYigUYiwPCAhMQLhiABBhDGKgDGIoFGJoDGIsDwgIKEC4YgAQYQxiKBcICEBAuGIAEGNEDGEMYxwEYigXCAgoQABiABBhDGIoFwgINEAAYgAQYQxiKBRiLA8ICEBAAGIAEGEMYyQMYigUYiwPCAhkQLhiABBjRAxjSAxhDGMcBGKgDGIoFGIsDwgILEAAYgAQYkQIYigXCAgUQABiABMICBxAAGIAEGArCAgoQABiABBgUGIcCwgIGEAAYFhgewgIIEAAYFhgKGB7CAgsQABiABBiGAxiKBcICCBAAGIAEGKIEwgIEECEYFcICBRAhGJ8FwgIHECEYoAEYCpgDAIgGAZAGCJIHCDE0LjQuMi4yoAfjogE&sclient=gws-wiz-serp) [#Gemma](https://ai.google.dev/gemma)