# Implementing Semantic Search for SQL Databases with Datasette

[Datasette](https://datasette.io/) is an open-source tool for exploring and publishing data. It allows you to create web interfaces for exploring databases with minimal effort. Developed by Simon Willison, a co-creator of the Django web framework, Datasette is similarily designed to be simple, lightweight, and easy to use. One of the things it greatly simplifies is the ability to quickly spin up "semantic search" against a SQLite database. 

Semantic search is a method for returning highly relevant results based on *meaning* rather than just keyword filtering. This allows users to find documents that may relate to their search term, even when the matching keywords are not present in the search query. This is ideal for those instances where we are trying to search in a general direction or where we don't yet know the right keywords to use.

I discussed this with a lawyer friend of mine and he immediately understood it as "vibe search." Fittingly, this is the same term that Willison used in his [excellent post](https://simonwillison.net/2023/Oct/23/embeddings/) that describes working with semantic search and embeddings in more detail. 

I'm going to walk through how I used Datasette to quickly create and deploy a functional semantic search engine on a database of 5,000 recipes. With this semantic search implementation, we will be able to search our database with the query phrase "a small levantine appetizer" and receive highly useful results.

The key point here is that you do not *need* a vector database or lots of tooling to quickly spin up powerful semantic search on a decent-sized data set.

## Tutorial

### Download dataset into project directory
We're going to work with a simple dataset that includes the title, ingredients, and instructions for 5,000 recipes. Grab the 5k-recipes.db file from this [recipe dataset repository](https://github.com/josephrmartinez/recipe-dataset).

Make a new project directory folder and move the 5k-recipes.db database file inside of this directory.


### Create and activate virtual environment
Navigate to your project directory and create and activate a virtual environment running python 3.11. Datasette requires Python 3.7 or higher.

```
python3.11 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
```
### Install datasette
It's generally a good practice to install Python packages within the virtual environment to keep dependencies isolated. This ensures that your project's dependencies won't interfere with other projects or the global Python environment. Use pip to install Datasette in your current virtual environment:

```pip install datasette```

Or follow [these instructions](https://docs.datasette.io/en/stable/installation.html) to install Datasette globally on your machine.

### Run Datasette
With the terminal inside of your project directory and your virtual environment activated, run the following command to launch the Datasette server with our 5k-recipes database:

```datasette 5k-recipes.db```

Open a browser and navigate to [http://127.0.0.1:8001](http://127.0.0.1:8001)

You should now be able to easily explore and apply SQL queries to the recipes table within the 5k-recipes database. 

### Enable full text search
Before we spin up semantic search, we are going to enable full text search on our recipes table. Eventually, we will be able to run comparisons between the two search methods and evaluate response time as well as query flexibility.

#### Install sqlite-utils
[sqlite-utils](https://datasette.io/tools/sqlite-utils), also by Simon Willison, is a CLI tool and Python library for manipulating SQLite databases. We're going to use sqlite-utils to [enable full-text search](https://sqlite-utils.datasette.io/en/stable/cli.html#configuring-full-text-search) across the Title, Ingredients, and Instructions columns in our recipes table:

```
pip install sqlite-utils
sqlite-utils enable-fts 5k-recipes.db recipes Title Ingredients Instructions
```

Now navigate back to our recipes table running on Datasette in the browser: [http://localhost:8001/5k-recipes/recipes](http://localhost:8001/5k-recipes/recipes)

Notice that there is now a search box.

Run a search for "comfort food."

We get only two results. This is because the phrase "comfort food" shows up in the instructions for each of these recipes: "enter comfort food heaven," and "serve with some rice or fresh bread for the ultimate comfort food." Surely there are more than just two comfort food options in this collection of 5,000 recipes?

There are. And to find those items, we are going to start by calculating embeddings for all of the recipes in our database.

### Calculate embeddings
An embedding serves as a condensed representation of text, capturing its nuanced meaning through a list of floating-point numbers. The OpenAI embedding model allows us to transform a given text into a 1,536-dimensional vector that encapsulates the intricate semantic features of the text. 

We will use the [openai-to-sqlite](https://datasette.io/tools/openai-to-sqlite) Datasette tool to efficiently calculate and store an embedding on each recipe in our SQLite database. 

```
pip install openai-to-sqlite
openai-to-sqlite embeddings 5k-recipes.db \
--sql 'select id, Title, Ingredients, Instructions from recipes' \
--table recipe_embeddings \
--token=sk-...<enter your API key here>...
```

This concatenates together the Title, Ingredients, and Instructions columns from the recipes table, runs them through the OpenAI embeddings API and stores the results in a new table called recipe_embeddings with the following schema:

```
CREATE TABLE [recipe_embeddings] (
   [id] INTEGER PRIMARY KEY,
   [embedding] BLOB
)
```

**Note:** this operation requires an OpenAI API key. Running this operation will take about two minutes and cost 1,909,410 tokens. That amounts to about $0.19 based on the current pricing for the ada v2 embedding model. Since I have already calculated these embeddings, you can skip this step and download the full-text search enabled database that also includes the embeddings table here: [https://www.kaggle.com/datasets/josephmdev/recipes/](https://www.kaggle.com/datasets/josephmdev/recipes/)

If you choose to download this resource, be sure to delete the old 5k-recipes.db file and replace it with the new resource. 

#### Create metadata file

When we enabled full text search on the recipes table, Datasette automatically enabled a search box for us to run queries against the table. In order to run semantic search queries in the Datasette web UI using the new recipe_embeddings table, we are going to create a [metadata](https://docs.datasette.io/en/stable/metadata.html) file with a ["canned query"](https://docs.datasette.io/en/stable/sql_queries.html#canned-queries) that uses some helpful functions from plugins that we will also install into Datasette.

Create a metadata.json file in your project folder:

```
{
    "title": "Recipe Search",
    "description": "Full text and semantic search on 5,000 recipes.",
    "databases": {
      "5k-recipes": {
        "source": "Recipe Dataset from Epicurious",
        "source_url": "https://github.com/josephrmartinez/recipe-dataset",
        "license": "CC BY-SA 3.0",
        "license_url": "https://creativecommons.org/licenses/by-sa/3.0/",
        "queries": {
        }
      }
    },
    "plugins": {
    }
  }
```
We will be adding information into the "queries" and "plugins" sections of the metadata file as we build out our custom query.

Close the terminal running your Datasette server and re-launch the server with the following command in order to restart the server but this time with the metadata:

```
datasette 5k-recipes.db --metadata metadata.json
```

Notice that the metadata above has been loaded into our Datasette web display: we see the database name, data source attribution, and so on. Now we are ready to build out the custom SQL query that will let us perform semantic search.

To perform the search operation, we will use two functions:
1. Calculate an embedding on the user's query
2. Perform a vector similarity calculation between the embedding of the user's query and all of the embeddings in our recipe_embeddings table.


### Function to calculate embedding on user's query

The [datasette-openai](https://datasette.io/plugins/datasette-openai) plugin allows us to use SQL functions within Datasette for calling the OpenAI APIs. Let's install this plugin so that we can use its openai_embeddings function to calculate an embedding on our user query:

```
datasette install datasette-openai
```

Having this plugin installed will allow us to use the openai_embeddings function in custom SQL queries:

openai_embedding(text, api_key)

This calls the OpenAI embedding endpoint and returns a binary object representing the floating point embedding for the provided text, in this case the user's query.

### Function to perform a vector similarity calculation

The function above gives us an embedding for the user's query. We can now perform a cosine similarity calculation between this query embedding and all of the values in our recipe_embeddings table to find the most similar results.

This is really exciting! Embeddings allow us to perform calculations on *meaning.* I'll never get over this.

A brute-force cosine similarity calculation in python looks like this:

```
def cosine_similarity(a, b):
    dot_product = sum(x * y for x, y in zip(a, b))
    magnitude_a = sum(x * x for x in a) ** 0.5
    magnitude_b = sum(x * x for x in b) ** 0.5
    return dot_product / (magnitude_a * magnitude_b)
```

This works, but it can be slow on larger datasets. To more efficiently perform this similarity search calculation, we are going to use the [Faiss](https://github.com/facebookresearch/faiss) library developed by Facebook AI Research. 

The [datasette-faiss](https://datasette.io/plugins/datasette-faiss) plugin enables us to use the Faiss library easily within Datasette. This plugin creates in-memory Faiss indexes for specified tables on startup, using an IndexFlatL2 Faiss index type. The tables to be indexed must have id and embedding columns. Our recipe_embeddings table is already formatted properly. Let's install the plugin:

```
datasette install datasette-faiss
```

In order for the datasette-faiss plugin to function properly, you must specify which tables should have indexes created for them by updating the "plugins" section of our metadata.json file:

{
    "plugins": {
        "datasette-faiss": {
            "tables": [
                ["5k-recipes", "recipe_embeddings"]
            ]
        }
    }
}

The plugin makes four new SQL functions available within Datasette. We will be using the following function in our custom query:

```
faiss_search_with_scores(database, table, embedding, k)
```

This function returns a JSON array of the k nearest neighbors and similarity score of each to the embedding found in the specified database and table.


### Develop custom SQL query

Now that we have installed the appropriate plugins and identified our two helper functions, we are ready to develop our cutom sql query to perform semantic search:

```
select
  value
from
  json_each(
    faiss_search_with_scores(
      '5krecipes',
      'embeddings',
      (
        select
          openai_embedding(:query, :openai_api_key)
      ),
      10
    )
  )
```

Describe what this does.

Now we just need to make two small updates. First, add "where length(coalesce(:query, '')) > 0" to prevent the query from running if the user hasn't added anything into the search box yet. If you do not add this, you will get this error message "user-defined function raised exception." Then, since we are going to put this query into our metadata.json file, the whole query should just be formatted as one long string.

Our custom sql query for embedding search should now look like this:
"select value from json_each(faiss_search_with_scores('5krecipes', 'embeddings', (select openai_embedding(:query, :openai_api_key)), 10)) where length(coalesce(:query, '')) > 0"

Update the metadata.json file queries section:

```
"queries": {
          "embeddingsearch": {
            "hide_sql": false,
            "sql": "SELECT ROUND(CAST(json_extract(value, '$[1]') AS REAL), 3) AS score, r.Title, r.Ingredients, r.Instructions FROM json_each(faiss_search_with_scores('5k-recipes', 'recipe_embeddings', (SELECT openai_embedding(:query, :openai_api_key)), 10)) AS json_data JOIN recipes AS r ON r.id = CAST(json_extract(json_data.value, '$[0]') AS INTEGER) WHERE length(coalesce(:query, ''))>0;"
          }
        }
```

### Run semantic search

It takes about 229 ms to run query my database of 5,000 recipes with the term 'Thanksgiving'. 

With "vibes-based search" I can also search for something much more conceptual, such as "a small levantine appetizer". If I try this query over on the recipes_fts table, I get the following response: "0 rows where search matches "a small levantine appetizer" sorted by rowid"

But with my custom query that creates an embedding on the query phrase and then uses FAISS to find the approximate nearest neighbors, I get "Sam's Spring Fattoush Salad", "spiced labneh", and eight other recipes. The word "levantine" does not show up at all in these recipes. Instead, the embedding gets at the underlying meaning in the word "levantine" and the similarity search algorithm helps us find the approximate nearest neighbors to this embedding space. We get some very useful results!