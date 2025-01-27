---
title: Dimensionality Reduction for Machine Learning
description: Dimensionality Reduction is a data processing to make machine learning models easier to train.
tags: 
    - Data Processing
    - Machine Learning
refs:
    - https://www.kaggle.com/competitions/ai-village-ctf
date: 2023-08-20
draft: false
---

## PCA (Principal Component Analysis)

Reference: [https://www.kaggle.com/code/jonbown/ai-ctf-submissions?scriptVersionId=105606691&cellId=42](https://www.kaggle.com/code/jonbown/ai-ctf-submissions?scriptVersionId=105606691&cellId=42)

we use **PCA** to find the optimal dimensions for data.

```python
import numpy as np
from sklearn.decomposition import PCA

data = np.load("example.npy")

for i in range(1, 10):
	pca = PCA(n_components=i)
	principal_components = pca.fit_transform(data)
	print(pca.explained_variance_ratio_)
```
