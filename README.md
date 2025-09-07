# TBD

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

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```

```python
```
