# Notebooks
- ```wcd_rrf2.ipynb``` is a manually implemented version of the algorithm
- ```wcd_fusion.ipynb``` is been implemented using Weaviate library that implements a fusion algorithm

# Documents
We will ask questions on two document:
- `CLI_AR.pdf`: Real Estate 2022 Annual Report by CAPITALAND INVESTMENT LIMITED
- `ML.md`: a Markdown document that we will use for Markdown Header Text Splitter

# ```wcd_rrf2.ipynb```
For the manually developed RRF, we are loading a CSV. We first process the csv with a preprocessing pipeline.
It involves the following steps:
- stopwords removal
- lemmatization

This is necessary because the BM25 library we are using doesn't implement preprocessing.
Each .csv corpus will be initially preprocessed. 
We then extract the rank of each document using BM25 and vector retrieval and with RRF we fuse the 2 metrics.
The query also goes through a preprocessing pipeline. For the generation we keep the 1st most relevant document.
We create the prompt by concatenating the query and the context. For context we use the original version (before the preprocessing).
We use Chroma for search and vector store and OpenAI for embeddings.

# ```wcd_fusion.ipynb```
Fusion algorithm implemented on Weaviate library.

## What is RAG Fusion?
RAG Fusion is a technique for generating search results, in an information retrieval sistem. Differently from simple RAG, we use reciprocal rank fusion (RRF), which is technique used to combine multiples retrieval algorithm in a single search. For example we can use both a vector based and a keyword based search.
Specifically in RRF, to obtain the final ranking of a given document we sum the reciprocals of the rank numbers of a document across of the different methods.
For instance if document A has rank 3 with BM25 (keyword search) and rank 5 with vector similarity search, the final rank will be
1/3 + 1/5 = 0.53
This way we take advantage of different search characteristic at once.

## What is chunking and which chunking strategy do we use ?
In terms of chunking strategy we are using recursive chunking. 
The separators are : "\n\n", "\n", " ", ""
This means that first we split the text in chunks on "\n\n". The resulting chunks are in turn split on the following separators, until each chunk size is at least smaller than the predefined maximum chunk size. If the splitting results in chunks that are smaller than the predefined minumum size, they will be then merged back. 
The advantage of recursive chunking is that usually this way we have a more logical separation, following also the structure decided by the authors.


## Other strategies?
Character text splitter would be the most basic chunking strategy, where we just define a sliding window size and overlap. 
We are also trying with a markdown document. We use the markdown splitter, which separates chunks based on where the headers are. This method could then in turn be combined with a simple recursive splitter on the internal chunks.
We are going to experiment with different strategies and assess the one that provide the best results.


## How do we extract keywords for the fusion pipeline?
For the fusion pipeline we are going to use 2 different ranking methods and merge the respective metrics. We can also decide to feed specific properties to the keyword search. Specifically, we will want to only feed keywords extracted from each chunk. To extract keywords from a document we will try with 2 different algorithms:
- YAKE (statistical, very fast)
- KeyBERT (transformer based, slower, more precise)

## What do we do in WeaviateClientSession?
We define the persistent vector store path to ./db

For each document we create 4 properties:
- document_id
the index for the document
- chunk_text
the actual text for the chunk
- chunk_index
the index of the chunk
- keywords
Keywords for the chunk, extracted eg with YAKE

## How does the search function work ?
When we setup the search function we can set specific parameters.
Alpha is the balance between the vector and the keyword search results.
In terms of number of results, we could get a specified number (limit argument) or we want to return the whole group of more relevant results. In this case we the the auto_limit, which specifies how many jumps we want in the similarity of the results.


