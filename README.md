<h1 align="center">
  <img src="https://avatars.githubusercontent.com/u/54333248?s=200&v=4">
    <br>
    Pinecone Text Client
    <br>
</h1>

The Pinecone Text Client is a Python package that provides text utilities designed for seamless integration with Pinecone's [sparse-dense](https://docs.pinecone.io/docs/hybrid-search) (hybrid) semantic search.

> **_⚠️ Warning_**
>
> This is a **public preview** ("Beta") version.
> For any issues or requests, please reach out to our [support](support@pinecone.io) team.
## Installation
To install the Pinecone Text Client, use the following command:
```bash
pip install -U pinecone-text
```

## Sparse Encoding

To convert your own text corpus to sparse vectors, you can either use [BM25](https://www.pinecone.io/learn/semantic-search/#bm25) or [SPLADE](https://www.pinecone.io/learn/splade/).

### BM25
To encode your documents and queries using BM25 as vector for dot product search, you can use the `BM25Encoder` class.

> **_📝 NOTE:_**
>
> Our current implementation of BM25 supports only static document frequency (meaning that the document frequency values are precomputed and fixed, and do not change dynamically based on new documents added to the collection).
>
> When conducting a search, you may come across queries that contain terms not found in the training corpus but are present in the database. To address this scenario, BM25Encoder uses a default document frequency value of 1 when encoding such terms.
#### Usage

For an end-to-end example, you can refer to our Quora dataset generation with BM25 [notebook](https://github.com/pinecone-io/examples/blob/master/pinecone/sparse/bm25/bm25-vector-generation.ipynb).

```python
from pinecone_text.sparse import BM25Encoder

corpus = ["The quick brown fox jumps over the lazy dog",
          "The lazy dog is brown",
          "The fox is brown"]

# Initialize BM25 and fit the corpus
bm25 = BM25Encoder()
bm25.fit(corpus)

# Encode a new document (for upsert to Pinecone index)
doc_sparse_vector = bm25.encode_documents("The brown fox is quick")
# {"indices": [102, 18, 12, ...], "values": [0.21, 0.38, 0.15, ...]}

# Encode a query (for search in Pinecone index)
query_sparse_vector = bm25.encode_queries("Which fox is brown?")
# {"indices": [102, 16, 18, ...], "values": [0.21, 0.11, 0.15, ...]}

# store BM25 params as json
bm25.dump("bm25_params.json")

# load BM25 params from json
bm25.load("bm25_params.json")
```

#### Load Default Parameters
If you want to use the default parameters for `BM25Encoder`, you can call the `default` method.
The default parameters were fitted on the [MS MARCO](https://microsoft.github.io/msmarco/)  passage ranking dataset.
```python
bm25 = BM25Encoder.default()
```

#### BM25 Parameters
The `BM25Encoder` class offers configurable parameters to customize the encoding:

* `b`: Controls document length normalization (default: 0.75).
* `k1`: Controls term frequency saturation (default: 1.2).
* Tokenization Options: Allows customization of the tokenization process, including options for handling case, punctuation, stopwords, stemming, and language selection.

These parameters can be specified when initializing the BM25Encoder class. Please read the BM25Encoder documentation for more details.

