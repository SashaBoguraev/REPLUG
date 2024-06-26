# REPLUG
Final Project for Cornell's Introduction to Deep Learning, CS4782

## Introduction

For our final project, we have chosen to reimplement the paper [**Replug: Retrieval-augmented black-box language models**](https://arxiv.org/abs/2301.12652), submitted to ACL 2024 by Weijia Shi, Sewon Min, Michihiro Yasunaga, Minjoon Seo, Rich James, Mike Lewis, Luke Zettlemoyer, and Wen-tau Yih.

This paper builds upon previous iterations of non-parametric language models, such as the highly popular Retrieval-Augmented Generation (RAG) architecture, making changes in the architecture that improve Language Model (LM) performance by a sizable margin on a host of metrics. The main methods chosen to do this are:

1. Ensemble next token generation across the top-k retrieved contexts appended to a query. This aids performance by giving a more holistic prediction, and enabling more context for generation without exceeding the token limit of the final LLM in the pipeline.
2. Provide a framework to fine-tune the retriever module. Previous non-parametric methods have attempted fine-tuning in a manner that fine-tunes the final generator, keeping the retriever parameters fixed. In comparison, REPLUG provides a framework which fixes the parameters of the final LM and instead fine-tunes the retriever.

## Chosen Result

We chose to reimplement the paper's results regarding Bits per Byte (BPB) performance increases when utilizing REPLUG and REPLUG LSR (where LSR is their fine tuning method) with GPT2 XL. In the paper, the authors find that GPT2 XL has a BPB of 1.16, which decreases to 1.09 with REPLUG, and 1.07 with REPLUG LSR. This represented a 6.0% and 7.8% performance increase respectively. BPB is a metric which measures exactly the average number of bits needed to encode an ASCII byte. This is important for language models, because a lower value indicates that they are more efficient at making predictions, needing less bits to predict the next token. This also be interpreted as a model's confidence in a given token, with a lower BPB representing higher confidence. As such, these results show that the author's methodologies give a large (6.0% and 7.8%) increase in model certainty, which is very impressive! 

Below, you can see the table which includes the result we are attempting to reproduce.

<p align="center">
  <img width="663" alt="Screen Shot 2024-05-13 at 8 58 44 PM" src="https://github.com/SashaBoguraev/REPLUG/assets/52136865/ce661231-8270-44fe-8117-0b40289d73e4">
<p>
  
## Re-implementation Details

In order to reimplement this paper, we aimed to reimplement the base REPLUG, with a stretch goal of implementing REPLUG LSR. In the end, we managed to reimplement a small, working example of REPLUG, whilst not managing to implement REPLUG LSR due to computational and memory constraints. I will describe the implementation strategies of each in turn.

### REPLUG

To reimplement REPLUG, we must first have a retriever model (and tokenizer) as well as a LLM (and tokenizer). Following from the paper, we use [Facebook's Contriever](https://huggingface.co/facebook/contriever) and (GPT2)[https://huggingface.co/openai-community/gpt2]. Furthermore, we must have data. As mentioned in our `data/README.md`, the original data used was from [The Pile](https://pile.eleuther.ai/), however this has been taken down due to copyright, and we utilized [AllenAI's C4 dataset](https://huggingface.co/datasets/allenai/c4). The instructions on how we got this data can be found in `data/README.md`, as well as our modifications from the original data accessing in the paper -- in short, we samples much less data than the paper due to Colab's memory constraints.

One we have our data and models, we can start implementing our paper. To impelment the base REPLUG, we use [Facebook AI's Similarity Search (FAISS)](https://arxiv.org/abs/1702.08734) to hold our datastore. As such, we create our embeddings for our datastore utilizing the Contriever model before constructing a FAISS datastore using these embeddings. This allows us very fast retrieval during that step of our pipeline. Once we have our datastore, we can build our pipeline by writing functions that allow us to retrieve context given a query, append that context to our query, feed it to GPT2, and then ensemble the results. We do this, with the same specifications as the paper, with the only limitation being the amount and type of data in our datastore.

### REPLUG LSR 

In order to reimplement REPLUG LSR, we develop functions that compute both the retrievel likelihood of our documents, as well as the LM likelihood of generation. With these, we can calculate the KL divergence between them, trying to minimize the distance between them, and average this value across our batch to get our loss. Our training loop then follows the specified training in the paper, where we update the retriever model weights by backpropagating this loss. Further, we recompute the datastore every _T_ steps, as specified by the paper, to ensure that our datastore and retriever are still in tandem.

### Hyper-Parameters

The paper states:

> " Given a query x, we retrieve the top 20 documents from the FAISS index and compute the retrieval likelihood and the LM likelihood with a temperature of 0.1. We train the retriever using the Adam optimizer (Kingma & Ba, 2015) with a learning rate of 2e-5, a batch size of 64, and a warmup ratio of 0.1. We re-compute the document embeddings every 3k steps and fine-tune the retriever for a total of 25k steps."

We follow all thse hyperparameters in our implementation.

### Modifications

Other than what is mentioned above, we are not aware of any other modifications we made.

## Results and Analysis

When we reimplement our results, we find that the base GPT2 XL has a BPB of 1.17, with GPT + REPLUG having a BPB of 1.19. This is a 1.9% performance decrease, compared to the 6.0% performance increase we were trying to replicate. We hypothesize that this is likely due to the size of our datastore. In the original paper, there were 36M documents in the datastore, which should allow for most of the sequences being tested to have their contexts in the datastore. As such, the model should be able to retrieve this context and generate tokens with **more confidence**. In comparison, our datastore is very small. As such, many of the test sequences **may not** have their contexts in our datastore, leading to the retrieval of irrelevant context, confusing the model more and lowering confidence.

Furthermore, we were unable to fine-tune our model. This is down to Colab's GPU memory constraints limiting us from being able to train the model. We tried many things to fix this, icnlduing attempting the use of different retrieval models, but we were unable to overcome this issue with our computational resources.

We believe that while our work doesn't show anything wrong with the original paper, we do note that our results show the difficulty in making this model architecture applicable to individuals or organizations who do not have a massively large amount of compute. This architecture is still valuable, but we believe our results show it perhaps not as useful nor commercially viable as classic RAG architectures.

## Conclusion and Future Work

In conclusion, we see the promise of non-parametric LM architectures, such as REPLUG. From the paper's description, they seem like very capable and useful methods to reduce hallucinations and add information not included in the LM's original training corpus. However, we also note that many of the paper's results are difficult, if not impossible, to reproduce without large amounts of computational resources. As such, we think future work should go into making these architectures much more resource efficient, able to run even on small scale examples.

## References

Shi, Weijia, et al. "Replug: Retrieval-augmented black-box language models." arXiv preprint arXiv:2301.12652 (2023).

Johnson, Jeff, Matthijs Douze, and Hervé Jégou. "Billion-scale similarity search with GPUs." IEEE Transactions on Big Data 7.3 (2019): 535-547.

Gao, Leo, et al. "The pile: An 800gb dataset of diverse text for language modeling." arXiv preprint arXiv:2101.00027 (2020).

Izacard, Gautier, et al. "Unsupervised dense information retrieval with contrastive learning." arXiv preprint arXiv:2112.09118 (2021).

Radford, Alec, et al. "Language models are unsupervised multitask learners." OpenAI blog 1.8 (2019): 9.
