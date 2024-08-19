---
layout: post
title: "Sizing an LLM: Parameters"
author: "Carlos Camacho"
categories:
  - blog
tags:
  # - draft
  - cloud
  - kubernetes
  - engineering
  - openshift
# hidden: true
favorite: true
commentIssueId: 94
refimage: '/static/llm/sizing_parameters.png'
---


Large language models (LLMs) like GPT-3 and GPT-4 have become a cornerstone of
modern natural language processing (NLP) applications, driving advancements in
machine translation, text generation, question answering, and more.
These models are powered by neural networks with billions of parameters,
making them incredibly powerful but also resource-intensive.

# Understanding Large Language Models: Parameters, Calculations, and VRAM Requirements

In this blog post, we'll explore what parameters in large language models
are, how they are calculated, and the implications of these calculations
on the space required to load a model into VRAM (Video RAM).

## What Are Large Language Models?

Large language models are deep learning models designed to understand and generate human language. They are typically based on transformer architectures, which allow the model to handle complex language tasks by capturing contextual information from input text sequences.

### Parameters: The Building Blocks of LLMs

In the context of neural networks, parameters refer to the weights and biases within the model. These are the elements that the model adjusts during training to minimize the difference between its predictions and the actual outcomes (i.e., to reduce the loss function). The parameters are what the model "learns" and are essential for the model's ability to generalize from training data to unseen text.

The number of parameters in a model is a key factor in its complexity and capacity. For example, GPT-3 has 175 billion parameters, while earlier versions like GPT-2 had 1.5 billion parameters. The sheer scale of these models means they can capture a wide range of linguistic patterns, making them versatile across different NLP tasks.

## How Are Parameters Calculated?

Calculating the parameters of a language model involves understanding the architecture of the model. For transformer-based models, the parameters are primarily located in:

1. **Embedding Layers**: These layers convert input tokens (words, subwords, or characters) into vectors of continuous numbers. The parameters here are the weights used to create these embeddings.

2. **Attention Mechanisms**: Transformers rely heavily on attention mechanisms, particularly self-attention, to determine the importance of each word in a sequence relative to others. The parameters in these mechanisms include weights that govern the interactions between different tokens.

3. **Feedforward Layers**: After the attention mechanism processes the input, the output is passed through feedforward neural networks, which consist of fully connected layers. Each of these layers has its own set of parameters (weights and biases).

4. **Output Layers**: Finally, the processed data is passed through the output layers, which convert the model's internal representation back into a format that can be interpreted (e.g., predicting the next word in a sequence).

### Example Calculation

Let's consider a simplified transformer model with the following characteristics:

- **Embedding Size**: 512
- **Number of Layers**: 12
- **Attention Heads**: 8
- **Vocabulary Size**: 50,000

The total number of parameters can be broken down as follows:

1. **Embedding Layer**: 
   - The embedding layer would have `Vocabulary Size * Embedding Size` parameters.
   - Example: \(50,000 \times 512 = 25,600,000\) parameters.

2. **Attention Mechanism**:
   - Each attention head has `Embedding Size / Attention Heads` parameters for queries, keys, and values.
   - Example: If we have 8 heads, each head would have \(512 / 8 = 64\) parameters per token.
   - For the entire model: \(12 \times 8 \times (Embedding Size \times Embedding Size / Attention Heads) = 12 \times 8 \times (512 \times 64) = 3,932,160\) parameters.

3. **Feedforward Layers**:
   - Each layer typically has `2 * Embedding Size^2` parameters.
   - Example: \(12 \times 2 \times 512^2 = 6,291,456\) parameters.

4. **Output Layers**:
   - The output layer usually has `Embedding Size * Vocabulary Size` parameters.
   - Example: \(512 \times 50,000 = 25,600,000\) parameters.

Adding these up, the total number of parameters for this simplified model would be approximately:

\[
25,600,000 + 3,932,160 + 6,291,456 + 25,600,000 = 61,423,616 \text{ parameters}
\]

## VRAM Requirements: How Much Space Do LLMs Need?

The enormous number of parameters in large language models poses significant challenges for deployment, especially concerning memory requirements. VRAM, which is specialized memory used by GPUs (Graphics Processing Units), plays a critical role in running these models efficiently.

### Parameter Storage

Each parameter in a model is typically stored as a floating-point number, usually in 16-bit (FP16), 32-bit (FP32), or in some cases, 8-bit (INT8) precision. The choice of precision affects both the VRAM requirements and the performance of the model. 

- **FP32 (32-bit precision)**: Each parameter requires 4 bytes.
- **FP16 (16-bit precision)**: Each parameter requires 2 bytes.
- **INT8 (8-bit precision)**: Each parameter requires 1 byte.

Using our previous example of a model with approximately 61 million parameters:

- **FP32**: \(61,423,616 \times 4 = 245,694,464\) bytes or roughly 234 MB of VRAM.
- **FP16**: \(61,423,616 \times 2 = 122,847,232\) bytes or roughly 117 MB of VRAM.
- **INT8**: \(61,423,616 \times 1 = 61,423,616\) bytes or roughly 58.6 MB of VRAM.

### Additional Memory Requirements

Besides storing the parameters, VRAM is also needed for:

1. **Activations**: Intermediate activations during the forward and backward passes require additional memory. The size of activations can be substantial, especially for deep networks with many layers.

2. **Optimizer States**: When training a model, optimizers like Adam maintain additional states for each parameter, such as momentums and velocities, which also consume VRAM.

3. **Batch Size**: The size of the input batch processed by the model in one go also affects VRAM usage. Larger batch sizes require more memory for both the input data and the activations.

### Estimating VRAM Requirements

To estimate the total VRAM required, you can use the following rule of thumb:

\[
\text{Total VRAM} \approx \text{Model Parameters} + \text{Activations} + \text{Optimizer States} + \text{Input Data}
\]

For a model with 61 million parameters using FP32 precision, a small batch size, and moderate complexity, you might need:

- **Model Parameters**: 234 MB (from above)
- **Activations**: 200 MB (estimated)
- **Optimizer States**: 468 MB (2 times model parameters, typical for Adam)
- **Input Data**: 50 MB (depends on input size and batch size)

Total estimated VRAM requirement: \(234 + 200 + 468 + 50 = 952\) MB.

## Conclusion

Large language models are incredibly powerful tools that rely on a vast number of parameters to perform complex language tasks. Understanding how these parameters are calculated and their impact on VRAM usage is crucial for effectively deploying these models in real-world applications. While the resource demands are significant, advances in model optimization, mixed precision training, and hardware capabilities are making it increasingly feasible to leverage these models in various settings. As technology continues to evolve, the balance between model size, performance, and resource efficiency will remain a critical area of focus in the development of large language models.
