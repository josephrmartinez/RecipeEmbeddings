Using Datasette to Implement Semantic Search Against a SQLite Database

Datasette is an open-source tool for exploring and publishing data. It allows you to create web interfaces for exploring databases with minimal effort. Developed by Simon Willison, Datasette is designed to be simple, lightweight, and easy to use. One of the things it greatly simplifies is the ability to quickly spin up "semantic search" against a SQLite database. 

Semantic search is a method for returning highly relavant results based on *meaning* rather than just simple keyword filtering. I discussed this with a lawyer friend of mine and he immediately understood it as "vibe search." Fittingly, this is the same term that Simon Willison used in his [excellent post](https://simonwillison.net/2023/Oct/23/embeddings/) that described working with embeddings in more detail. 

This is super exciting!

I'm going to walk through how I used Datasette to quickly create and deploy a functional semantic search engine on a database of 5,000 recipes. The key point here is that you do not need a vector database or lots of tooling to spin up semantic search on a decent-sized data set.

## Download the dataset
For this project, I started with an open-source dataset from Kaggle. The [Food Ingredients and Recipes Dataset with Images](https://www.kaggle.com/datasets/pes12017000148/food-ingredients-and-recipe-dataset-with-images) is 216 mb and includes over 13,000 recipes as well as images. For this project, I created a smaller dataset with just 5,000 recipes and no images. That file is just 20 mb and can be downloaded from here.


## Create directory, activate virtual environment
Create a directory for this project and move the CSV recipe dataset inside of this directory.

Navigate to your project directory and create and activate a virtual environment running python 3.11. Datasette requires Python 3.7 or higher.

# Create and activate virtual environment
It's generally a good practice to install Python packages, including Datasette and sqlite-utils, within the virtual environment to keep dependencies isolated. This ensures that your project's dependencies won't interfere with other projects or the global Python environment.

```python3.11 -m venv venv
source venv/bin/activate  # On Windows, use `venv\Scripts\activate`
```
# Install Datasette and sqlite-utils within the virtual environment

sqlite-utils, also by Simon Willison, is a CLI tool and Python library for manipulating SQLite databases. We're going to use sqlite-utils to extract the data we want from our CSV file and populate the SQLite database that we will then be working on with Datasette:

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

