<!-- cspell:ignore astype groupby nunique openpyxl Menlo -->

# Python Pandas Code snippets

```python

# ========== Import ==========

import pandas as pd
# import csv file to df
df = pd.read_csv("demo.csv")
# import excel file to df
df = pd.read_excel("demo.xlsx")

# ========== Concatenate ==========

df["address"] = df["address"] + ", " + df["city"] + ", " + df["state"] + " " + df["zip"].astype("str")
# option 2
df["address"] = df[["address", "city", "state", "zip"]].apply(lambda x: ", ".join(x), axis=1)
# note: if one of the column is not of type "str", convert it first to "str", like this:
df["zip"] = df["zip"].astype("str")

# ========== Split ==========

df["last name"] = df["full name"].str.split(", ").str[0]
df["first name"] = df["full name"].str.split(", ").str[1]

# ========== Replace a substring ==========

df["address"] = df["address"].str.replace("St", "Street")

# replace substring in multiple columns
df[["address", "city"]] = df[["address", "city"]].replace("a", "@", regex=True)
# note: you have to specify "regex=True". if not, only a whole string will be replaced, not a substring.

# replace substring in a whole data-frame
df = df.replace("a", "@", regex=True)

# multiple replacements
df["address"] = df["address"].replace({"St": "Street", "Ave": "Avenue"}, regex=True)

# replace substring in headers
df.columns = df.columns.str.replace("c", "@")
# you can also use "regex=True" with this
df.columns = df.columns.str.replace(r"\s{2,}", " ", regex=True)

# ========== Trim and remove double spaces ==========

df = df.replace(r"\s{2,}", " ", regex=True)
df = df.replace(r"^\s|\s$", "", regex=True)
# note: this doesn't work for headers

# trim headers
df = df.rename(str.strip, axis=1)

# ========== Rename Columns ==========

df = df.rename({"address": "address1", "city": "city1"}, axis=1)

# ========== add columns to a data-frame ==========

df[["column1", "column2"]] = "" # this will add 2 columns with all values as empty strings.

# to add column from a list even if the list contains columns that are already in the df, use a list comprehension. otherwise it will override the original column
insert_columns = ["address", "column1", "column2"]
df[[c for c in insert_columns if c not in df.columns]] = ""

# ========== drop columns ==========

...

# ========== filter a data-frame ==========

df = df[df["state"] == "TX"]
# or
df = df[df["amount"] > 5]

# ========== Grouping ==========

# When grouping a column, it returns a "GroupBy" object. To convert it back to a DataFrame, you need to aggregate one or more of the other columns and then reset the index. If you just want the count of all other columns, you can use `size()`, which will return a Series. To convert it back to a DataFrame, reset the index and pass the parameter `name=name of count column`.

# This will return the state column and the sum of the amount column.
df = df.groupby("state")[["amount"]].sum()  # df is a DataFrame. (Grouping and Aggregation in 1 line)
df = df.reset_index()

# Multiple aggregations.
df = df.groupby("address")  # df is a GroupBy object
df = df.agg({"state": "nunique", "amount": "sum"})  # df is converted back to a DataFrame
df = df.reset_index()

# Group by 2 columns and the count of all other rows.
df = df.groupby(["address", "state"]).size()  # df is a Series
df = df.reset_index(name="count of rows")  # df is converted back to a DataFrame

# ========== Export ==========

df.to_csv("demo.csv", index=False)

# ========== Styling ==========
style = [
    """\
color: green;\
background-color: #D3D3D3;\
font-family: Menlo;\
font-style: italic;\
font-size: 18px\
"""
]

df_styled = df.style.apply(lambda x: style * len(x))

# then export
df_styled.to_excel("styled_data.xlsx", engine="openpyxl", index=False)

# conditional formatting (applies this style where the "state" = "TX")
df_styled = df.style.apply(lambda x: df["state"].map({"TX": "color: red"}))
# then export

```
