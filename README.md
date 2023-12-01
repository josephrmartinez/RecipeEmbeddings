Publish datasette
- Deploy datasettewith metadata, installed plugins, and .env link to api key
- Store api key as env on openai_embeddings plug in?
- install with plugins: datasette-openai, datasette-faiss

Run datasette with metadata: datasette 5krecipes.db --metadata metadata.json

View recipes_fts on index page
- Update metadata to set hidden=false on the recipes_fts table - NOT WORKING

Format response to vibesearch with LINK to recipes table
- Values in id column should LINK to recipes table



Create templates / separate UI site for datasette
- Create template the allows full text and vibe search on database

Connect with "similar recipes" data
- Create "similar" column, link to ten recipes. Or set up a link that displays the similar results.



DONE:

Download dataset of 15k+ recipes
Dataset:
https://www.kaggle.com/datasets/pes12017000148/food-ingredients-and-recipe-dataset-with-images
- Created excerpt version of database with just 5000 recipes


- Calculate embeddings of each recipe (combine title, ingredients, instructions): 
2 mins to calculate embeddings 
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
"Search embeddings using cosine similarity against a query." This does NOT use FAISS, but a custom implementation:
def cosine_similarity(a, b):
    dot_product = sum(x * y for x, y in zip(a, b))
    magnitude_a = sum(x * x for x in a) ** 0.5
    magnitude_b = sum(x * x for x in b) ** 0.5
    return dot_product / (magnitude_a * magnitude_b)

The output will be a list of 10 cosine similarity scores and content IDs.

openai-to-sqlite search 5krecipes.db 'thanksgiving'
returns ten dishes with cosine similarities ranging from 0.775(Thanksgiving Dinner for One) to 0.766 (Mustard Seed Gravy). A few of the recipes do contain the word 'Thanksgiving' but other results are included such as a vegan Tofurkey with Mushroom Stuffing and Gravy and Chili of Forgiveness. Every dish includes Turkey (or Tofurkey).
Running this from the command line took 2.62 seconds

- Use FAISS search on datasette:
"a highly regarded solutions for fast vector similarity calculations is FAISS, by Facebook AI research"

Install plugins: 
datasette install datasette-openai
    - This allows me to use the openai_embeddings function.
    openai_embedding(text, api_key)
    This calls the OpenAI embedding endpoint and returns a binary object representing the floating point embedding for the provided text.

datasette install datasette-faiss
faiss_search_with_scores(database, table, embedding, k)
Takes the same arguments as above, but the return value is a JSON array of pairs, each with an ID and a score

Write custom datasette sql query:
select value from json_each(faiss_search_with_scores('5krecipes', 'embeddings', (select openai_embedding(:query, :openai_api_key)), 10))

Add "where length(coalesce(:query, '')) > 0" to prevent the query from running with an empty query. If you do not add this, you will get this error message "user-defined function raised exception". 
"adding where length(coalesce(:query, '')) > 0 to the query means that the query won’t run if the user hasn’t entered any text into the search box." 

select value from json_each(faiss_search_with_scores('5krecipes', 'embeddings', (select openai_embedding(:query, :openai_api_key)), 10)) where length(coalesce(:query, '')) > 0

It takes about 229 ms to run query my database of 5,000 recipes with the term 'Thanksgiving'. 
Compare this to the 2.62 seconds it took to run the query using openai-to-sqlite search in the command line with the custom cosine similarity search function.
Both queries returned the exact same ten recipes. The scores, however, were different. 
The openai-to-sqlite search values ranged from 0.775 to 0.766, the faiss_search_with_scores values ranged from 0.449 to 0.468. Notably, the values were associated with the IDs in the opposite order for each method. Check to see what this means or if additional score formatting is required. 

With "vibes-based search" I can also search for something much more conceptual, such as "a small levantine appetizer". If I try using full-text search for this query, I get the following response: "0 rows where search matches "a small levantine appetizer" sorted by rowid"
But with my custom query that creates an embedding on the query phrase and then uses FAISS to find the approximate nearest neighbors, I get "Sam's Spring Fattoush Salad", "spiced labneh", and eight other recipes. The word "levantine" does not show up at all in these recipes. Instead, the embedding helps us get at the underlying meaning in the query and ...


Deploy the app:
datasette publish fly 5krecipes.db --app="recipe-embeddings" -m metadata.json --install=datasette-openai --install=datasette-faiss --install=datasette-scale-to-zero --generate-dir /Users/admin/Desktop/programming/repos/RecipeEmbeddings/deploy



- I was able to deploy the app along with the metadata on the fly.io "free tier" with no problem: datasette publish fly 5krecipes.db --app="recipe-embeddings" -m metadata.json

- But when I tried deploying the app with the plugins, I received an error: 
"Your “recipe-embeddings” application hosted on Fly.io crashed because it ran out of memory. Specifically, the instance 3287174c57e585. Adding more RAM to your application might help!"








- Spin up RecipeVector frontend that allows users to run queries against the DB:



    1. Compile Our Database: We’ll extract our embeddings table from the llm database and load it into a new SQLite database. We then load this database with a separate table containing image and product URLs for each faucet.

    2. Install Some Plugins: Install the datasette plugins we’ll need: dataSETTE-template-sql, datasette-faiss, datasette-publish-fly, and datasette-scale-to-zero.
    
    3. Design Our Templates: We want two pages for our app. The first serves up a random set of faucet images. Clicking one of these faucets sends you to our second page, which shows you faucets similar to the one you clicked. The first will be our home page, the second will be a “canned query” page that uses datasette-faiss to query our embeddings and accepts a faucet ID as an input parameter. We’ll build both pages using custom templates, overriding datasette’s default styles. For our home page, we’ll execute our query using data-template-sql.

    4. Deploy Our App: We’ll use Fly.io, datasette-publish-fly and datasette-scale-to-zero.



- Use Principal Component Analysis to create an interactive 2D visualization: https://simonwillison.net/2023/Oct/23/embeddings/#visualize-in-2d-with-principal-component-analysis




semantic search / vibe search
—"find documents that match the meaning of the user’s search term, even if the matching keywords are not present."