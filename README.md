# REPLUG
Final Project for Cornell's Introduction to Deep Learning, CS4782

## Introduction

For our final project, we have chosen to reimplement the paper [**Replug: Retrieval-augmented black-box language models**](https://arxiv.org/abs/2301.12652), submitted to ACL 2024 by Weijia Shi, Sewon Min, Michihiro Yasunaga, Minjoon Seo, Rich James, Mike Lewis, Luke Zettlemoyer, and Wen-tau Yih.

This paper builds upon previous iterations of non-parametric language models, such as the highly popular Retrieval-Augmented Generation (RAG) architecture, making changes in the architecture that improve Language Model (LM) performance by a sizable margin on a host of metrics. The main methods chosen to do this are:

1. Ensemble next token generation across the top-k retrieved contexts appended to a query. This aids performance by giving a more holistic prediction, and enabling more context for generation without exceeding the token limit of the final LLM in the pipeline.
2. Provide a framework to fine-tune the retriever module. Previous non-parametric methods have attempted fine-tuning in a manner that fine-tunes the final generator, keeping the retriever parameters fixed. In comparison, REPLUG provides a framework which fixes the parameters of the final LM and instead fine-tunes the retriever.

## Chosen Result

We chose to reimplement the paper's results regarding Bits per Byte (BPB) performance increases when utilizing REPLUG and REPLUG LSR (where LSR is their fine tuning method) with GPT2 small. In the paper, the authors find that GPT2 small has a BPB of 1.33, which decreases to 1.26 with REPLUG, and 1.21 with REPLUG LSR. This represented a 5.3% and 9% performance increase respectively. BPB is a metric which measures exactly the average number of bits needed to encode an ASCII byte. This is important for language models, because a lower value indicates that they are more efficient at making predictions, needing less bits to predict the next token. This also be interpreted as a model's confidence in a given token, with a lower BPB representing higher confidence. As such, these results show that the author's methodologies give a large (5.3% and 9%) increase in model certainty, which is very impressive! 

Below, you can see the table which includes the result we are attempting to reproduce.

<p align="center">
  <img width="663" alt="Screen Shot 2024-05-13 at 8 58 44 PM" src="https://github.com/SashaBoguraev/REPLUG/assets/52136865/ce661231-8270-44fe-8117-0b40289d73e4">
<p>
  
## Re-implementation Details
## Results and Analysis
## Conclusion and Future Work
## References
