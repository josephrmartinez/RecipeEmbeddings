TODO:
- Populate an ID column for the 5krecipes.db

- Import data into SQLite, clean it, explore using Datasette: https://datasette.io/tutorials/clean-data

- install llm

- Use llm multi-embed to calculate and store multiple embeddings: https://llm.datasette.io/en/stable/embeddings/cli.html#llm-embed-multi


- Store and serve related documents with openai embeddings: https://til.simonwillison.net/llms/openai-embeddings-related-content (create embeddings for each recipe, store in db)
- Spin up RecipeVector frontend that allows users to run Word2Vec style queries against DB
- Use Principal Component Analysis to create an interactive 2D visualization: https://simonwillison.net/2023/Oct/23/embeddings/#visualize-in-2d-with-principal-component-analysis


Dataset:
https://www.kaggle.com/datasets/pes12017000148/food-ingredients-and-recipe-dataset-with-images
- Created excerpt version of database with just 5000 recipes


semantic search
—"find documents that match the meaning of the user’s search term, even if the matching keywords are not present."