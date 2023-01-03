---
layout: post
title: Hybrid Search Explained
description: "Learn about the new hybrid search feature introduced in Weaviate 1.17."
published: true
author: Erika Cardenas 
author-img: /img/people/erika.jpg
card-img: /img/blog/hybrid-search-explained/hybrid-search.png
hero-img: /img/blog/hybrid-search-explained/hybrid-search.png
og: /img/blog/hybrid-search-explained/hybrid-search.png
date: 2023-01-03
toc: true
---
Hybrid search is a technique that **combines** multiple search algorithms to improve the accuracy and relevance of search results. It uses the best features of both keyword-based search algorithms with vector search techniques. By leveraging the strengths of different algorithms, it provides a more effective search experience for users.

The [Hybrid search](/developers/weaviate/current/graphql-references/vector-search-parameters.html#hybrid) feature was introduced in Weaviate 1.17. It uses sparse and dense vectors to represent the meaning and context of search queries and documents.

In this blog post, you will learn about the implementation of hybrid search in Weaviate and how to use it. 

## Sparse and Dense Vectors
Sparse and dense vectors are calculated with distinct algorithms. Sparse vectors have mostly zero values with only a few non-zero values, while dense vectors mostly contain non-zero values. Sparse embeddings are generated from algorithms like [BM25](https://en.wikipedia.org/wiki/Okapi_BM25) and [SPLADE](https://arxiv.org/abs/2107.05720). Dense embeddings are generated from machine learning models like [GloVe](https://text2vec.org/glove.html) and [Transformers](https://huggingface.co/docs/transformers/index). 

Note, the current implementation of hybrid search in Weaviate uses BM25/BM25F and vector search. 

If you’re interested to learn about how dense vector indexes are built and optimized in Weaviate, check out this [article](https://weaviate.io/blog/2022/09/Why-is-Vector-Search-so-fast.html). 
### BM25
BM25 builds on the keyword scoring method [TF-IDF](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) (Term-Frequency Inverse-Document Frequency) by taking the [Binary Independence Model](https://en.wikipedia.org/wiki/Binary_Independence_Model) from the IDF calculation and adding a normalization penalty that weighs a document’s length relative to the average length of all the documents in the database. 

The image below presents the scoring calculation of BM25:
![BM25 calculation](/img/blog/hybrid-search-explained/BM25-calculation.png)
<div align="center">Source: Wikipedia page on Okapi BM25</div>


The score of the document, query pair is determined by weighing the uniqueness of each keyword in the query relative to the collection of texts. BM25 contains additional static parameters, k1 and b that may help calibrate performance to particular datasets.

### BM25F
BM25F was also implemented in Weaviate `1.17`. BM25F is a variant of BM25 that allows multiple text fields per object to be given different weights in the ranking calculation. These weights are important for when fields in a document are more important than others. For example, a title may be given more weight than the abstract, since the title is sometimes more informative and concise. This type of weighting makes BM25F more flexible and customizable than BM25. 

### Dense Vectors 
Dense vectors represent information stored in a database; this includes text, images, and other types of data. These embeddings are generated from machine learning models that convert data to vectors. The vectors are densely packed with information and are mostly made up of non-zero values. The meaning of each value in the vector depends on the machine learning model that you used. 

Vector search engines, like [Weaviate](/developers/weaviate/current/), store these embeddings and calculate the distance between the two vectors. [Distance metrics](/blog/2022/09/Distance-Metrics-in-Vector-Search.html) show how similar or dissimilar two vector embeddings are. The search query is converted to a vector, similar to the data vectors, and the distance value determines how close the vectors are. 

![Hybrid Search](/img/blog/hybrid-search-explained/hybrid-search.png)

## Hybrid Search Explained
Hybrid search merges dense and sparse vectors together to deliver the best of both search methods. Generally speaking, dense vectors excel at understanding the context of the query, whereas sparse vectors excel at keyword matches. Consider the query, “How to catch an Alaskan Pollock”. The dense vector representation is able to disambiguate “catch” as meaning fishing rather than baseball or sickness. The sparse vector search will match the phrase “Alaskan Pollock” only. This example query shows where hybrid search combines the best of both sparse and dense vectors.

## Reciprocal Rank Fusion (RRF) 

While researching hybrid search, we needed a way to combine the results of BM25 and dense vector search into a single ranked list. We came across a paper from Benham and Culpepper exploring rank fusion techniques. This paper analyzed seven strategies for combining the ranked results of two lists into a single ranking. We decided to start with the RRF score. The RRF score is calculated by taking the sum of the reciprocal rankings that is given from each list. By putting the rank of the document in the denominator, it penalizes the documents that are ranked lower in the list. 

![RRF Calculation](/img/blog/hybrid-search-explained/RRF-calculation.png){:width="50%"}
<div align="center"> Source: Benham and Culpepper 2017 </div>

Let’s look at an example of this. We have three documents labeled `A`, `B`, and `C` and have run a BM25 and Dense search. In this example, we have set the constant *k* to 0. 

| BM25 Ranking | Dense Ranking | Results
| --- | --- | --- |
| A | B | A: 1/1 + 1/3 = 1.3 |
| B | C | B: 1/2 + 1/1 = 1.5 |
| C | A | C: 1/3 + 1/2 = 0.83|

The above table shows the ranked order of the BM25 and Dense search. To fuse the two lists together, we need to take the sum of the reciprocal ranks. Based on the results, the top document is `Document B` with a ranking of 1.5, then `Document A` at 1.3, and `Document C` at 0.83. 

Re-ranking is a necessary step when implementing hybrid search. The alpha parameter dictates the weighting of each algorithm and determines the re-ranking of the results. 

## Weaviate UX
To use hybrid search in Weaviate, you only need to confirm that you’re using Weaviate `v1.17` or a later version. You can run the hybrid queries in GraphQL or the other various client programming languages. 

There are five parameters needed to run the hybrid search query (some are optional):
* `hybrid`: shows that you want to use hybrid search
* `query`: search query 
* `alpha`(optional): weighting for each search algorithm (alpha = 0 (sparse), alpha = 1 (dense), alpha = 0.5 (equal weight for sparse and dense))
* `vector` (optional): optional to supply your own vector 
* `score`(optional): additional information on how much the sparse and dense method contributed to the result

With just a few lines of code, you can start using hybrid search. You can run a test query in the [Weaviate console](https://link.semi.technology/3IhrVbB) using GraphQL. The query is, “Fisherman that catches salmon” (similar to the example above). When we set the alpha to 0.5 it is equally weighing the dense and sparse vector results. 

```
{
    Get {
    Article (
      hybrid: {
        query: "Fisherman that catches salmon"
        alpha: 0.5
      })
     {
      title
      summary
      _additional {score}
    }
  }
}
```

Check out the [documentation](/developers/weaviate/current/graphql-references/vector-search-parameters.html#hybrid) for more information on hybrid search! 

## Stay Connected
Thank you so much for reading! If you would like to talk to us more about this topic, please connect with us on [Slack](https://join.slack.com/t/weaviate/shared_invite/zt-goaoifjr-o8FuVz9b1HLzhlUfyfddhw){:target="_blank"} or [Twitter](https://twitter.com/weaviate_io){:target="_blank"}. 

Weaviate is open-source, you can follow the project on [GitHub](https://github.com/semi-technologies/weaviate){:target="_blank"}. Don't forget to give us a ⭐️ while you are there.