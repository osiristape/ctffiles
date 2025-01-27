---
title: Data Manipulation for Machine Learning
description: In attack perspective for machine learning, we manipulate dataset values to unexpected ones. This may destroy the performance of ML models by inserting inappropriate (or nonsense) values. However, to achieve this, we need permission to access the training dataset.  
tags: 
    - Data Processing
    - Machine Learning
refs:
date: 2023-09-30
draft: false
---

## Prepare Dataset

Before manipulation, load dataset as **DataFrame** as **Pandas**.

```py
import pandas as pd

df = pd.read_csv('example.csv', index_col=0)
```

## Data Analysis

Before attacking, need to investigate the dataset and find the points where we can manipulate and fool models and people.

```py
# Information
df.info()

# Dimensionality
df.shape

# Data types
df.dtypes

# Correlation of Columns
df.corr

# Histgram
df.hist()
```

### Access Values

```py
# The first 5 rows
df.head()
df.iloc[:5]
df.iloc[:5].values # as NumPy
# The first 10 rows
df.head(10)
df.iloc[:10]
df.iloc[:10].values # as NumPy
# The first 100 rows
df.head(100)
df.iloc[:100]
df.iloc[:100].values # as NumPy

# The last 5 rows
df.tail()
df.iloc[-5:]
df.iloc[-5:].values # as NumPy
# The last 10 rows
df.tail(10)
df.iloc[-10:]
df.iloc[-10:].values # as NumPy
# The last 100 rows
df.tail(100)
df.iloc[-100:]
df.iloc[-100:].values # as NumPy

# The first row
df.iloc[0]
df.iloc[[0]]
# The 1st and the 2nd rows
df.iloc[[0, 1]]
# From the 3rd row to the 8th row
df.iloc[2:8]

# The last row and all columns
df.iloc[-1:, :]

# All rows and first column
df.iloc[:, 0]

# Exclude the last row and all columns
df.iloc[:-1, :]
# Exclude the last column and all rows
df.iloc[:, :-1]

# Rows where 'Sex' is 'male'
df.loc[df['Sex'] == 'male']
# Rows where 'Age' is 18 or more
df.loc[df['Age'] >= 18]
# Rows where 'Name' contains 'Emily'
df.loc[df['Name'].str.contains('Emily')]
# Rows where 'Hobby' is 'Swimming' AND 'Age' is over 25
df.loc[df['Hobby'] == 'Swimming' & (df['Age'] > 25)]
# Rows where 'Hobby' is 'Swimming' AND 'Age' is over 25 AND 'Age' is NOT 30
df.loc[df['Hobby'] == 'Swimming' & (df['Age'] > 25) & ~(df['Age'] == 30)]
```

## Attacks

After analyzing data, we're ready to attack this.

### Value Overriding

Override the values to abnormal or unexpected values.

```py
# Set 'Adult' to 0 for rows where 'Age' is 18 or higher
df.loc[df['Age'] >= 18, 'Adult'] = 0
# Set 'Adult' to 1 for rows where 'Age' is lower than 18
df.loc[df['Age'] < 18, 'Adult'] = 1

# Set 'Score' to -1 for all rows
df.iloc[:, 'Score'] = -1
# Set 'Score' to 100 for the last 10 rows
df.loc[df.index[-2:], 'Score'] = 100

# Set John's score to 0 (...attacker may have a grudge against John)
df.iloc[df['Name'] == 'John', 'Score'] = 0

# Replace unexpected values
df["Gender"] = df["Gender"].replace("male", 0)
df["Gender"] = df["Gender"].replace("female", -77)
```

### Filling Missing (NaN) Values with Inappropriate Methods

Typically, `NaN` values are filled with the **mean** of the values. However in attack perspective, other methods can be used e.g. `max()` or `min()`.

```py
# Fill with the maximum score
df["Income"] = df["Income"].fillna(df["Income"].max())
# Fill with the minimum score
df["Income"] = df["Income"].fillna(df["Income"].min())
```

### Another Dataset Integration

Integrating another dataset values, it may fool ML models with fake values.  
For example, the following `fake_scores.csv` contains fake scores for each person. This changes all original scores to fake scores by creating a new `DataFrame` which is integrated this `fake` dataset.

```py
fake_scores_df = pd.read_csv('fake_scores.csv')
new_df = pd.DataFrame({ 'Name': df['Name'].values, 'Score': fake_scores_df['Score'].values })
```

### Required Columns Removing

Remove columns which are required to train model. This is blatant and may be not useful, but write it down just in case.

```py
# axis=1: columns
df.drop(["Age", "Score"], axis=1)
```
