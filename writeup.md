Using Datasette to Implement Semantic Search Against a SQLite Database

[Datasette](https://datasette.io/) is an open-source tool for exploring and publishing data. It allows you to create web interfaces for exploring databases with minimal effort. Developed by Simon Willison, the co-creator of the Django web framework, Datasette is similarily designed to be simple, lightweight, and easy to use. One of the things it greatly simplifies is the ability to quickly spin up "semantic search" against a SQLite database. 

Semantic search is a method for returning highly relavant results based on *meaning* rather than just simple keyword filtering. In simplified terms, imagine you generate an associative meaning map around every item in a database (e.g. blog posts, documents, transcripts, articles). Semantic search would allow you to explore these resources based on their meaning maps, not just tags and keywords. This would allow you to search a recipe database with a query "comfort food" even if no items were actually tagged with this term.   

I discussed this with a lawyer friend of mine and he immediately understood it as "vibe search." Fittingly, this is the same term that Simon Willison used in his [excellent post](https://simonwillison.net/2023/Oct/23/embeddings/) that describes working with semantic search and embeddings in more detail. 

This is super exciting!

I'm going to walk through how I used Datasette to quickly create and deploy a functional semantic search engine on a database of 5,000 recipes. The key point here is that you do not need a vector database or lots of tooling to quickly spin up powerful semantic search on a decent-sized data set.

# Tutorial

## Download the dataset
Grab the 5k-recipes.db file from this [free, simple, open recipe dataset repository](https://github.com/josephrmartinez/recipe-dataset). 5k-recipes.db is a simple dataset that includes the title, ingredients, and instructions for 5,000 recipes.

## Create directory, activate virtual environment
Create a directory for this project and move the 5k-recipes.db database file inside of this directory.

Navigate to your project directory and create and activate a virtual environment running python 3.11. Datasette requires Python 3.7 or higher.

# Create and activate virtual environment
It's generally a good practice to install Python packages, including Datasette and sqlite-utils, within the virtual environment to keep dependencies isolated. This ensures that your project's dependencies won't interfere with other projects or the global Python environment.

```python3.11 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
```

- Install datasette

- Calculate embeddings of each recipe (combine title, ingredients, instructions): 
2 mins to calculate embeddings 
Total tokens used: 1,909,617
Less than $0.20 ???
5,001 rows


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


With "vibes-based search" I can also search for something much more conceptual, such as "a small levantine appetizer". If I try using full-text search for this query, I get the following response: "0 rows where search matches "a small levantine appetizer" sorted by rowid"
But with my custom query that creates an embedding on the query phrase and then uses FAISS to find the approximate nearest neighbors, I get "Sam's Spring Fattoush Salad", "spiced labneh", and eight other recipes. The word "levantine" does not show up at all in these recipes. Instead, the embedding helps us get at the underlying meaning in the query and ...














# Install Datasette and sqlite-utils within the virtual environment

[sqlite-utils](https://datasette.io/tools/sqlite-utils), also by Simon Willison, is a CLI tool and Python library for manipulating SQLite databases. We're going to use sqlite-utils to extract the data we want from our CSV file and populate the SQLite database that we will then be working on with Datasette:

```pip install datasette sqlite-utils```

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

