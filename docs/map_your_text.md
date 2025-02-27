# Map your text
Map your text with Atlas by sending over lists of JSON files. Atlas handles the embedding for you. Alternatively, embed text with an external model and send the embeddings.


## Map text with Atlas
Send over lists of JSON or Python dictionaries via the `AtlasClient` and Atlas makes a map. When sending text
you should specify an `indexed_field`. This lets Atlas know what metadata field to use when building and organizing the map.

=== "Atlas Embed"

    ``` py title="map_text_with_nomic.py"
    from nomic import AtlasClient
    import numpy as np
    from datasets import load_dataset
    
    atlas = AtlasClient()
    
    #make dataset
    max_documents = 10000
    dataset = load_dataset("sentiment140")['train']
    documents = [dataset[i] for i in np.random.randint(len(dataset), size=max_documents).tolist()]
    
    response = atlas.map_text(data=documents,
                              indexed_field='text',
                              is_public=True,
                              map_name='10k subsample of sentiment140',
                              map_description='A 10,000 point sample of the huggingface sentiment140 dataset embedded with Nomic's text embedder.,
                              organization_name=None, #defaults to your current user.
                              num_workers=10)
    print(response)
    ```

=== "Output"

    ``` bash
    map='https://atlas.nomic.ai/map/ff2f89df-451e-49c4-b7a3-a608d7375961/f433cbd1-e728-49da-8c83-685cd613788b'
    job_id='b4f97377-e2aa-4305-8bc6-db7f5f6eeabf'
    index_id='46445e68-8c9f-470a-aa82-847e78c0f10e'
    ```


## Map text with your own models
Nomic integrates with embedding providers such as [co:here](https://cohere.ai/) and [huggingface](https://huggingface.co/models) to help you build maps of text.


### Text maps with a 🤗 HuggingFace model
This code snippet is a complete example of how to make a map with a HuggingFace model.
[Example Huggingface Map](https://atlas.nomic.ai/map/60e57e91-c573-4d1f-85ac-2f00f2a075ae/f5bf58cf-f40b-439d-bd0d-d3a4a8b98496)

!!! note

    This example requires additional packages. Install them with
    ```bash
    pip install datasets transformers torch
    ```
=== "HuggingFace Example"

    ``` py title="map_with_huggingface.py"
    from nomic import AtlasClient
    from transformers import AutoTokenizer, AutoModel
    import numpy as np
    import torch
    from datasets import load_dataset
    
    atlas = AtlasClient()
    
    #make dataset
    max_documents = 10000
    dataset = load_dataset("sentiment140")['train']
    documents = [dataset[i] for i in np.random.randint(len(dataset), size=max_documents).tolist()]
    
    
    model = AutoModel.from_pretrained("prajjwal1/bert-mini")
    tokenizer = AutoTokenizer.from_pretrained("prajjwal1/bert-mini")
    
    embeddings = []
    
    with torch.no_grad():
        batch_size = 10 # lower this if needed
        for i in range(0, len(documents), batch_size):
            batch = [document['text'] for document in documents[i:i+batch_size]]
            encoded_input = tokenizer(batch, return_tensors='pt', padding=True)
            cls_embeddings = model(**encoded_input)['last_hidden_state'][:, 0] #
            embeddings.append(cls_embeddings)
    
    embeddings = torch.cat(embeddings).numpy()
    
    response = atlas.map_embeddings(embeddings=embeddings,
                                    data=documents,
                                    colorable_fields=['sentiment'],
                                    is_public=True,
                                    map_name="Huggingface Model Example",
                                    map_description="An example of building a text map with a huggingface model.")
    
    print(response)
    ```

=== "Output"

    ``` bash
    map='https://atlas.nomic.ai/map/60e57e91-c573-4d1f-85ac-2f00f2a075ae/f5bf58cf-f40b-439d-bd0d-d3a4a8b98496'
    job_id='f898193b-99bf-4640-9f5c-ab5318eb8c2e'
    index_id='536841ff-e0b5-4dcc-8547-cfdc825e6e94'
    ```


### Text maps with a Cohere model

Obtain an API key from [cohere.ai](https://os.cohere.ai) to embed your text data.

Add your Cohere API key to the below example to see how their large language model organizes text from a sentiment analysis dataset.

[Sentiment Analysis Map](https://atlas.nomic.ai/map/63b3d891-f807-44c5-abdf-2a95dad05b41/db0fa89e-6589-4a82-884b-f58bfb60d641)

!!! note

    This example requires additional packages. Install them with
    ```bash
    pip install datasets
    ```

=== "Co:here Example"

    ``` py title="map_hf_dataset_with_cohere.py"
    from nomic import AtlasClient
    from nomic import CohereEmbedder
    import numpy as np
    from datasets import load_dataset
    
    atlas = AtlasClient()
    cohere_api_key = ''
    
    dataset = load_dataset("sentiment140")['train']
    
    max_documents = 10000
    subset_idxs = np.random.randint(len(dataset), size=max_documents).tolist()
    documents = [dataset[i] for i in subset_idxs]

    embedder = CohereEmbedder(cohere_api_key=cohere_api_key)
    
    print(f"Embedding {len(documents)} documents with Cohere API")
    embeddings = embedder.embed(texts=[document['user'] for document in documents],
                                model='small',
                                num_workers=10,
                                shard_size=1000,)
    
    if len(embeddings) != len(documents):
        raise Exception("Embedding job failed")
    print("Embedding job complete.")
    
    response = atlas.map_embeddings(embeddings=np.array(embeddings),
                                    data=documents,
                                    colorable_fields=['sentiment'],
                                    is_public=True,
                                    map_name='Sentiment 140',
                                    map_description='A 10,000 point sample of the huggingface sentiment140 dataset embedded with the co:here small model.',
                                    organization_name=None, #defaults to your current user.
                                    num_workers=20)
    print(response)
    ```

=== "Output"

    ``` bash
    map='https://atlas.nomic.ai/map/ff2f89df-451e-49c4-b7a3-a608d7375961/f433cbd1-e728-49da-8c83-685cd613788b'
    job_id='b4f97377-e2aa-4305-8bc6-db7f5f6eeabf'
    index_id='46445e68-8c9f-470a-aa82-847e78c0f10e'
    ```