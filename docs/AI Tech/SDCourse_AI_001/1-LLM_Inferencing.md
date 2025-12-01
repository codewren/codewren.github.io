# LLM Inferencing

## LLM Architecture

Large Language Models (LLMs) are neural networks designed to understand and generate human language. They typically use transformer architectures, which rely on attention mechanisms to process input text efficiently and capture long-range dependencies.

### Key Components of LLM Architecture

- **Embedding Layer:** Converts input tokens into dense vector representations.
- **Positional Encoding:** Adds information about token positions to the embeddings.
- **Attention Mechanism:** Enables the model to focus on relevant parts of the input sequence.
- **Feedforward Layers:** Process the outputs of the attention mechanism for deeper representation.
- **Layer Normalization:** Stabilizes and accelerates training by normalizing activations.
- **Output Layer:** Maps processed representations back to the vocabulary for prediction.


# Basic Term of Model 
##  Perplexity 
Perplexity in machine learning, particularly in **Natural Language Processing (NLP)**, is a metric used to evaluate how well a **probability model** (like a language model) predicts a sample of text.

### 🎯 Key Concepts

* **Predictive Power:** Perplexity quantifies the **uncertainty** or "surprise" a model has when predicting the next word (or token) in a sequence.
* **Interpretation:** A **lower perplexity score is better** because it indicates the model is more confident and accurate in its predictions, reflecting a stronger grasp of the language structure.
* **Analogy:** A perplexity score of 10 means the model is, on average, as uncertain as if it were choosing uniformly from **10 equally likely options** for the next word. A perplexity of 1 would mean perfect prediction.

### 📝 Calculation Summary

Mathematically, perplexity (**PP**) is related to the **cross-entropy** (**H**) of the model's predictions over a sequence of words, $(w_1, w_2, \dots, w_N)$:
$$\text{PP} = e^{H}$$
Where $H$ is the average negative log-likelihood of the sequence.

$$\text{PP}(W) = \left( \prod_{i=1}^N \frac{1}{P(w_i \mid w_1, \dots, w_{i-1})} \right)^{\frac{1}{N}}$$

In simpler terms, it's the inverse of the normalized probability that the model assigns to the test text.

Would you like a more detailed explanation of the mathematical relationship between perplexity and cross-entropy?

You can watch this video for a breakdown of this key metric: [perplexity in NLP: The Key Metric Behind AI Language Models. Understanding #ai #modelevaluation](https://www.youtube.com/watch?v=9dp4rsNpBKA). This video explains what perplexity is in the context of AI and NLP and why it's a key metric for evaluating language models.


http://googleusercontent.com/youtube_content/0




