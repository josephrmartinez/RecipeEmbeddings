TODO:
- Update metadata to set hidden=false on the recipes_fts table - NOT WORKING

- Enable semantic search on recipes table

- Create "similar" column, link to ten recipes. Or set up a link that displays the similar results.

Run datasette with metadata: datasette 5krecipes.db --metadata metadata.json --setting sql_time_limit_ms 5500




- Calculate 5k embeddings of recipes: 
2 mins to calculate embeddings (combine title, ingredients, instructions)
Total tokens used: 1,909,617
Less than $0.20 ???
5,001 rows

- Calculate and save top ten similarity calculations for every item in the database
Time for completion: 2:03:42 

CREATE TABLE [similarities] (
   [id] TEXT,
   [other_id] TEXT,
   [score] FLOAT,
   PRIMARY KEY ([id], [other_id])
);

This table contains ten rows for each id. These are the top ten similarity scores for each id.
50,010 rows


- Seach embeddings with the search command: openai-to-sqlite search 5krecipes.db 'search term'
The output will be a list of 10 cosine similarity scores and content IDs.

openai-to-sqlite search 5krecipes.db 'thanksgiving'
returns ten dishes with cosine similarities ranging from 0.775(Thanksgiving Dinner for One) to 0.766 (Mustard Seed Gravy). A few of the recipes do contain the word 'Thanksgiving' but other results are included such as a vegan Tofurkey with Mushroom Stuffing and Gravy and Chili of Forgiveness. Every dish includes Turkey (or Tofurkey).

- Search for similar content with the similar command:
openai-to-sqlite similar 5krecipes.db '1'; (the last value in quotes is the id).
This returns the ten rows associated with the item ID '1'.
[id] TEXT,
[other_id] TEXT,
[score] FLOAT,



- Spin up RecipeVector frontend that allows users to run queries against the DB:



    1. Compile Our Database: We’ll extract our embeddings table from the llm database and load it into a new SQLite database. We then load this database with a separate table containing image and product URLs for each faucet.

    2. Install Some Plugins: Install the datasette plugins we’ll need: dataSETTE-template-sql, datasette-faiss, datasette-publish-fly, and datasette-scale-to-zero.
    
    3. Design Our Templates: We want two pages for our app. The first serves up a random set of faucet images. Clicking one of these faucets sends you to our second page, which shows you faucets similar to the one you clicked. The first will be our home page, the second will be a “canned query” page that uses datasette-faiss to query our embeddings and accepts a faucet ID as an input parameter. We’ll build both pages using custom templates, overriding datasette’s default styles. For our home page, we’ll execute our query using data-template-sql.

    4. Deploy Our App: We’ll use Fly.io, datasette-publish-fly and datasette-scale-to-zero.



- Use Principal Component Analysis to create an interactive 2D visualization: https://simonwillison.net/2023/Oct/23/embeddings/#visualize-in-2d-with-principal-component-analysis


Dataset:
https://www.kaggle.com/datasets/pes12017000148/food-ingredients-and-recipe-dataset-with-images
- Created excerpt version of database with just 5000 recipes


semantic search
—"find documents that match the meaning of the user’s search term, even if the matching keywords are not present."
def search(db_path, query, token, table_name, count):
    """
    Search embeddings using cosine similarity against a query.

    The query you pass will be embedded using the OpenAI API,
    then the closest matching records from the table will be shown.
    """
    if not token:
        raise click.ClickException(
            "OpenAI API token is required, use --token=x or set the "
            "OPENAI_API_KEY environment variable"
        )
    db = sqlite_utils.Database(db_path)
    table = db[table_name]
    if not table.exists():
        raise click.ClickException(f"Table {table_name} does not exist")
    # Fetch the embedding for the query
    response = httpx.post(
        "https://api.openai.com/v1/embeddings",
        headers={
            "Authorization": f"Bearer {token}",
            "Content-Type": "application/json",
        },
        json={"input": query, "model": "text-embedding-ada-002"},
    )
    response.raise_for_status()
    data = response.json()
    vector = data["data"][0]["embedding"]
    # Now calculate cosine similarity with everything in the database table
    other_vectors = [(row["id"], decode(row["embedding"])) for row in table.rows]
    results = [
        (id, cosine_similarity(vector, other_vector))
        for id, other_vector in other_vectors
    ]
    results.sort(key=lambda r: r[1], reverse=True)
    for id, score in results[:count]:
        print(f"{score:.3f} {id}")