# BTM - Biterm Topic Modelling for Short Text with R

This is an R package wrapping the C++ code available at https://github.com/xiaohuiyan/BTM for constructing a **Biterm Topic Model (BTM)**. This model models word-word co-occurrences patterns (e.g., biterms). 

> Topic modelling using biterms is particularly good for finding topics in short texts (as occurs in short survey answers or twitter data).

### Installation

This R package is not on CRAN, you can only install it as follows: `devtools::install_github("jwijffels/BTM")`

### What

The Biterm Topic Model (BTM) is a word co-occurrence based topic model that learns topics by modeling word-word co-occurrences patterns (e.g., biterms)

- A biterm consists of two words co-occurring in the same context, for example, in the same short text window. 
- BTM models the biterm occurrences in a corpus (unlike LDA models which model the word occurrences in a document). 
- It's a generative model. In the generation procedure, a biterm is generated by drawing two words independently from a same topic `z`. In other words, the distribution of a biterm `b=(wi,wj)` is defined as: `P(b) = sum_k{P(wi|z)*P(wj|z)*P(z)}` where k is the number of topics you want to extract.
- Estimation of the topic model is done with the Gibbs sampling algorithm. Where estimates are provided for `P(w|k)=phi` and `P(z)=theta`.

More detail can be referred to the following paper:

> Xiaohui Yan, Jiafeng Guo, Yanyan Lan, Xueqi Cheng. A Biterm Topic Model For Short Text. WWW2013.
> https://github.com/xiaohuiyan/xiaohuiyan.github.io/blob/master/paper/BTM-WWW13.pdf


### Example

```
library(udpipe)
library(BTM)
data("brussels_reviews_anno", package = "udpipe")

## Taking only nouns of Dutch data
x <- subset(brussels_reviews_anno, language == "nl")
x <- subset(x, xpos %in% c("NN", "NNP", "NNS"))
x <- x[, c("doc_id", "token")]

## Building the model
set.seed(321)
model  <- BTM(x, k = 4, beta = 0.01, iter = 1000, trace = 100)

## Inspect the model - topic frequency + conditional term probabilities
model$theta
[1] 0.3353083 0.2023678 0.2043752 0.2579486

topicterms <- terms(model, top_n = 10)
topicterms
[[1]]
         token probability
1  appartement 0.049776587
2      Brussel 0.024257148
3        kamer 0.016732859
4        buurt 0.014101989
5      centrum 0.013523198
6          bed 0.011418502
7     badkamer 0.010260919
8       mensen 0.010208302
9      locatie 0.010103067
10  slaapkamer 0.009682128

[[2]]
         token probability
1  appartement 0.076861701
2      Brussel 0.041358005
3      centrum 0.020945280
4        buurt 0.018854591
5     verblijf 0.011784262
6      minuten 0.011518175
7     aanrader 0.010187736
8     aankomst 0.010035686
9         stad 0.009465498
10     locatie 0.009389473

[[3]]
         token probability
1  appartement  0.04586403
2      Brussel  0.02829498
3        kamer  0.01957048
4      centrum  0.01668900
5         huis  0.01592861
6     verblijf  0.01352737
7        buurt  0.01288704
8      locatie  0.01136626
9         stad  0.01032572
10    badkamer  0.01000555

[[4]]
         token probability
1  appartement  0.05632430
2      Brussel  0.03434098
3         huis  0.02144649
4        buurt  0.02081968
5     verblijf  0.01732742
6      centrum  0.01477539
7        kamer  0.01423811
8     aanrader  0.01356653
9         stad  0.01222335
10 loopafstand  0.01101449

scores <- terms(predict, newdata = x)
```

Make a specific topic called the background

```
# If you set background to TRUE
# The first topic is set to a background topic that equals to the empirical word dsitribution. 
# This can be used to filter out common words. Defaults to FALSE.
set.seed(321)
model  <- BTM(x, k = 5, beta = 0.01, background = TRUE, iter = 1000, trace = 100)
topicterms <- terms(model, top_n = 5)
topicterms
```