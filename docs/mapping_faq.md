# Mapping FAQ
Frequently asked questions about Atlas maps.

## Mapping Latency

Map creation latency once Nomic has received your embeddings.

|  Number of datums/embeddings  |  Map availability latency (s)  |
|:-----------------------------:|:------------------------------:|
|            10,000             |            instant             |
|        10,001 - 99,999        |             10-40              |
|       100,000 - 499,999       |             40-180             |
|       500,000 - 999,999       |            180-600             |
|     1,000,000 - 9,999,999     |              600+              |


## Who can see my maps?
When you create a map, you can toggle it as private or public. Private maps are only
accessible by authenticated individuals in your Nomic organization. Public maps are accessible by anyone with a link.

=== "Atlas Client Private Map Example"

    ``` py title="map_embeddings_private.py"
    from nomic import AtlasClient
    import numpy as np
    
    atlas = AtlasClient()
    
    num_embeddings = 10000
    embeddings = np.random.rand(num_embeddings, 256)
    
    response = atlas.map_embeddings(embeddings=embeddings,
                                    is_public=False,
                                    organization_name='my_organization'
                                    )
    print(response)
    ```

## Making maps under an organization
If you are added to a Nomic organization by someone (such as your employer), you can create projects under them
by specifying an `organization_name` in the `map_embedding` method of the AtlasClient. By default, projects are
made under your own account.

## How do I make maps of a dataset I have already uploaded?
You need to make a new index on the project you have uploaded your data to.
See [How does Atlas work?](how_does_atlas_work.md) for details.

## Disabling logging
Nomic utilizes the `loguru` module for logging. We recognize that logging can sometimes be annoying.
You can disable or change the logging level by including the following snippet at the top of any script.
```py
from loguru import logger
import sys
logger.remove(0)
logger.add(sys.stderr, level="ERROR", filter='nomic')

```