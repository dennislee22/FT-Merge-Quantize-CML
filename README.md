LLM: Fine-Tune > Merge > Quantize > Infer .. on CML
===

## <a name="toc_0"></a>Table of Contents
[//]: # (TOC)
[1. Objective](#toc_0)<br>
[2. Summary & Benchmark Score](#toc_2)<br>
[3. Preparation](#toc_3)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[3.1. Dataset & Model](#toc_4)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[3.2. CML Session](#toc_5)<br>
[4. bigscience/bloom-1b1](#toc_6)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[4.1. Fine-Tune (w/o Quantization) > Merge > Inference](#toc_7)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[4.2. Quantize > Inference](#toc_8)<br>
[5. bigscience/bloom-7b1](#toc_9)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[5.1. Fine-Tune (w/o Quantization) > Merge > Inference](#toc_10)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[5.1. Fine-Tune (8-bit) > Merge > Inference](#toc_11)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[5.2. Quantize > Inference](#toc_12)<br>
[6. tiiuae/falcon-1b](#toc_13)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[6.1. Fine-Tune (w/o Quantization) > Merge > Inference](#toc_14)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[6.2. Quantize > Inference](#toc_15)<br>
[7. tiiuae/falcon-7b](#toc_16)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[7.1. Fine-Tune (w/o Quantization) > Merge > Inference](#toc_17)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[7.1. Fine-Tune (8-bit) > Merge > Inference](#toc_18)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[7.2. Quantize > Inference](#toc_19)<br>
[7. Salesforce/codegen2-1B](#toc_20)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[7.1. Fine-Tune & Merge](#toc_21)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[7.2. Quantize](#toc_22)<br>
&nbsp;&nbsp;&nbsp;&nbsp;[7.3. Inference](#toc_23)<br>

[//]: # (/TOC)

### <a name="toc_0"></a>1. Objective

- In the event that you have limited GPU resources or even have no GPU in your infrastructure landscape, you may run your GenAI application using quantized models. This articles focuses on how to quantize your language models in 8, 4, or even 2 bits without **significant** performance degradation and quicker inference speed, with the help of Transformers API.
GPTQ, a Post-Training Quantization (PTQ) technique.
- GPTQ adopts a mixed int4/fp16 quantization scheme where weights are quantized as int4 while activations remain in float16. During inference, weights are dequantized on the fly and the actual compute is performed in float16.
- bitsandbytes (zero-shot quantization)
- To comprehend this, it’s crucial to realize that during model training, the model states are the main contributors to memory usage. These include tensors composed of optimizer states, gradients, and parameters. In addition to these model states, there are activations, temporary buffers, and fragmented memory, collectively known as residual states, that consume the remaining memory.
- The latest way to train big models using the newest NVIDIA graphics cards uses a method known as mixed-precision (FP16/32) training. FP32 is called full precision (4 bytes), while FP16 are referred to as half-precision (2 bytes). Here, important model components like parameters and activations are stored as FP16. This storage method allows these graphics cards to process large amounts of data very quickly.
- During this training process, both the forward and backward steps are done using FP16 weights and activations. However, to properly calculate and apply the updates at the end of the backward step, the mixed-precision optimizer keeps an FP32 copy of the parameters and all other states used in the optimizer.



#### <a name="toc_2"></a>2. Summary & Benchmark Score

- Table shows the benchmark result of fine-tuning the specific base model with **Text-to-SQL** dataset.
  
| Model | Training | Duration | 
| :---      |     :---:      |   ---: |
| bloom-1b  | No quantization     | sec   |
| bloom-1b  | BitsAndBytes      | sec     |

- Quantization: A quick check at the Open LLM Leaderboard reveals that performance degradation is quite minimal.
  
### <a name="toc_3"></a>3. Preparation

#### <a name="toc_4"></a>3.1 Dataset & Model

- Download or use the following the following model directly from 🤗.<br> 
&nbsp;a. `bigscience/bloom-1b1`<br>
&nbsp;b. `tiiuae/falcon-7b`<br>
&nbsp;c. `Salesforce/codegen2-1B`<br>

- Download or use the following sample dataset directly from 🤗. <br> 
&nbsp;a. Dataset for fine-tuning: <br> 
&nbsp;b. Dataset for quantization: Quantization requires sample data to calibrate and enhance quality of the quantization. In this benchmark test, [C4 dataset](https://huggingface.co/datasets/c4) is utilized. C4 is a large-scale, multilingual collection of web text gathered from the Common Crawl project. <br> 

#### <a name="toc_5"></a>3.2 CML Session

- CML
1. Create a CML project using Python 3.9 with Nvidia GPU runtime.
2. Create a CML session (Jupyter) with the resource profile of 4CPU and 64GB memory and 1GPU.
3. In the CML session, install the necessary Python packages.
```
pip install -r requirements.txt
```

### <a name="toc_6"></a>4. `bigscience/bloom-1b1`

#### <a name="toc_7"></a>4.1. Fine-Tune (w/o Quantization) > Merge > Inference

- Use this Jupyter code to fine-tune, merge and perform a simple inference on the merged model.
  
- Code Snippet:
```
base_model = AutoModelForCausalLM.from_pretrained(base_model, use_cache = False, device_map=device_map)
```

- Load model before fine-tuning/training starts:
```
Base Model Memory Footprint in VRAM: 4063.8516 MB
--------------------------------------
Parameters loaded for model bloom-1b1:
Total parameters: 1065.3143 M
Trainable parameters: 1065.3143 M


Data types for loaded model bloom-1b1:
torch.float32, 1065.3143 M, 100.00 %
```

<img width="973" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/4e557656-abb9-409f-8a56-23601af785f9"><br>

- During fine-tuning/training:
<img width="974" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/d1a594f6-c284-4cb7-bda5-17d19227626d">

- It takes ~12mins to complete the training.
```
{'loss': 0.8376, 'learning_rate': 0.0001936370577755154, 'epoch': 2.03}
{'loss': 0.7142, 'learning_rate': 0.0001935522185458556, 'epoch': 2.03}
{'loss': 0.6476, 'learning_rate': 0.00019346737931619584, 'epoch': 2.03}
{'train_runtime': 715.2236, 'train_samples_per_second': 32.96, 'train_steps_per_second': 16.48, 'train_loss': 0.8183029612163445, 'epoch': 2.03}
Training Done
```

- After the training is completed, merge the base model with the PEFT-trained adapters.
- Load the merged model:
```
Merged Model Memory Footprint in VRAM: 4063.8516 MB

Data types:
torch.float32, 1065.3143 M, 100.00 %
```
<img width="973" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/021a1854-f943-4257-9165-f90bde98c5e8"><br>

- Run inference on the fine-tuned/merged model and the base model:
```
--------------------------------------
Prompt:
# Instruction:
Use the context below to produce the result
# context:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is not Dennis Lee?
# result:

--------------------------------------
Fine-tuned Model Result :
SELECT Title FROM book WHERE Writer <> 'Dennis Lee'
```

```
Base Model Result :
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is Dennis Lee?
# result:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is not Dennis Lee?
# result:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is Dennis Lee?
```

#### <a name="toc_8"></a>4.2. Quantize > Inference
- During quantization:
<img width="1059" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/414dca58-025a-48b2-93e4-816b5781e0ce">

<img width="974" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/218470a5-4358-41ce-8661-0dc8b21bf224"><br>

- Time taken to quantize:
```
Total Seconds Taken to Quantize Using cuda:0: 282.6761214733124
```

- Loaded quantized model:
```
cuda:0 Memory Footprint: 1400.0977 MB

Data types:
torch.float16, 385.5053 M, 100.00 %

```
<img width="975" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/75965ac1-81ce-4c5e-8aca-83246cf674ab"><br>

- Infer quantized model:
```
--------------------------------------
Prompt:
# Instruction:
Use the context below to produce the result
# context:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is not Dennis Lee?
# result:

--------------------------------------
Quantized Model Result :
SELECT Title FROM book WHERE Writer = 'Not Dennis Lee'
```

### <a name="toc_16"></a>7. `tiiuae/falcon-7b`

#### <a name="toc_17"></a>7.1. Fine-Tune (wo Quantization) > Merge > Inference

- Code Snippet:
```
base_model = AutoModelForCausalLM.from_pretrained(base_model, use_cache = False, device_map=device_map)
```

- Load model before fine-tuning/training starts:
```
Base Model Memory Footprint in VRAM: 26404.2729 MB
--------------------------------------
Parameters loaded for model falcon-7b:
Total parameters: 6921.7207 M
Trainable parameters: 6921.7207 M


Data types for loaded model falcon-7b:
torch.float32, 6921.7207 M, 100.00 %
```
<img width="973" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/dd499a83-f7b9-41d3-8c59-955e9e16a0fc"><br>

- During fine-tuning/training:

```
OutOfMemoryError: CUDA out of memory. Tried to allocate 1.11 GiB. GPU 0 has a total capacty of 39.39 GiB of which 345.94 MiB is free. Process 1618370 has 39.04 GiB memory in use. Of the allocated memory 37.50 GiB is allocated by PyTorch, and 1.05 GiB is reserved by PyTorch but unallocated. If reserved but unallocated memory is large try setting max_split_size_mb to avoid fragmentation.  See documentation for Memory Management and PYTORCH_CUDA_ALLOC_CONF
```
<img width="973" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/0e91da7b-f704-4b03-a824-b5391819a6c8"><br>

#### <a name="toc_18"></a>7.2. Fine-Tune (w 8-bit Quantization) > Merge > Inference

- Code Snippet:
```
bnb_config = BitsAndBytesConfig(
    load_in_8bit=True,
)
base_model = AutoModelForCausalLM.from_pretrained(base_model, quantization_config=bnb_config, use_cache = False, device_map=device_map)
```

- Load model before fine-tuning/training starts:
```
Base Model Memory Footprint in VRAM: 6883.1384 MB
--------------------------------------
Parameters loaded for model falcon-7b:
Total parameters: 6921.7207 M
Trainable parameters: 295.7690 M


Data types for loaded model falcon-7b:
torch.float16, 295.7690 M, 4.27 %
torch.int8, 6625.9517 M, 95.73 %
```
<img width="974" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/a228e9ef-6e4d-438b-8b3a-53d74f7d127b"><br>

- During fine-tuning/training:
<img width="975" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/6a743e35-672f-4163-916b-0b491d88bf42">

```
warnings.warn(f"MatMul8bitLt: inputs will be cast from {A.dtype} to float16 during quantization")
```

- Time taken to quantize:
```

```

- After training is completed, merge the base model with the PEFT-trained adapters.
  
- Load the merged model:
```
Merged Model Memory Footprint in VRAM: 4063.8516 MB

Data types:
torch.float32, 1065.3143 M, 100.00 %
```
<img width="973" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-Infer-CML/assets/35444414/021a1854-f943-4257-9165-f90bde98c5e8">

- Run inference on the fine-tuned/merged model and the base model:
```
--------------------------------------
Prompt:
# Instruction:
Use the context below to produce the result
# context:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is not Dennis Lee?
# result:

--------------------------------------
Fine-tuned Model Result :
SELECT Title FROM book WHERE Writer <> 'Dennis Lee'
```

```
Base Model Result :
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is Dennis Lee?
# result:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is not Dennis Lee?
# result:
CREATE TABLE book (Title VARCHAR, Writer VARCHAR). What are the titles of the books whose writer is Dennis Lee?
```

#### <a name="toc_19"></a>7.3. Quantize > Inference
- During quantization:

- Time taken to quantize:
```

```

- Loaded quantized model:
```

```

- Infer quantized model:
```

```










#### Notes

- During quantization process:

<img width="1056" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-CML/assets/35444414/a7935a5b-3b3d-419b-8257-8635f829e4e9">

<img width="1023" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-CML/assets/35444414/e07355d0-6f08-4fe0-a708-40380c1323cd">

- When exllama is enabled, 'Assertion error:`
- Disabling exllama allowing the quantization process to complete. Notice that CPU is also being used:

<img width="1022" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-CML/assets/35444414/127c3e28-b194-407d-acef-f7f3a75b70ce">

<img width="900" alt="image" src="https://github.com/dennislee22/FT-Merge-Quantize-CML/assets/35444414/daf4c454-4689-4e5f-99e4-5dfb5216ebfa">

```
Total Seconds Taken to Quantize Using cuda:0: 1350.0081555843353
```

```
ls -lh gptq-merged_falcon-7b_4bit
total 3.8G
-rw-r--r--. 1 cdsw cdsw 1.7K Nov  1 05:42 config.json
-rw-r--r--. 1 cdsw cdsw  118 Nov  1 05:42 generation_config.json
-rw-r--r--. 1 cdsw cdsw 3.8G Nov  1 05:42 pytorch_model.bin
-rw-r--r--. 1 cdsw cdsw  541 Nov  1 05:42 special_tokens_map.json
-rw-r--r--. 1 cdsw cdsw 2.6K Nov  1 05:42 tokenizer_config.json
-rw-r--r--. 1 cdsw cdsw 2.7M Nov  1 05:42 tokenizer.json
```

8-bit Parameter Precision Info:
```
cuda:0 Memory Footprint: 7038.3259 MB
Total parameters: 295.7690 M
Trainable parameters: 295.7690 M

Data types:
torch.float16, 295.7690 M, 100.00 %
```
