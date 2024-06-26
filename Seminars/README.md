Here will briefly write about the seminars and the most important things I learned in them.

---

## 1. Word Embeddings

- **Principial Component Analysis** (PCA) $-$ the simplest linear dimensionality reduction method.

<br>

- **t-distributed Stochastic Neighbor Embedding** (t-SNE) $-$ popular method for exploring high-dimensional data, ability to create compelling two-dimensonal “maps” from data with hundreds or even thousands of dimensions.

<br>

<br>

## 2. Text Classification 

- **Generative** vs **Discriminative** $-$ in generative we learn the joint distribution of data (we can then generate this data independently), discriminative - we are only interested in the separating boundary between classes 

<br>

- **Laplace (add-one) smoothing** $-$ adding a small amount to the numerator and denominator of a fraction to yield a non-zero probability of encountering the word in the document 

<br>

### *Classical Classification methods:*

1. **Naive Bayes classifier** $-$ generative model, which assumes that all words are in a certain "bag" and their order is not important to us; not a very accurate model, but it is easy to control what and where went wrong

<br>

2. **Logistic Regression** $-$ there is some set of attributes, by which we define the object, and we introduce weights for each attribute; as a probability we take the softmax of the scalar product $w^{(k)}f$

<br>

3. **Support Vector Machine** (SVM) $-$ also match features for an object (sentence), then apply SVM for classification (uses bag of words + biagrams)

<br>

### *Neural Classification methods:*

Instead of hand selection, the features vector is selected by a neural network.

1. *Weighted* Bag of Words (BOW) $-$ just summarize embeddings of words in the text and get feature vector

<br>

2. Convolutional Neural Network (CNN) $-$ like CNN's idea for pictures (find a cat no matter where it is);
    - **convolution** $-$ linear layer that takes only some window of vectors
    - **non-lineariry** (ReLU) $-$ 0 if no cat, 1 if cat (like threshold)
    - **pooling** $-$ averaging, if there is a cat at least somewhere 

<br>

3. Recurrent Neural Network (RNN) $-$ a more natural idea for texts, the RNN reads the text word for word;
    - **two RNN layers** can be done: remember everything that you read and read one more time
    - **disadvantage**: can forget the beginning of the phrase if it's long
    - **fix**: one RNN reads left-to-right, the second reads right-to-left and then result is concatenated 

### To Note:
- **Fine-tuning importance:** in Word2Vec, for example, antonyms will have very close embeddings ("black" and "white", "good" and "bad") because they often occur in similar contexts: this can be bad for the semantics task.
- **Data Augmentation:**
    - word dropout (replace word with UNK/random word/synonyoums)
    - paraphrasing (translate and translate back)
- **Tokenization** $-$ Tokenization and vocabulary building should happen on **training set only**
- **Embedding layer** $-$ words are represented as one-hot vectors, which are high-dimensional and sparse, which makes them difficult to work with; the embedding layer could map each word to a low-dimensional vector

<br>

<br>

## 3. Language Modeling
Language Models (LMs) estimate the probability of different linguistic units: symbols, tokens, token sequences.

- One of the main questions in Language Models is to find the probablility of some specific sentence. Formally, let $y_1, y_2, \ldots, y_n$ be tokens in a sentence, and $P\left(y_1, y_2, \ldots, y_n\right)$ the probability to see all these tokens (in this order). Using the product rule of probability (aka the chain rule), we get:

$$
P\left( y_1, y_2, \ldots, y_n \right) = P\left(y_1\right) \cdot P\left(y_2 \mid y_1\right) \cdot P\left(y_3 \mid y_1, y_2\right) \cdots P\left(y_n \mid y_1, \ldots, y_{n-1}\right) = \prod_{t=1}^n P\left(y_t \mid  y_{\textless t} \right)
$$

What we got is the standard left-to-right language modeling framework. This framework is quite general: N-gram and neural language models differ only in a way they compute the conditional probabilities: 

- **N-gram models** $-$ *Markov Property*, probability of a word only depends on a fixed number of previous words (unigram model = bag of words)
    - Backoff (aka Stupid Backoff)
    - Linear interpolation

<br>

- **Neural models** $-$ main idea here is to get a vector representation for the previous context; using this representation, a model predicts a probability distribution for the next token; looks similar to **classification**!
    - Last linear layer maps to the size of vocabulary, then apply Softmax
    - Types of NM:
        - Recurent NN (few Layers of RNN) [RNN $\longrightarrow$ GRU $\longrightarrow$ LSTM]
        - Convolution NN

Generation quality metric $-$ instead of cross-entropy, it is more common to report its transformation called perplexity which is from 1 to $|V|$ (**note that** the size of vocabulaty is important! what is token, since that boarders of perplexity can be different)

<br>

<br>

## 4. Seq2seq and Attention
**Sequence to Sequence task** $-$ usually, translation from one natural language to another; encoder-decoder is the standard modeling paradigm for sequence-to-sequence tasks:
- Encoder $-$ reads source sequence and produces its representation
- Decoder $-$ uses source representation from the encoder to generate the target sequence

<br>

**Attention** (2014) $-$ fixed source representation is suboptimal: (i) for the encoder, it is hard to compress the sentence; (ii) for the decoder, different information may be relevant at different steps $-$ in each state, I'd like to peek at which part of the source is most useful right now 

Ways to compute attention score:

- dot-product 
- bilinear function 
- multi-layer perceptron 

<br>

**Transformer** (2017) $-$ model based solely on attention mechanisms, introduced a new modeling paradigm: in contrast to previous models where processing within encoder and decoder was done with recurrence or convolutions, Transformer operates using only attention.

<br>

<br>

## 5. Transfer Learning

**Transfer Learning** $-$ "transfer" knowledge from one task/model to another: e.g. you don't have a huge amount of data for the classification task, and it is hard to get a good model using only this data; instead, you can have data for some other task (language modeling), which is easier to get (for LM you don't need labels at all - plain texts are enough)

- **Simplest Transfer** $-$ Word Embeddings
    - take pretrained (Word2Vec, GloVe, etc) and fix them (use them as static vectors)
    - initialize with pretrained embeddings and train them with the network ("fine-tune")

<br>

The two great ideas:
1. from words to words-in-context, Put Representations Instead of Word Embeddings: Word2Vec/GloVe/etc $\longrightarrow$ **Cove/ELMo**
2. Refuse From Task-Specific Models $-$ CoVe/ELMo replace word embeddings, but **GPT/BERT** replace entire models!

<br>

### Great Idea 1: Embeddings from Language Models

**ELMo:** (2018)
Just by replacing word embeddings (GloVe) with embeddings from LM they got a huge improvement

- authors represent words as outputs of a **character-level network**: CNN operates over this charecter representation, which is very simple and consists of the components we already saw before: convolution, global pooling, highway connections, and linear layers; in this way, word representations know their characters by construction, and we can represent even those words we've never seen in training (BPE will have some strange tokens, while this way will make words with typos similar)

![image](https://github.com/victoriazinkovich/NLP-YSDA/assets/78615928/4a3a4e79-5a00-46be-a84f-953d427a5b60)

- consists of the two-layer LSTM language models: forward and backward; two models are used so that each token could have both contexts: left and right
- once the model is trained, we can use it to get word representations: combine representations from the corresponding layers from the forward and backward LSTMs

![image](https://github.com/victoriazinkovich/NLP-YSDA/assets/78615928/60e28b89-bfa4-4dbb-af7b-97ebe60e6da6)


<br>

### Great Idea 2: Refuse From Task-Specific Models
GPT/BERT act not as a replacement for word embeddings, but as a replacement for task-specific models

- model is first pretrained using a huge amount of unlabeled data (plain texts)
- model is fine-tuned on each of the downstream tasks
- now during fine-tuning you have to only **feed the data in a certain way** instead of modifying model architecture.



















