# Common Sales Monthly Reporting Data Contract

For this data contract, we define the schema and validation rules for the derived Recipe Reviews dataset.

We are using the [`datacontract`](https://datacontract.com/) tool to manage and validate the data contract.

[![Data Contract CLI](https://img.shields.io/badge/Data%20Contract-Valid-green)](https://github.com/datacontract/datacontract-cli)

## Source Data

The source data consists of two tables: `recipes` and `reviews` located in the `ingestion` BigQuery dataset..

## Prepare the prepared data in a nested structure

```sql
CREATE SCHEMA IF NOT EXISTS `rinf-retail-recipe-reviews.prepared_recipe_reviews` OPTIONS (location = 'EU');

CREATE OR REPLACE VIEW `prepared_recipe_reviews.recipe_reviews_nested` AS
SELECT
  recipe.*,
  ARRAY_AGG(
    STRUCT(
      review.ReviewId,
      review.AuthorId,
      review.AuthorName,
      review.Rating,
      review.Review,
      review.DateSubmitted,
      review.DateModified
    )
  ) AS Reviews
FROM `ingestion.recipes` recipe
LEFT JOIN `ingestion.reviews` review USING (RecipeId)
GROUP BY ALL
```

## Prepare the derived data

```sql
CREATE SCHEMA IF NOT EXISTS `rinf-retail-recipe-reviews.derived_recipe_reviews` OPTIONS (location = 'EU');

CREATE OR REPLACE TABLE `derived_recipe_reviews.recipe_reviews_v1`
(
  recipe_id INT64 OPTIONS(description="Unique identifier for each record"),
  recipe_name STRING OPTIONS(description="The name of the recipe."),
  recipe_category STRING OPTIONS(description="The category of the recipe."),
  recipe_date_published TIMESTAMP OPTIONS(description="The timestamp when the recipe was published."),
  recipe_year_published INT64 OPTIONS(description="The calendar year when the recipe was published."),
  recipe_author_id INT64 OPTIONS(description="The name of the author."),
  recipe_author_name STRING OPTIONS(description="The full name of the author of the recipe."),
  number_of_reviews INT64 OPTIONS(description="The number of reviews for the recipe."),
  average_review_rating FLOAT64 OPTIONS(description="The average rating of the recipe based on all reviews.")
)
AS SELECT
  CAST(RecipeId AS INT64) AS recipe_id,
  Name AS recipe_name,
  RecipeCategory AS recipe_category,
  DatePublished AS recipe_date_published,
  EXTRACT(YEAR FROM DatePublished) AS recipe_year_published,
  s.AuthorId AS recipe_author_id,
  s.AuthorName AS recipe_author_name,
  COUNT(DISTINCT review.ReviewId) AS number_of_reviews,
  COALESCE(ROUND(AVG(review.Rating), 1), 0) AS average_review_rating
FROM `prepared_recipe_reviews.recipe_reviews_nested` s, UNNEST(Reviews) AS review
GROUP BY ALL
```

## Configure data contract authentication for BigQuery

Access the following link to set up authentication for the `datacontract` CLI tool: [Authentication Guide](https://gpt.datacontract.com/sources/cli.datacontract.com/#bigquery).

## Install Dependencies

```bash
pip install -r requirements.txt
```

## Data contract tooling

[Data contract tooling](https://datacontract.com/#tooling)

## Create a data contract using import

```bash
datacontract import --bigquery-project rinf-retail-recipe-reviews --bigquery-dataset derived_recipe_reviews --bigquery-table recipe_reviews_v1 --format bigquery --output datacontract-recipe-reviews-import-v1.yaml
```

## Create an empty data contract

```bash
datacontract init datacontract-recipe-reviews-v1.yaml
```

## Validate the data contract

```bash
datacontract lint datacontract-recipe-reviews-v1.yaml
```

## Test the data contract

```bash
datacontract test datacontract-recipe-reviews-v1.yaml
```

## Export the data contract as HTML

```bash
datacontract export --format html datacontract-recipe-reviews-v1.yaml --output datacontract-recipe-reviews-v1.html
```

## Data COntract CLI Usage

For more information on using the Data Contract CLI, refer to the [Data Contract CLI Documentation](https://github.com/datacontract/datacontract-cli/tree/main?tab=readme-ov-file#usage).
