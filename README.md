TODO:
- Calculate and save similarity calculations for every item in the database
Time for completion: 


- Seach embeddings with the search command: openai-to-sqlite search 5krecipes.db 'search term'
The output will be a list of 10 cosine similarity scores and content IDs.

- Search for similar content with the similar command:
openai-to-sqlite similar 5krecipes.db '<content identifier(id?? title??)>'




- Spin up RecipeVector frontend that allows users to run queries against the DB:

    1. Compile Our Database: We’ll extract our embeddings table from the llm database and load it into a new SQLite database. We then load this database with a separate table containing image and product URLs for each faucet.

    2. Install Some Plugins: Install the datasette plugins we’ll need: data-template-sql, datasette-faiss, datasette-publish-fly, and datasette-scale-to-zero.
    
    3. Design Our Templates: We want two pages for our app. The first serves up a random set of faucet images. Clicking one of these faucets sends you to our second page, which shows you faucets similar to the one you clicked. The first will be our home page, the second will be a “canned query” page that uses datasette-faiss to query our embeddings and accepts a faucet ID as an input parameter. We’ll build both pages using custom templates, overriding datasette’s default styles. For our home page, we’ll execute our query using data-template-sql.

    4. Deploy Our App: We’ll use Fly.io, datasette-publish-fly and datasette-scale-to-zero.



- Use Principal Component Analysis to create an interactive 2D visualization: https://simonwillison.net/2023/Oct/23/embeddings/#visualize-in-2d-with-principal-component-analysis


Dataset:
https://www.kaggle.com/datasets/pes12017000148/food-ingredients-and-recipe-dataset-with-images
- Created excerpt version of database with just 5000 recipes


semantic search
—"find documents that match the meaning of the user’s search term, even if the matching keywords are not present."