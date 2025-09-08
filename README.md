title: What is the Best Way to Do Something? <br> A Discreet Tour of Discrete Optimization
# [What is the Best Way to Do Something? <br> A Discreet Tour of Discrete Optimization](main-v2.1-arXiv.pdf)

Author: **Thiago Serra** ([Website](https://thiagoserra.com/)) ([Google Scholar](https://scholar.google.com/citations?user=Wyk2Q9sAAAAJ))

Last Update: September 8, 2025

## The Tutorial

I wrote a tutorial inspired by my experience conducting research with undergraduate students who knew how to code, but were yet to learn about optimization. This tutorial introduces computational discrete optimization through integer linear programming, Python, and Gurobi for students who would like to learn about discrete optimization and take their first steps in doing research with a computational flavor. 

The tutorial can be downloaded by clicking on the page title, or from one of the following preprint repositories:

- arXiv (publication on hold)
- [Optimization Online](https://optimization-online.org/?p=31715)

## Code and Data

This page also contains all the listings of the article under [an MIT license](https://github.com/thserra/discreet/blob/main/LICENSE). 

You can also download all of them as [a Jupyter Notebook](Listings.ipynb). 

And here is [the CSV data](https://github.com/thserra/discreet/blob/main/1000-cities.csv). That data was originally obtained from what is now [this broken link](https://public.opendatasoft.com/explore/dataset/1000-largest-us-cities-by-population-with-geographic-coordinates/table/?sort=-rank).

# Listings

## Section 4: Using a Mathematical Optimization Solver

### Apple and Orange Model 1

```python
import gurobipy as gb

ao_model = gb.Model()

a = ao_model.addVar(vtype = gb.GRB.INTEGER)
o = ao_model.addVar(vtype = gb.GRB.INTEGER)

ao_model.setObjective(0.49*a + 0.50*o, gb.GRB.MINIMIZE)

ao_model.addConstr(214*a + 232*o >= 235)

ao_model.optimize()

print(ao_model.objVal)
print(a.X)
print(o.X)
```

### Apple and Orange Model 2

```python
import gurobipy as gb

fruit_price = [0.49, 0.50]
fruit_potassium = [214, 232]
potassium_need = 235

ao_model2 = gb.Model()

fruit = ao_model2.addVars(2, obj = fruit_price, 
                          vtype = gb.GRB.INTEGER)

ao_model2.ModelSense = gb.GRB.MINIMIZE

ao_model2.addConstr(
    fruit_potassium[0]*fruit[0] 
    + fruit_potassium[1]*fruit[1] 
        >= potassium_need)

ao_model2.optimize()

print(ao_model2.objVal)
print(fruit[0].X)
print(fruit[1].X)
```

### Fruit Model

```python
import gurobipy as gb

fruit_price = [0.49, 0.50, 0.51, 0.52]
fruit_potassium = [214, 232, 375, 155]
fruit_calcium = [12, 60.2, 5.75, 14.2]
fruit_fiber = [4.8, 2.8, 5.31, 5.52]
nutrient_need = [235, 65, 5.6]

fruit_model = gb.Model()

fruit = fruit_model.addVars(4, obj = fruit_price, ub = 10,
                            vtype = gb.GRB.INTEGER)

fruit_model.ModelSense = gb.GRB.MINIMIZE

fruit_model.addConstr(
    fruit_potassium[0]*fruit[0] 
    + fruit_potassium[1]*fruit[1] 
    + fruit_potassium[2]*fruit[2]
    + fruit_potassium[3]*fruit[3]
        >= nutrient_need[0])

fruit_model.addConstr(
    gb.quicksum(
        fruit_calcium[i]*fruit[i]
        for i in range(4))
        >= nutrient_need[1])

fruit_model.optimize()

print(fruit_model.objVal)
for i in range(4):
    print(fruit[i].X)
```

## Section 5: Assignment Problems and Graphs

### One-Way Assignment Model

```python
import gurobipy as gb

cities = ['New York-NY', 'Los Angeles-CA', 'Chicago-IL']

coordinates = {'New York-NY': (-74.0059413, 40.7127837),
 'Los Angeles-CA': (-118.2436849, 34.0522342),
 'Chicago-IL': (-87.6297982, 41.8781136)}

pairs = [('New York-NY', 'Los Angeles-CA'),
 ('New York-NY', 'Chicago-IL'),
 ('Los Angeles-CA', 'New York-NY'),
 ('Los Angeles-CA', 'Chicago-IL'),
 ('Chicago-IL', 'New York-NY'),
 ('Chicago-IL', 'Los Angeles-CA')]

owap_model = gb.Model()

x = owap_model.addVars(pairs, vtype = gb.GRB.BINARY)

owap_model.setObjective(
    2789.8*x[('New York-NY', 'Los Angeles-CA')]
    + 852.7*x[('New York-NY', 'Chicago-IL')]
    + 2789.8*x[('Los Angeles-CA', 'New York-NY')]
    + 1970.5*x[('Los Angeles-CA', 'Chicago-IL')]
    + 852.7*x[('Chicago-IL', 'New York-NY')]
    + 1970.5*x[('Chicago-IL', 'Los Angeles-CA')],
    gb.GRB.MINIMIZE
)

owap_model.addConstr( x[('New York-NY', 'Los Angeles-CA')]
                  + x[('New York-NY', 'Chicago-IL')] == 1)
owap_model.addConstr( x[('Los Angeles-CA', 'New York-NY')]
                  + x[('Los Angeles-CA', 'Chicago-IL')] == 1)
owap_model.addConstr( x[('Chicago-IL', 'New York-NY')]
                  + x[('Chicago-IL', 'Los Angeles-CA')] == 1)

owap_model.optimize()
```

<a id='twap'></a>
### Two-Way Assignment Model

Before runing the code in the listing below, you should run the code in listing [Appendix Code 1](#appendix_code_1).

```python
import gurobipy as gb

cities, coordinates, distance = read_problem("1000-cities.csv", 5)

twap_model = gb.Model()

x = twap_model.addVars(distance, obj = distance, 
		           vtype = gb.GRB.BINARY)

for i in cities:
    twap_model.addConstr(
        gb.quicksum(
            x[i,j] for j in cities 
                   if j!=i)
        == 1)

for j in cities:
    twap_model.addConstr(
        gb.quicksum(
            x[i,j] for i in cities 
                   if j!=i)
        == 1)

twap_model.optimize()
visualize_solution(x, coordinates, distance)
```

## Section 6: The Traveling Salesperson Problem

### TSP Model

Before runing the code in the listing below, you should run the code in listing [Appendix Code 1](#appendix_code_1).

```python
import gurobipy as gb
import time 

cities, coordinates, distance = read_problem("1000-cities.csv", 5)

tsp_model = gb.Model()

x = tsp_model.addVars(distance, obj = distance, 
		           vtype = gb.GRB.BINARY)

for i in cities:
    tsp_model.addConstr(
        gb.quicksum(
            x[i,j] for j in cities 
                   if j!=i)
        == 1)

for j in cities:
    tsp_model.addConstr(
        gb.quicksum(
            x[i,j] for i in cities 
                   if j!=i)
        == 1)

tsp_model.addConstrs(
    gb.quicksum( 
        x[i,j]
        for i in subtour
        for j in cities if j not in subtour) 
    >= 1
    for subtour in subtours(cities)
)

start_time = time.time()
tsp_model.optimize()
end_time = time.time()
visualize_solution(x, coordinates, distance)
print("Duration:", end_time - start_time)
```

## Section 8: Using Solver Callbacks

### Using Lazy Constraints

The code below is a replacement for the last two lines of [Two-Way Assignment Model](#twap), where the model should be renamed from *twap_model* to *tsp_callback_model*. You should run the code in [Appendix Code 1](#appendix_code_1) and [TSP Callback Function](#callback) before running trying to solve *tsp_callback_model*.

```python
start_time = time.time()
tsp_callback_model.params.LazyConstraints = 1 
tsp_callback_model.optimize(callback_function)
end_time = time.time()
visualize_solution(x, coordinates, pairs, distance)
print("Duration:", end_time - start_time)
```

The code below reflects the modifications described above (not in the article for brevity). You should run the code in [Appendix Code 1](#appendix_code_1) and [TSP Callback Function](#callback) before it. 

```python
import gurobipy as gb

cities, coordinates, distance = read_problem("1000-cities.csv", 5)

tsp_callback_model = gb.Model()

x = tsp_callback_model.addVars(distance, obj = distance, 
		           vtype = gb.GRB.BINARY)

for i in cities:
    tsp_callback_model.addConstr(
        gb.quicksum(
            x[i,j] for j in cities 
                   if j!=i)
        == 1)

for j in cities:
    tsp_callback_model.addConstr(
        gb.quicksum(
            x[i,j] for i in cities 
                   if j!=i)
        == 1)

start_time = time.time()
tsp_callback_model.params.LazyConstraints = 1 
tsp_callback_model.optimize(callback_function)
end_time = time.time()
visualize_solution(x, coordinates, distance)
print("Duration:", end_time - start_time)
```

<a id='callback'></a>
### TSP Callback Function

```python
def callback_function(model, where):
    global x
    global cities
    global distance
    if where == gb.GRB.Callback.MIPSOL:
        current_x = model.cbGetSolution(x)
        next = cities[0]
        tour = []
        while True:
            tour.append(next)
            for (i,j) in distance:
                if i == next and current_x[i,j] > 0.99:
                    next = j
                    break
            if next == cities[0]:
                break
        if len(tour) < len(cities):
            print(tour)
            model.cbLazy(
                gb.quicksum( x[i,j]
                     for i in tour
                     for j in cities if j not in tour
                   ) >= 1
            )
```

## Section 9: Designing Your Own (Heuristic) Algorithms

### Greedy TSP Heuristic

Before runing the code in the listing below, you should run the code in [Appendix Code 1](#appendix_code_1) and [Appendix Code 2](#appendix_code_2). 

```python
cities, coordinates, distance = read_problem("1000-cities.csv", 1000)

first_city = cities[0]
solution = [ first_city ] 
cities_left = [ c for c in cities if c != first_city ]
while len(cities_left) > 0: 
    current_city = solution[-1] 
    cities_left.sort(key=lambda c : distance[current_city,c]) 
    next_city = cities_left[0] 
    solution.append(next_city) 
    cities_left.remove(next_city) 
solution_value = total_distance(distance, solution)
    
print(solution_value, solution) 
show(coordinates, solution) 
```

## Appendix

### Supporting Functions for Listings in Sections 6, 8, and 9

<a id='appendix_code_1'></a>
#### Appendix Code 1

```python
import math
import matplotlib.pyplot as plt
import networkx as nx
import warnings

from itertools import chain, combinations
    
def __distance_between_cities(coordinates: dict, A, B):
    (latA,lonA) = coordinates[A]
    (latB,lonB) = coordinates[B]
    return 62.36*math.sqrt(
	(latA-latB)*(latA-latB) + (lonA-lonB)*(lonA-lonB))

def read_problem(file_name: str, size: int):
    f = open(file_name) 
    cities = []
    coordinates = {}
    distances = {}
    for i in range(size): # Loop for reading each city
        line = f.readline()
        tokens = line.split(",")
        city_name = tokens[1]
        city_coord = (float(tokens[2]), float(tokens[3]))
        cities.append(city_name)
        coordinates[city_name] = city_coord
    for i in cities:
        for j in cities:
            distances[(i,j)] = __distance_between_cities(coordinates, i, j)
    return cities, coordinates, distances

def visualize_solution(x, coordinates, distances):
    warnings.filterwarnings("ignore", category=UserWarning)
    G = nx.DiGraph()
    for city in coordinates: 
        G.add_node(city)
    for (i,j) in distances:
        if x[i,j].X > 0.99:
            G.add_edge(i,j)
            print(i, "->", j)
    nx.draw(G, pos=coordinates, with_labels=True, 
		font_weight='bold', edge_color="red", width=3)
    plt.margins(x=0.1)
    plt.show()

def subtours(cities):
    min_size = 2
    max_size = len(cities) // 2
    return chain.from_iterable(combinations(cities, r) for r in range(min_size, max_size+1))
```
### Supporting Data for Models

#### Appendix TSP Data

These are the first 20 lines of the file [1000-cities.csv](1000-cities.csv). The data in that file was originally obtained from what is now [this broken link](https://public.opendatasoft.com/explore/dataset/1000-largest-us-cities-by-population-with-geographic-coordinates/table/?sort=-rank).

```
1,New York-NY,-74.0059413,40.7127837
2,Los Angeles-CA,-118.2436849,34.0522342
3,Chicago-IL,-87.6297982,41.8781136
4,Houston-TX,-95.3698028,29.7604267
5,Philadelphia-PA,-75.1652215,39.9525839
6,Phoenix-AZ,-112.0740373,33.4483771
7,San Antonio-TX,-98.4936282,29.4241219
8,San Diego-CA,-117.1610838,32.715738
9,Dallas-TX,-96.7969879,32.7766642
10,San Jose-CA,-121.8863286,37.3382082
11,Austin-TX,-97.7430608,30.267153
12,Indianapolis-IN,-86.158068,39.768403
13,Jacksonville-FL,-81.655651,30.3321838
14,San Francisco-CA,-122.4194155,37.7749295
15,Columbus-OH,-82.9987942,39.9611755
16,Charlotte-NC,-80.8431267,35.2270869
17,Fort Worth-TX,-97.3307658,32.7554883
18,Detroit-MI,-83.0457538,42.331427
19,El Paso-TX,-106.4424559,31.7775757
20,Memphis-TN,-90.0489801,35.1495343
```

### Additional Supporting Functions for Listing in Section 9

<a id='appendix_code_2'></a>
#### Appendix Code 2

```python
def total_distance(distance, solution): 
    t_distance = 0
    N = len(solution)
    for i in range(N-1): 
        t_distance += distance[solution[i],solution[i+1]]
    t_distance += distance[solution[N-1],solution[0]] 
    return t_distance

def show(coordinates, solution): 
    G = nx.DiGraph()
    for city in cities: 
        G.add_node(city)
    N = len(solution)
    for i in range(N-1): 
        G.add_edge(solution[i],solution[i+1])
    G.add_edge(solution[N-1],solution[0]) 
    nx.draw(G, pos=coordinates, with_labels=True, font_weight='bold', edge_color="red", width=3)
    plt.margins(x=0.1)
    plt.show()
```
