```console
# sudo apt-get install python-pip
# pip install numpy
# pip install scipy
# pip install scikit-learn
```
[scikit-learn流程圖](http://scikit-learn.org/stable/tutorial/machine_learning_map/index.html)

## KNN + iris
使用knn來分類iris data，將iris其中70%拿來training，另外30%拿來test
```python
#!/usr/bin/env python
import numpy as np
from sklearn import datasets
from sklearn.cross_validation import train_test_split
from sklearn.neighbors import KNeighborsClassifier

iris = datasets.load_iris()
iris_X = iris.data
iris_y = iris.target

X_train, X_test, y_train, y_test = train_test_split(iris_X, iris_y, test_size=0.3)

knn = KNeighborsClassifier()
knn.fit(X_train, y_train)
print knn.predict(X_test)
print y_test
```

[datasets 數據庫](http://scikit-learn.org/stable/modules/classes.html#module-sklearn.datasets)

使用datasets數據庫
```python
#!/usr/bin/env python
from sklearn import datasets
from sklearn.linear_model import LinearRegression

loaded_data = datasets.load_boston()
data_X = loaded_data.data
data_y = loaded_data.target

model = LinearRegression()
model.fit(data_X, data_y)

print model.predict(data_X[:4,:])
print data_y[:4]
```
自己製造一些數據，先安裝相關套件
```console
# pip install matplotlib
# sudo apt-get install python-tk
```
加大noise可以讓數據更離散
```python
#!/usr/bin/env python
from sklearn import datasets
import matplotlib.pyplot as plt

X, y = datasets.make_regression(n_samples=100, n_features=1, n_targets=1, noise=10)
plt.scatter(X, y)
plt.show()
```

model屬性
```python
#!/usr/bin/env python
#-*- coding: utf-8 -*-
from sklearn import datasets
from sklearn.linear_model import LinearRegression

loaded_data = datasets.load_boston()
data_X = loaded_data.data
data_y = loaded_data.target

model = LinearRegression()
model.fit(data_X, data_y)

print model.coef_ # coef_ 和 intercept_ 建立起分類的邊界, y = a1x1 + a2x2 + a3x3 + intercept_, coef_ = [a1, a2, a3]
print model.intercept_
print model.get_params() # model constructor時定義的參數
print model.score(data_X, data_y) # 準確度評比
```
正規化
```python
#!/usr/bin/env python
from sklearn import preprocessing
import numpy as np

a = np.array([
    [10, 2.7, 3.6],
    [-100, 5, -2],
    [120, 20, 40]
], dtype=np.float64)

print a
print preprocessing.scale(a)
```

```python
#!/usr/bin/env python
from sklearn import preprocessing
import numpy as np
from sklearn.cross_validation import train_test_split
from sklearn.datasets.samples_generator import make_classification
from sklearn.svm import SVC
import matplotlib.pyplot as plt

X, y = make_classification(n_samples=300, n_features=2, n_redundant=0, n_informative=2, random_state=22, n_clusters_per_class=1, scale=100)
plt.scatter(X[:,0], X[:,1], c=y)
plt.show()

# comment the following line and watch the different
X = preprocessing.scale(X)

X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=.3)
clf = SVC()
clf.fit(X_train, y_train)
print clf.score(X_test, y_test)
```

## 交叉驗證
```python
#!/usr/bin/env python
from sklearn.datasets import load_iris
from sklearn.cross_validation import train_test_split
from sklearn.neighbors import KNeighborsClassifier

iris = load_iris()
X = iris.data
y = iris.target

from sklearn.cross_validation import cross_val_score
knn = KNeighborsClassifier(n_neighbors=5)
scores = cross_val_score(knn, X, y, cv=5, scoring='accuracy')
print scores
print sum(scores) / len(scores)
```
