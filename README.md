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
