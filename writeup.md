Using Datasette to Implement Semantic Search Against a SQLite Database

[Datasette](https://datasette.io/) is an open-source tool for exploring and publishing data. It allows you to create web interfaces for exploring databases with minimal effort. Developed by Simon Willison, the co-creator of the Django web framework, Datasette is similarily designed to be simple, lightweight, and easy to use. One of the things it greatly simplifies is the ability to quickly spin up "semantic search" against a SQLite database. 

Semantic search is a method for returning highly relavant results based on *meaning* rather than just simple keyword filtering. In simplified terms, imagine you generate an associative meaning map around every item in a database (e.g. blog posts, documents, transcripts, articles). Semantic search would allow you to explore these resources based on their meaning maps, not just tags and keywords. We could search a recipe database with a query "comfort food" even if we never tagged any items with this term. I discussed this with a lawyer friend of mine and he immediately understood it as "vibe search." Fittingly, this is the same term that Simon Willison used in his [excellent post](https://simonwillison.net/2023/Oct/23/embeddings/) that describes working with embeddings in more detail. 

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

