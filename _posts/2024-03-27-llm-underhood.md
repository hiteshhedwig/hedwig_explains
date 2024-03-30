---
layout: post
title: "Explained : Attention mechanisam & Transformers"
subtitle: "what actually is LLM ?"
date: 2024-03-27
author: "Hitesh Kumar"
header-img: "img/llm-underhood/assets/pexels-andrew-neel-3178786.jpg"
tags: [lidar, sensor, hardware, working, theory]
---


# Introduction 

Year of 2022 and 2023, probably go down as the "infliction year", on similar magnitude to rise of internet services. ChatGPT, Claude, Gemini based high quality LLM(large language models) which can assist with plenty of tasks. But how come these being so powerful and state of the art ?

Well, this core approach of LLM was introduced back in 2017, the groundbreaking paper "Attention is all you need". In this blog, the scope is to understand attention mechanism and how it powers transformer model which directly powers LLMs. We will discuss the intuitive and methematical approch to comprehend LLMs.


# Intuition 

## Idea
![photo1](/hedwig_explains/img/llm-underhood/assets/attention_sentence.png)
Assume a state where you encounter a word "apple". Now, you have to learn if this word is related to apple being a fruit or a company. So, you'd like to learn that what is the context here ? If the chain of words is something like "apple's iphone is quite expensive", you can now understand. What does apple refer to here ?
So, its always about the context. What is the context says defines the meaning of the word in that sentence. This is what attention mechanism addresses. Learning from the context and develop a reasoning. 

![photo1](/hedwig_explains/img/llm-underhood/assets/attention_1.png)

In the above image, you can see the word "She" is being strongly associated with "Nurse" and mildly with other words. This is the essence of attention mechanism. 

However, human brain works in highly complicated ways. Which is difficult to breakdown. We have to tailor it for computers. We employ embedding which encapsulates and pulls right context words into right places that is similar words group. 

![photo1](/hedwig_explains/img/llm-underhood/assets/computer_sees.png)

![photo1](/hedwig_explains/img/llm-underhood/assets/context_pulls.png)

These embeddings are a hyperparameter which gets optimized during the training. This is a single example of embedding. But during training we want plently of embeddings, so we apply linear transformtion to stretch and transform embedding so the model can learn based on different embeddings. 


## Embeddings and Attention formula :


The formula of attention is pretty neat and simple looking. Q, K, V are the matrices. And the softmax function to normalize the part of output. 

![photo1](/hedwig_explains/img/llm-underhood/assets/formula.png)

> The Q & K matrices help us to get new embeddings from the existing ones. This embedding knows color, size, feature about the word.

![photo1](/hedwig_explains/img/llm-underhood/assets/kq_similarity.png)


> The V matrix is basically to knows the next best word.

![photo1](/hedwig_explains/img/llm-underhood/assets/self-attention.png)

The key/value/query concept is analogous to retrieval systems. For example, when you search for videos on Youtube, the search engine will map your query (text in the search bar) against a set of keys (video title, description, etc.) associated with candidate videos in their database, then present you the best matched videos (values).

### Attention and it's variations
#### Self attention  

Assume a sentence "Hi, How are you ?". Before arriving it to attention. The sentence goes through the process of tokenization, embedding creation, and position encoding. After all these steps the obtained matrix is of size (512,6) which is fed into attention mechanism.

> skip SOS, EOS for now.

![photo1](/hedwig_explains/img/llm-underhood/assets/self_attention.png)

In self-attention : K, Q, V matrices are derived from input sequence matrix. Which finding context between the words within the sentence (that's why it's called self attention). 

![ph1](/hedwig_explains/img/llm-underhood/assets/self_attention_kqv.png)

Now, following the formula where we are taking a dot product of Q and K(transpose).  
Depending on the problem statement, we do not want current token to communicate with future tokens. Rather, we want them to communicate with previous context token to generate new token. For this we turn the elements after diagonal into -infinity. So that after softmax function those elements are turned to zero.

As said, depending on problem statement. For example, in sentiment analysis kind of a problem we need to communicate with other tokens to know the proper sentiment of the sentence. In this case, we don't need to do that "-infinity" step. 

> Known as "Masked Attention"

![ph1](/hedwig_explains/img/llm-underhood/assets/self_attention_matrix.png)


Each row of the attention scores matrix represents the attention weights for a specific position in the input sequence, indicating how much attention should be paid to each position.

We will get a matrix output which will further be divided by underroot d_k which is 512 according to the paper and apply softmax function. 
![ph1](/hedwig_explains/img/llm-underhood/assets/self_attention_dk.png)

Now, this will result in a matrix. Which is ready to be multiplied by V (value) matrix to give us the attention output scores! 


#### Multi-Head Attention

In this case, we have multiple K, Q, V matrices. The idea is to allow K,Q,V matrices to focus on subset of the embedding to understand different context and be more robust.

![ph1](/hedwig_explains/img/llm-underhood/assets/multi_head_attention.png)

Attention output of respective attention heads are concatenated together and further multiplied by a matrix W_0 to provide multihead attention output.

![ph1](/hedwig_explains/img/llm-underhood/assets/multi-head_attention_map.png)


This is probably all you need to know about attention.


## Transformers & It's working

looks something like this :

![photo1](/hedwig_explains/img/llm-underhood/assets/transformers.png)

There are 3 parts to a transformer model. 
1. Encoder 
2. Decoder
3. Output

![photo1](/hedwig_explains/img/llm-underhood/assets/transformer_annotated.png)


Let's consider an example where you have to convert this korean text "안녕하세요, 만나서 반가워요" to English. Which is a machine translation problem.

Before the Korean text is fed into encoder. Raw text is 
1. Tokenized
2. Embeddings
3. Positional Encoding


### Training 

The training pipeline is slightly different from inference sessions. But in both situations we need raw text to be tokenized and all. 

- Encoder : Goes through Multihead attention and other components to produce a vector which is ready to be fed into decoder.
    ![photo1](/hedwig_explains/img/llm-underhood/assets/encoder.png)


- Decoder : The raw target text is encoded something like "SOS [][] [] EOS" and processed. And the idea of "cross attention" is introduced in there. Encoder output is utilized in decoder to produce attention output. 

    ![photo1](/hedwig_explains/img/llm-underhood/assets/decoder_real.png)


- Output : The language model head that uses linear layer to map (seq, d_model) to (seq, vocab_size). And calculate cross entropy loss between labels and model's prediction. 
    
    ![photo1](/hedwig_explains/img/llm-underhood/assets/decoder_real_annotated.png)

The loss gets minimized by the optimization function and backpropogated.


### Inference

Similarly, 
- Encoder : Goes through Multihead attention and other components to produce a vector which is ready to be fed into decoder.
    ![photo1](/hedwig_explains/img/llm-underhood/assets/encoder.png)


- Decoder : There's a slight change in decoder during inference. Now that we do not know the target text. At the very first timestamp T=0. We pass a default token which is SOS(start of sentence). If model trained correctly, in this case will produce an output "hey".
    ![photo1](/hedwig_explains/img/llm-underhood/assets/decoder_inference.png)

- At (timestamp) T=1, decoder will be fed, "SOS Hey" which will produce.. ","
- At (timestamp) T=2, decoder will be fed, "SOS Hey," which will produce.. "good"
- At (timestamp) T=3, decoder will be fed, "SOS Hey, good" which will produce.. "to"

This will continue until its decoder outputs EOS (end of sentence). So the translated text will be "hey, good to know you".


------

Hopefully, this is enough to understand the working of transformers. If any doubts or corrections, please let me know !