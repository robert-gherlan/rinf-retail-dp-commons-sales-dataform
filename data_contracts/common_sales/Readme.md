# Common Sales Monthly Reporting Data Contract

For this data contract, we define the schema and validation rules for the Common Sales Monthly Reporting dataset. This dataset is used for generating monthly sales reports across various regions and product categories.

We are using the [`datacontract`](https://datacontract.com/) tool to manage and validate the data contract.

[![Data Contract CLI](https://img.shields.io/badge/Data%20Contract-Valid-green)](https://github.com/datacontract/datacontract-cli)

## Install Dependencies

```bash
pip install -r requirements.txt
```

## Validate the data contract

```bash
datacontract lint common_sales_monthly_reporting_v1.yaml
```

## Test the data contract

```bash
datacontract test common_sales_monthly_reporting_v1.yaml
```

## Export the data contract as HTML

```bash
datacontract export --format html common_sales_monthly_reporting_v1.yaml --output common_sales_monthly_reporting_v1.html
```
