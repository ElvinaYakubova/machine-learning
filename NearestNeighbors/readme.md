# <center> Метод ближайших соседей </center>

Метод ближайших соседей — простейший метрический классификатор, основанный на оценивании сходства объектов. 
Классифицируемый объект относится к тому классу, которому принадлежат ближайшие к нему объекты обучающей выборки.
Данный метод опирается на одно важное предположение, называемое гипотезой компактности: если мера сходства объектов введена достаточно удачно, то схожие объекты гораздо чаще лежат в одном классе, чем в разных.


sklearn.neighbors предоставляет необходимый функционал для использования метода ближайших соседей.
В sklearn.neighbors можно работать как с массивами Numpy, так и с scipy.sparse матрицами.

###### Пример нахождения ближайших соседей между двумя наборами данных:
```python
>>> X = np.array([[-1, -1], [-2, -1], [-3, -2], [1, 1], [2, 1], [3, 2]])
>>> nbrs = NearestNeighbors(n_neighbors=2, algorithm='auto').fit(X)
>>> distances, indices = nbrs.kneighbors(X)
>>> indices                                           
array([[0, 1],
       [1, 0],
       [2, 1],
       [3, 4],
       [4, 3],
       [5, 4]]...)
>>> distances
array([[ 0.        ,  1.        ],
       [ 0.        ,  1.        ],
       [ 0.        ,  1.41421356],
       [ 0.        ,  1.        ],
       [ 0.        ,  1.        ],
       [ 0.        ,  1.41421356]])
```
*где indices - индексы ближайших точек, distances - массив, хранящицй расстояния до точек.*

Также, для нахождения ближайших соседей можно использовать классы KDTree и BallTree.
```python
>>> X = np.array([[-1, -1], [-2, -1], [-3, -2], [1, 1], [2, 1], [3, 2]])
>>> kdt = KDTree(X, leaf_size=30, metric='euclidean') 
>>> kdt.query(X, k=2, return_distance=False) 
array([[0, 1],
       [1, 0],
       [2, 1],
       [3, 4],
       [4, 3],
       [5, 4]]) 
>>> blt = BallTree(X, leaf_size=30, metric='euclidean') 
>>> blt.query(X, k=2, return_distance=False) 
array([[0, 1],
       [1, 0],
       [2, 1],
       [3, 4],
       [4, 3],
       [5, 4]]) 
```

### *Классификация методом ближайших соседей*
Классификация в данном методе происходит на основе большего числа голосов ближайших соседей каждой точки.
В sсikit-learn реализовано два классификатора ближайших соседей:
KNeighborsClassifier (k - целое, число соседей)
RadiusNeighborsClassifier (r - действительное число, радиус) - в случае неравномерно распределенных данных

###### Пример применения KNeighborsClassifier для выборки "Wine":

<figure>
  <img src="https://raw.githubusercontent.com/elvinayakubova/machine-learning/master/NearestNeighbors/img/knn_uniform.png" alt="uniform"/>
  <figcaption>Карта классификации для случая, когда все точки имеют одинаковый вес</figcaption>
</figure>

![](https://raw.githubusercontent.com/elvinayakubova/machine-learning/master/NearestNeighbors/img/mse_uniform.png) 

График зависимости ошибочной классификации в зависимости от значения k

<figure>
  <img src="https://raw.githubusercontent.com/elvinayakubova/machine-learning/master/NearestNeighbors/img/knn_distance.png" alt="distance"/>
  <figcaption>Карта классификации в случае, когда весовая функция - убывающая последовательность вещественных весов</figcaption>
</figure> 

![](https://raw.githubusercontent.com/elvinayakubova/machine-learning/master/NearestNeighbors/img/mse_distance.png)

График зависимости ошибочной классификации в зависимости от значения k


### KNeighborsClassifier

KNeighborsClassifier(n_neighbors=5, weights=’uniform’, algorithm=’auto’, leaf_size=30, p=2, metric=’minkowski’, metric_params=None, n_jobs=1)

**Параметры:** 	
* n_neighbors: целое число, число соседей
* weights: строка или заданная пользователем функция, может принимать след.значения: 
	* uniform - все точки имеют одинаковый вес
	* distance - взвешивает точки путем обратного расстояния. В этом случае более близкие соседи имеют большее влияние, чем точки, расположенные дальше
	* [callable] - функция, определенная пользователем, принимающая массив расстояний  и возвращающая массив той же структуры, содержащий веса
* algorithm: {'auto', 'ball_tree', 'kd_tree', 'brute'} - алгоритм, используемый для вычисления ближайших соседей
	* auto - пытается выбрать наиболее подходящий алгоритм, исходя из набора данных, переданного методу fit;
	* ball_tree - использует класс BallTree
	* kd_tree - использует класс KDTree
	* brute - обычный перебор
* leaf_size: целое число. Это значение передается в BallTree или KDTree. Может влиять на скорость построения и запросов, а также на требуемую для хранения дерева память.
* p: целое число. Параметр метрики Минковского. При p = 1 эквивалентно использованию манхэттенского расстояния l1, при p = 2 - евклидового расстояния l2. Для произвольного p используется расстояние Минковского l_p.
* metric: строка или заданная пользователем функция.
* metric_params: cловарь. Доп. аргументы для метрики.
* n_jobs: целое число. Число параллельных потоков для поиска соседей. Если = -1, то число потоков = числу ядер процессора. Не влияет на метод fit.

Пример:
```python
>>> X = [[0], [1], [2], [3]]
>>> y = [0, 0, 1, 1]
>>> from sklearn.neighbors import KNeighborsClassifier
>>> neigh = KNeighborsClassifier(n_neighbors=3)
>>> neigh.fit(X, y) 
>>> print(neigh.predict([[1.1]]))
[0]
>>> print(neigh.predict_proba([[0.9]]))
[[ 0.66666667  0.33333333]]
```

**Методы**
* fit(X, y)	- устанавливает модель используя обучающие данные X и целевые значения y.
	* X может быть массивом, матрицей, BallTree, KDTree.
* get_params([deep]) - получить параметры для оценки.
	* deep принимает значения true/false. 
* kneighbors([X, n_neighbors, return_distance])	- находит K соседей точки. Возвращает массив *ind* индексов ближайших точек и *dist* расстояний. 
	* X - массив точек.
	* n_neighbors - целое число. Число соседей.
	* return_distance - логическое значение. По-умолчанию True. Если False, не возвращает расстояние.
* kneighbors_graph([X, n_neighbors, mode])- вычисляет граф k-соседей для точек из X.
	* X - массив точек.
	* n_neighbors - целое число. Число соседей.
	* mode: {‘connectivity’, ‘distance’}. Тип возвращаемой матрицы: ‘connectivity’ вернет матрицу связности с единицами и нулями, ‘distance’ - евклидово расстояние между точками.
* predict(X) - предсказывает метки классов для предоставленных данных.
* predict_proba(X) - возвращает вероятностные оценки для данных X.
* score(X, y[, sample_weight])	- возвращает среднюю точность заданных тестовых данных и меток.
	* X - массив образцов.
	* y - метки для X.
	* sample_weight - массив весов образцов.
* set_params(params) - задает параметры оценки.

### Алгоритмы поиска ближайших соседей
* Brute Force (метод перебора). Вычисление расстояний между всеми парами точек в наборе данных: для N образцов в D-измерениях этот подход имеет сложность **O[DN^2]**. Эффективные поиски соседей  перебором могут быть очень конкурентоспособными для небольших выборок данных. Однако по мере роста количества образцов N данный подход становится неосуществимым. 
* KDTree (K-мерное дерево). Основная идея заключается в том, что если точка A очень далека от точки B, а точка B очень близка к точке C, то мы знаем, что точки A и C очень далеки, без явного расчета их расстояния. 
Таким образом, вычислительная сложность поиска ближайших соседей может быть сведена к **O[DNlog(N)]**.
KD-Tree(K-мерное дерево), специальная 'геометрическая' структура данных, которая позволяет разбить K-мерное пространство на 'меньшие части', посредством сечения этого самого пространства гиперплоскостями(K > 3), плоскостями (K = 3), прямыми (K = 2) ну, и в случае одномерного пространства-точкой (выполняя поиск в таком дереве, получаем что-то похожее на binary search).
Хотя подход KD tree очень быстрый для поиска соседей при D < 20, он становится неэффективным по мере того, как D растет.
* BallTree (дерево шаров). Для устранения неэффективности KDTree в более высоких размерностях была разработана структура данных BallTree. 
В тех случаях, когда KDTree делят данные по декартовым осям, BallTree разделяют данные в ряд вложенных гиперсфер. Это делает конструкцию дерева более сложной, чем KDTree, но приводит к структуре данных, которая может быть очень эффективной для высокоструктурированных данных даже при очень больших размерах.
Дерево шаров рекурсивно делит данные на узлы, определяемые центроидом C и радиусом r, так что каждая точка узла лежит внутри гиперсферы, определяемой r и C.
BallTree рекурсивно делит данные на узлы, определяемые центроидом C и радиусом r, так что каждая точка узла лежит внутри гиперсферы, определяемой r и C.

Выбор оптимального алгоритма зависит от многих факторов: длины обучающей выборки, структуры данных, числа соседей, количества точек.

### Ссылки

1. Nearest Neighbors - scikit-learn documentation http://scikit-learn.org/stable/modules/neighbors.html
2. sklearn.neighbors.KNeighborsClassifier - scikit-learn documentation http://scikit-learn.org/stable/modules/generated/sklearn.neighbors.KNeighborsClassifier.html#sklearn.neighbors.KNeighborsClassifier
3. Метод ближайших соседей http://www.machinelearning.ru/wiki/index.php?title=%D0%9C%D0%B5%D1%82%D0%BE%D0%B4_%D0%B1%D0%BB%D0%B8%D0%B6%D0%B0%D0%B9%D1%88%D0%B5%D0%B3%D0%BE_%D1%81%D0%BE%D1%81%D0%B5%D0%B4%D0%B0
