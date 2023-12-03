# Using Datasette to Implement Semantic Search Against a SQLite Database

[Datasette](https://datasette.io/) is an open-source tool for exploring and publishing data. It allows you to create web interfaces for exploring databases with minimal effort. Developed by Simon Willison, the co-creator of the Django web framework, Datasette is similarily designed to be simple, lightweight, and easy to use. One of the things it greatly simplifies is the ability to quickly spin up "semantic search" against a SQLite database. 

Semantic search is a method for returning highly relevant results based on *meaning* rather than just keyword filtering. This allows us to find documents that may relate to the underlying meanings and associations in the user's search term, even when the matching keywords are not present. 

This is super exciting!

I discussed this with a lawyer friend of mine and he immediately understood it as "vibe search." Fittingly, this is the same term that Willison used in his [excellent post](https://simonwillison.net/2023/Oct/23/embeddings/) that describes working with semantic search and embeddings in more detail. 


I'm going to walk through how I used Datasette to quickly create and deploy a functional semantic search engine on a database of 5,000 recipes. Semantic search will allow us to go beyond simple keyword lookup and use terms like "................."  The key point here is that you do not *need* a vector database or lots of tooling to quickly spin up powerful semantic search on a decent-sized data set.

## Tutorial

## Download dataset into project directory
Grab the 5k-recipes.db file from this [free, simple, open recipe dataset repository](https://github.com/josephrmartinez/recipe-dataset). 5k-recipes.db is a simple dataset that includes the title, ingredients, and instructions for 5,000 recipes.

Create a project directory and move the 5k-recipes.db database file inside of this directory.


## Create and activate virtual environment
Navigate to your project directory and create and activate a virtual environment running python 3.11. Datasette requires Python 3.7 or higher.

```python3.11 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
```
## Install datasette
It's generally a good practice to install Python packages within the virtual environment to keep dependencies isolated. This ensures that your project's dependencies won't interfere with other projects or the global Python environment. Use pip to install Datasette in your current virtual environment:

```pip install datasette```

Or follow [these instructions](https://docs.datasette.io/en/stable/installation.html) to install Datasette globally on your machine.

## Run datasette
With the terminal inside of your project directory and your virtual environment activated, run the following command to launch the Datasette server with our 5k-recipes database:

```datasette 5k-recipes.db```

Open a browser and navigate to [http://127.0.0.1:8001](http://127.0.0.1:8001)

You should be able to explore the 5k-recipes database easily now within the Datasette framework. This will allo you to 

## Enable full text search
Before we spin up semantic search, we are going to enable full text search on our recipes table. Enevtually, we will be able to run comparisons between the two search methods and evaluate response time as well as query flexibility.

### Install sqlite-utils
[sqlite-utils](https://datasette.io/tools/sqlite-utils), also by Simon Willison, is a CLI tool and Python library for manipulating SQLite databases. We're going to use sqlite-utils to [enable full-text search](https://sqlite-utils.datasette.io/en/stable/cli.html#configuring-full-text-search) across the Title, Ingredients, and Instructions columns in our recipes table:

```pip install sqlite-utils
sqlite-utils enable-fts 5k-recipes.db recipes Title Ingredients Instructions
```

Now navigate back to our recipes table running on Datasette in the browser: [http://localhost:8001/5k-recipes/recipes](http://localhost:8001/5k-recipes/recipes)

Notice that there is now a search box!

Run a search for "comfort food."
Notice that we get only two results and that the phrase "comfort food" shows up in the instructions for each of these recipes: "enter comfort food heaven," and "serve with some rice or fresh bread for the ultimate comfort food." But surely there are more than just two comfort food options in this collection of 5,000 recipes?

There are. And to find those items, we are going to start by calculating embeddings for all of the recipes in our database.

## Calculate embeddings

One of the wonderful things about Datasette is the ability to extend functionality by installing [plugins](https://docs.datasette.io/en/stable/plugins.html). 

The [datasette-openai](https://datasette.io/plugins/datasette-openai) plugin allows us to use SQL functions within Datasette for calling the OpenAI APIs. Let's install this plugin and use it to efficiently calculate and store an embedding for each recipe in our database:

```
datasette install datasette-openai

```

- Calculate embeddings of each recipe (combine title, ingredients, instructions): 


Install plugin: 
datasette install datasette-openai
    - This allows me to use the openai_embeddings function.
    openai_embedding(text, api_key)
    This calls the OpenAI embedding endpoint and returns a binary object representing the floating point embedding for the provided text.

2 mins to calculate embeddings 
Total tokens used: 1,909,617
Less than $0.20 ???
5,001 rows

- UPLOAD EMBEDDINGS TABLE TO github. Offer download link. Offer instructions on how to join table to 5k-recipes.db


- Use FAISS search on datasette:
"a highly regarded solutions for fast vector similarity calculations is FAISS, by Facebook AI research"


Install plugin: 
datasette install datasette-faiss
faiss_search_with_scores(database, table, embedding, k)
Takes the same arguments as above, but the return value is a JSON array of pairs, each with an ID and a score

Write custom datasette sql query:
select value from json_each(faiss_search_with_scores('5krecipes', 'embeddings', (select openai_embedding(:query, :openai_api_key)), 10))

Add "where length(coalesce(:query, '')) > 0" to prevent the query from running if the user hasn't added anything into the search box yet. If you do not add this, you will get this error message "user-defined function raised exception."

Our custom sql query for embedding search should now look like this:
select value from json_each(faiss_search_with_scores('5krecipes', 'embeddings', (select openai_embedding(:query, :openai_api_key)), 10)) where length(coalesce(:query, '')) > 0

It takes about 229 ms to run query my database of 5,000 recipes with the term 'Thanksgiving'. 

With "vibes-based search" I can also search for something much more conceptual, such as "a small levantine appetizer". If I try this query over on the recipes_fts table, I get the following response: "0 rows where search matches "a small levantine appetizer" sorted by rowid"

But with my custom query that creates an embedding on the query phrase and then uses FAISS to find the approximate nearest neighbors, I get "Sam's Spring Fattoush Salad", "spiced labneh", and eight other recipes. The word "levantine" does not show up at all in these recipes. Instead, the embedding gets at the underlying meaning in the word "levantine" and the similarity search algorithm helps us find the approximate nearest neighbors to this embedding space. We get some very useful results!
















# Install Datasette and sqlite-utils within the virtual environment



## Use sqlite-utils to import CSV into a SQLite database







Download dataset of 15k+ recipes
Dataset:
https://www.kaggle.com/datasets/pes12017000148/food-ingredients-and-recipe-dataset-with-images

- Created excerpt version of database with just 5000 recipes


- Calculate embeddings of each recipe (combine title, ingredients, instructions): 



2 mins to calculate embeddings 
Total tokens used: 1,909,617
Less than $0.20 ???
5,001 rows



Full-text search functionality is easy to configure on a SQLite database by using the appropriate [virtual table module](https://www.sqlite.org/fts5.html). This allows us to create a search engine to "efficiently search a large collection of documents for the subset that contain one or more instances of a search term."

Semantic search 

