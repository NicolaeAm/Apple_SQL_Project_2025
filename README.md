# Apple’s global retail and product performance project.

![Apple Logo](https://github.com/NicolaeAm/Apple_SQL_Project_2025/blob/main/apple.webp)

## Overview 

Project Overview
This project is an SQL-based analysis of Apple’s global retail and product performance. Built on a custom relational database with over 1 million sales records, it examines key business metrics including store performance, product category trends, warranty claims, and seasonal sales dynamics.

## Objective

The main goal of this project is to analyze Apple’s sales and warranty data to uncover key insights about:
  -Revenue distribution across countries and stores
  -Product category performance and demand trends
  -Warranty claim patterns and reliability metrics
  -Store growth ratios and performance benchmarking
  -Seasonality and correlation analysis for sales optimization

## Dataset

The dataset used in this project represents Apple sales environment, containing over one million transaction records.
It models a realistic business structure with five relational tables that store data about Apple’s stores, products, categories, sales, and warranty operations.

## Dataset schema

```sql
CREATE TABLE stores(
    store_id VARCHAR(5) PRIMARY KEY,
    store_name VARCHAR(30),
    city VARCHAR(25),
    country VARCHAR(25)
);

CREATE TABLE category(
    category_id VARCHAR(10) PRIMARY KEY,
    category_name VARCHAR(20)
);

CREATE TABLE products(
    product_id VARCHAR(10) PRIMARY KEY,
    product_name VARCHAR(35),
    category_id VARCHAR(10),
    launch_date DATE,
    price FLOAT,
    CONSTRAINT fk_category FOREIGN KEY (category_id) REFERENCES category(category_id)
);

CREATE TABLE sales(
    sale_id VARCHAR(15) PRIMARY KEY,
    sale_date DATE,
    store_id VARCHAR(10),
    product_id VARCHAR(10),
    quantity INT,
    CONSTRAINT fk_store FOREIGN KEY (store_id) REFERENCES stores(store_id),
    CONSTRAINT fk_product FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE warranty(
    claim_id VARCHAR(10) PRIMARY KEY,
    claim_date DATE,
    sale_id VARCHAR(15),
    repair_status VARCHAR(15),
    CONSTRAINT fk_orders FOREIGN KEY (sale_id) REFERENCES sales(sale_id)
);
```







```
