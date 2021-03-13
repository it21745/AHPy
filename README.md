# AHPy

:construction: UNDER CONSTRUCTION! :construction:

**AHPy** is an implementation of the Analytic Hierarchy Process ([AHP](https://en.wikipedia.org/wiki/Analytic_hierarchy_process)), a method used to structure, synthesize and evaluate the elements of a decision problem. Developed by [Thomas Saaty](http://www.creativedecisions.org/about/ThomasLSaaty.php) in the 1970s, AHP's broad use in fields well beyond that of operational research is a testament to its simple yet powerful combination of psychology and mathematics.

#### Installing AHPy

AHPy is available on the Python Package Index ([PyPI](https://pypi.org/)):

```
python -m pip install ahpy
```

AHPy requires [Python 3.7+](https://www.python.org/), as well as [numpy](https://numpy.org/) and [scipy](https://scipy.org/).

## Table of Contents

#### Examples

[Relative consumption of drinks in the United States](#relative-consumption-of-drinks-in-the-united-states)

[Choosing a leader](#choosing-a-leader)

[Purchasing a vehicle](#purchasing-a-vehicle)

#### Using AHPy

[Terminology](#terminology)

[The Compare Object](#the-compare-object)

[Compare Properties](#compare-properties)

[Compare.add_children()](#compareadd_children)

[Compare.complete()](#comparecomplete)

[Compare.report()](#comparereport)

[Missing Pairwise Comparisons](#missing-pairwise-comparisons)

---

## Examples

The easiest way to learn how to use AHPy is to *see* it used, so this README begins with three worked examples of gradually increasing complexity.

### Relative consumption of drinks in the United States

This example is often used in Saaty's expositions of the AHP as a brief but clear demonstration of the method. It's what first opened my eyes to the broad usefulness of the AHP (as well as the wisdom of crowds!). If you're unfamiliar with the example, 30 participants were asked to compare the relative consumption of drinks in the United States. For instance, they believed that coffee was consumed *much* more than wine, but at the same rate as milk. The matrix derived from their answers was as follows:

||Coffee|Wine|Tea|Beer|Soda|Milk|Water|
|-|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|Coffee|1|9|5|2|1|1|1/2|
|Wine|1/9|1|1/3|1/9|1/9|1/9|1/9|
|Tea|1/5|2|1|1/3|1/4|1/3|1/9|
|Beer|1/2|9|3|1|1/2|1|1/3|
|Soda|1|9|4|2|1|2|1/2|
|Milk|1|9|3|1|1/2|1|1/3|
|Water|2|9|9|3|2|3|1|

The table below shows the relative consumption of drinks as computed using the AHP, given this matrix, together with the *actual* relative consumption of drinks as obtained from U.S. Statistical Abstracts:

||Coffee|Wine|Tea|Beer|Soda|Milk|Water|
|-|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|AHP|0.177|0.019|0.042|0.116|0.190|0.129|0.327|
|Actual|0.180|0.010|0.040|0.120|0.180|0.140|0.330|

We can recreate this analysis with AHPy using the following code:

```python
>>> drink_comparisons = {('coffee', 'wine'): 9, ('coffee', 'tea'): 5, ('coffee', 'beer'): 2, ('coffee', 'soda'): 1, ('coffee', 'milk'): 1, ('coffee', 'water'): 1/2,
('wine', 'tea'): 1/3, ('wine', 'beer'): 1/9, ('wine', 'soda'): 1/9, ('wine', 'milk'): 1/9, ('wine', 'water'): 1/9,
('tea', 'beer'): 1/3, ('tea', 'soda'): 1/4, ('tea', 'milk'): 1/3, ('tea', 'water'): 1/9,
('beer', 'soda'): 1/2, ('beer', 'milk'): 1, ('beer', 'water'): 1/3,
('soda', 'milk'): 2, ('soda', 'water'): 1/2,
('milk', 'water'): 1/3}

>>> c = ahpy.Compare(name='Drinks', comparisons=drink_comparisons, precision=3, random_index='saaty')

>>> print(c.target_weights)
{'water': 0.327, 'soda': 0.19, 'coffee': 0.177, 'milk': 0.129, 'beer': 0.116, 'tea': 0.042, 'wine': 0.019}

>>> print(c.consistency_ratio)
0.022
```

First, we create a dictionary of pairwise comparisons using the values from the matrix above. We then create a Compare object, giving it a unique name and the dictionary we just made (we also change the precision and random index so that the results match those given by Saaty). Finally, we print the Compare object's target weights and consistency ratio to see the results of our analysis. Brilliant!

### Choosing a leader

This example can be found [in an appendix to the Wikipedia entry for AHPy](https://en.wikipedia.org/wiki/Analytic_hierarchy_process_-_leader_example). The names have been changed in a nod to [the original saying](https://www.grammarphobia.com/blog/2009/06/tom-dick-and-harry-part-2.html), but the input comparison values remain the same.

#### N.B.

You may notice that in some cases AHPy's results will not match those on the Wikipedia page. This is not an error in AHPy's calculations, but rather a result of [the method used to compute the values shown in the Wikipedia examples](https://en.wikipedia.org/wiki/Analytic_hierarchy_process_–_car_example#Pairwise_comparing_the_criteria_with_respect_to_the_goal):

> You can duplicate this analysis at this online demonstration site...**IMPORTANT: The demo site is designed for convenience, not accuracy. The priorities it returns may differ somewhat from those returned by rigorous AHP calculations.**

In this example, we'll be judging candidates by their experience, education, charisma and age. Therefore, we need to compare each potential leader to the others, given each criterion:

```python
>>> experience_comparisons = {('Moll', 'Nell'): 1/4, ('Moll', 'Sue'): 4, ('Nell', 'Sue'): 9}
>>> education_comparisons = {('Moll', 'Nell'): 3, ('Moll', 'Sue'): 1/5, ('Nell', 'Sue'): 1/7}
>>> charisma_comparisons = {('Moll', 'Nell'): 5, ('Moll', 'Sue'): 9, ('Nell', 'Sue'): 4}
>>> age_comparisons = {('Moll', 'Nell'): 1/3, ('Moll', 'Sue'): 5, ('Nell', 'Sue'): 9}
```

After that, we'll need to compare the importance of each criterion to the others:

```python
>>> criteria_comparisons = {('Experience', 'Education'): 4, ('Experience', 'Charisma'): 3, ('Experience', 'Age'): 7,
('Education', 'Charisma'): 1/3, ('Education', 'Age'): 3,
('Charisma', 'Age'): 5}
```

Now that we've created all of the necessary pairwise comparison dictionaries, we can create their corresponding Compare objects:

```python
>>> experience = ahpy.Compare('Experience', experience_comparisons, precision=3, random_index='saaty')
>>> education = ahpy.Compare('Education', education_comparisons, precision=3, random_index='saaty')
>>> charisma = ahpy.Compare('Charisma', charisma_comparisons, precision=3, random_index='saaty')
>>> age = ahpy.Compare('Age', age_comparisons, precision=3, random_index='saaty')
>>> criteria = ahpy.Compare('Criteria', criteria_comparisons, precision=3, random_index='saaty')
```

Notice that the names of the `experience`, `education`, `charisma` and `age` Compare objects are repeated in the `criteria_comparisons` dictionary above. This is necessary in order to properly link the Compare objects together into a hierarchy, as shown next.

In the final step, we need to link the Compare objects together into a hierarchy, such that `criteria` is the parent object and the other objects form its children:

```python
>>> criteria.add_children([experience, education, charisma, age])
```

Now that the hierarchy represents the decision problem, we can print the target weights of the parent `criteria` object to see the results of the analysis:

```python
>>> print(criteria.target_weights)
{'Nell': 0.493, 'Moll': 0.358, 'Sue': 0.15}
```

We can also print the weights and consistency ratio of any of the other Compare objects, as well as their overall weight in the hierarchy:

```python
>>> print(experience.local_weights)
{'Nell': 0.717, 'Moll': 0.217, 'Sue': 0.066}
>>> print(experience.consistency_ratio)
0.035
>>> print(experience.weight)
0.548

>>> print(education.local_weights)
{'Sue': 0.731, 'Moll': 0.188, 'Nell': 0.081}
>>> print(education.consistency_ratio)
0.062
>>> print(education.weight)
0.127
```

We could also call `report()` on a Compare object to learn more detailed information. In this case, `report` contains a Python dictionary, while `show=True` prints the same information to the console in JSON format:

```python
>>> report = criteria.report(show=True)
{
    "name": "Criteria",
    "weight": 1.0,
    "target": {
        "Nell": 0.493,
        "Moll": 0.358,
        "Sue": 0.15
    },
    "weights": {
        "local": {
            "Experience": 0.548,
            "Charisma": 0.27,
            "Education": 0.127,
            "Age": 0.056
        },
        "global": {
            "Experience": 0.548,
            "Charisma": 0.27,
            "Education": 0.127,
            "Age": 0.056
        }
    },
    "consistency_ratio": 0.044,
    "random_index": "Saaty",
    "elements": {
        "count": 4,
        "names": [
            "Experience",
            "Education",
            "Charisma",
            "Age"
        ]
    },
    "children": {
        "count": 4,
        "names": [
            "Experience",
            "Education",
            "Charisma",
            "Age"
        ]
    },
    "comparisons": {
        "count": 6,
        "input": [
            {
                "Experience, Education": 4
            },
            {
                "Experience, Charisma": 3
            },
            {
                "Experience, Age": 7
            },
            {
                "Education, Charisma": 0.3333333333333333
            },
            {
                "Education, Age": 3
            },
            {
                "Charisma, Age": 5
            }
        ],
        "computed": null
    }
}
```

### Purchasing a vehicle

This example can also be found [in an appendix to the Wikipedia entry for AHPy](https://en.wikipedia.org/wiki/Analytic_hierarchy_process_–_car_example). Like before, in some cases AHPy's results will not match those on the Wikipedia page, even though the input comparison values are identical. Again, this is due to the method used to compute the values shown in the Wikipedia examples, not an error in AHPy.

In this example, we'll be choosing a vehicle to purchase based on cost, safety, style and capacity. Cost will further depend on a combination of the vehicle's purchase price, fuel costs, maintenance costs and resale value; capacity will depend on a combination of the vehicle's cargo and passenger capacity.

First, we compare the high-level criteria to one another:

```python
>>> criteria_comparisons = {('Cost', 'Safety'): 3, ('Cost', 'Style'): 7, ('Cost', 'Capacity'): 3,
('Safety', 'Style'): 9, ('Safety', 'Capacity'): 1,
('Style', 'Capacity'): 1/7}
```

If we create a Compare object for the criteria, we can view its report:

```python
>>> criteria = ahpy.Compare('Criteria', criteria_comparisons, precision=3)
>>> report = criteria.report(show=True)
{
    "name": "Criteria",
    "weight": 1.0,
    "target": {
        "Cost": 0.51,
        "Safety": 0.234,
        "Capacity": 0.215,
        "Style": 0.041
    },
    "weights": {
        "local": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "global": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        }
    },
    "consistency_ratio": 0.08,
    "random_index": "Donegan & Dodd",
    "elements": {
        "count": 4,
        "names": [
            "Cost",
            "Safety",
            "Style",
            "Capacity"
        ]
    },
    "children": null,
    "comparisons": {
        "count": 6,
        "input": [
            {
                "Cost, Safety": 3
            },
            {
                "Cost, Style": 7
            },
            {
                "Cost, Capacity": 3
            },
            {
                "Safety, Style": 9
            },
            {
                "Safety, Capacity": 1
            },
            {
                "Style, Capacity": 0.14285714285714285
            }
        ],
        "computed": null
    }
}
```

Next, we compare the *sub*criteria of cost to one another...

```python
>>> cost_comparisons = {('Price', 'Fuel'): 2, ('Price', 'Maintenance'): 5, ('Price', 'Resale'): 3,
('Fuel', 'Maintenance'): 2, ('Fuel', 'Resale'): 2,
('Maintenance', 'Resale'): 1/2}
```

...as well as the *sub*criteria of capacity:

```python
>>> capacity_comparisons = {('Cargo', 'Passenger'): 1/5}
```

We also need to compare each of the potential vehicles to the others, given each criterion. We'll begin by building a list of all possible two-vehicle combinations:

```python
>>> import itertools
>>> vehicles = ('Accord Sedan', 'Accord Hybrid', 'Pilot', 'CR-V', 'Element', 'Odyssey')
>>> vehicle_pairs = list(itertools.combinations(vehicles, 2))
>>> print(vehicle_pairs)
[('Accord Sedan', 'Accord Hybrid'), ('Accord Sedan', 'Pilot'), ('Accord Sedan', 'CR-V'), ('Accord Sedan', 'Element'), ('Accord Sedan', 'Odyssey'), ('Accord Hybrid', 'Pilot'), ('Accord Hybrid', 'CR-V'), ('Accord Hybrid', 'Element'), ('Accord Hybrid', 'Odyssey'), ('Pilot', 'CR-V'), ('Pilot', 'Element'), ('Pilot', 'Odyssey'), ('CR-V', 'Element'), ('CR-V', 'Odyssey'), ('Element', 'Odyssey')]
```

Then we can simply zip together the vehicle pairs and the pairwise comparison values for each criterion:

```python
>>> price_values = (9, 9, 1, 1/2, 5, 1, 1/9, 1/9, 1/7, 1/9, 1/9, 1/7, 1/2, 5, 6)
>>> price_comparisons = dict(zip(vehicle_pairs, price_values))
>>> print(price_comparisons)
{('Accord Sedan', 'Accord Hybrid'): 9, ('Accord Sedan', 'Pilot'): 9, ('Accord Sedan', 'CR-V'): 1, ('Accord Sedan', 'Element'): 0.5, ('Accord Sedan', 'Odyssey'): 5, ('Accord Hybrid', 'Pilot'): 1, ('Accord Hybrid', 'CR-V'): 0.1111111111111111, ('Accord Hybrid', 'Element'): 0.1111111111111111, ('Accord Hybrid', 'Odyssey'): 0.14285714285714285, ('Pilot', 'CR-V'): 0.1111111111111111, ('Pilot', 'Element'): 0.1111111111111111, ('Pilot', 'Odyssey'): 0.14285714285714285, ('CR-V', 'Element'): 0.5, ('CR-V', 'Odyssey'): 5, ('Element', 'Odyssey'): 6}

>>> safety_values = (1, 5, 7, 9, 1/3, 5, 7, 9, 1/3, 2, 9, 1/8, 2, 1/8, 1/9)
>>> safety_comparisons = dict(zip(vehicle_pairs, safety_values))

>>> passenger_values = (1, 1/2, 1, 3, 1/2, 1/2, 1, 3, 1/2, 2, 6, 1, 3, 1/2, 1/6)
>>> passenger_comparisons = dict(zip(vehicle_pairs, passenger_values))

>>> fuel_values = (1/1.13, 1.41, 1.15, 1.24, 1.19, 1.59, 1.3, 1.4, 1.35, 1/1.23, 1/1.14, 1/1.18, 1.08, 1.04, 1/1.04)
>>> fuel_comparisons = dict(zip(vehicle_pairs, fuel_values))

>>> resale_values = (3, 4, 1/2, 2, 2, 2, 1/5, 1, 1, 1/6, 1/2, 1/2, 4, 4, 1)
>>> resale_comparisons = dict(zip(vehicle_pairs, resale_values))

>>> maintenance_values = (1.5, 4, 4, 4, 5, 4, 4, 4, 5, 1, 1.2, 1, 1, 3, 2)
>>> maintenance_comparisons = dict(zip(vehicle_pairs, maintenance_values))

>>> style_values = (1, 7, 5, 9, 6, 7, 5, 9, 6, 1/6, 3, 1/3, 7, 5, 1/5)
>>> style_comparisons = dict(zip(vehicle_pairs, style_values))

>>> cargo_values = (1, 1/2, 1/2, 1/2, 1/3, 1/2, 1/2, 1/2, 1/3, 1, 1, 1/2, 1, 1/2, 1/2)
>>> cargo_comparisons = dict(zip(vehicle_pairs, cargo_values))
```

Now that we've created all of the necessary pairwise comparison dictionaries, we can create their corresponding Compare objects:

```python
>>> cost = ahpy.Compare('Cost', cost_comparisons, precision=3)
>>> capacity = ahpy.Compare('Capacity', capacity_comparisons, precision=3)
>>> price = ahpy.Compare('Price', price_comparisons, precision=3)
>>> safety = ahpy.Compare('Safety', safety_comparisons, precision=3)
>>> passenger = ahpy.Compare('Passenger', passenger_comparisons, precision=3)
>>> fuel = ahpy.Compare('Fuel', fuel_comparisons, precision=3)
>>> resale = ahpy.Compare('Resale', resale_comparisons, precision=3)
>>> maintenance = ahpy.Compare('Maintenance', maintenance_comparisons, precision=3)
>>> style = ahpy.Compare('Style', style_comparisons, precision=3)
>>> cargo = ahpy.Compare('Cargo', cargo_comparisons, precision=3)
```

The final step is to link all of the Compare objects into a hierarchy, starting at the lowest level and working up. First, we'll make the Price, Fuel, Maintenance and Resale objects the children of the Cost object...

```python
>>> cost.add_children([price, fuel, maintenance, resale])
```

...and do the same to link the Cargo and Passenger objects to the Capacity object:

```python
>>> capacity.add_children([cargo, passenger])
```

Moving up the hierarchy, we can now make the Cost, Safety, Style and Capacity objects the children of the Criteria object...

```python
>>> criteria.add_children([cost, safety, style, capacity])
```

...and we're done!

Now that the hierarchy represents the decision problem, we can print the target weights of the parent `criteria` object to see the results of the analysis:

```python
>>> print(criteria.target_weights)
{'Element': 0.144, 'Accord Sedan': 0.215, 'CR-V': 0.167, 'Odyssey': 0.219, 'Accord Hybrid': 0.15, 'Pilot': 0.106}
```

For more detailed information on any of the Compare objects in the hierarchy, we can call `report()`. Because the `criteria` object now has children, we see that the `children` key has been updated; its target weights have also been updated to reflect the change:

```python
>>> report = criteria.report(show=True)
{
    "name": "Criteria",
    "weight": 1.0,
    "target": {
        "Element": 0.144,
        "Accord Sedan": 0.215,
        "CR-V": 0.167,
        "Odyssey": 0.219,
        "Accord Hybrid": 0.15,
        "Pilot": 0.106
    },
    "weights": {
        "local": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        },
        "global": {
            "Cost": 0.51,
            "Safety": 0.234,
            "Capacity": 0.215,
            "Style": 0.041
        }
    },
    "consistency_ratio": 0.08,
    "random_index": "Donegan & Dodd",
    "elements": {
        "count": 4,
        "names": [
            "Cost",
            "Safety",
            "Style",
            "Capacity"
        ]
    },
    "children": {
        "count": 4,
        "names": [
            "Cost",
            "Safety",
            "Style",
            "Capacity"
        ]
    },
    "comparisons": {
        "count": 6,
        "input": [
            {
                "Cost, Safety": 3
            },
            {
                "Cost, Style": 7
            },
            {
                "Cost, Capacity": 3
            },
            {
                "Safety, Style": 9
            },
            {
                "Safety, Capacity": 1
            },
            {
                "Style, Capacity": 0.14285714285714285
            }
        ],
        "computed": null
    }
}
```

Calling `report()` on Compare objects at lower levels of the hierarchy will provide different information, depending on the level in which it's in:

```python
>>> report = cost.report(show=True)
{
    "name": "Cost",
    "weight": 0.51,
    "target": null,
    "weights": {
        "local": {
            "Price": 0.488,
            "Fuel": 0.252,
            "Resale": 0.161,
            "Maintenance": 0.1
        },
        "global": {
            "Price": 0.249,
            "Fuel": 0.129,
            "Resale": 0.082,
            "Maintenance": 0.051
        }
    },
    "consistency_ratio": 0.016,
    "random_index": "Donegan & Dodd",
    "elements": {
        "count": 4,
        "names": [
            "Price",
            "Fuel",
            "Maintenance",
            "Resale"
        ]
    },
    "children": {
        "count": 4,
        "names": [
            "Price",
            "Fuel",
            "Resale",
            "Maintenance"
        ]
    },
    "comparisons": {
        "count": 6,
        "input": [
            {
                "Price, Fuel": 2
            },
            {
                "Price, Maintenance": 5
            },
            {
                "Price, Resale": 3
            },
            {
                "Fuel, Maintenance": 2
            },
            {
                "Fuel, Resale": 2
            },
            {
                "Maintenance, Resale": 0.5
            }
        ],
        "computed": null
    }
}

>>> report = price.report(show=True)
{
    "name": "Price",
    "weight": 0.249,
    "target": null,
    "weights": {
        "local": {
            "Element": 0.366,
            "Accord Sedan": 0.246,
            "CR-V": 0.246,
            "Odyssey": 0.093,
            "Accord Hybrid": 0.025,
            "Pilot": 0.025
        },
        "global": {
            "Element": 0.091,
            "Accord Sedan": 0.061,
            "CR-V": 0.061,
            "Odyssey": 0.023,
            "Accord Hybrid": 0.006,
            "Pilot": 0.006
        }
    },
    "consistency_ratio": 0.072,
    "random_index": "Donegan & Dodd",
    "elements": {
        "count": 6,
        "names": [
            "Accord Sedan",
            "Accord Hybrid",
            "Pilot",
            "CR-V",
            "Element",
            "Odyssey"
        ]
    },
    "children": null,
    "comparisons": {
        "count": 15,
        "input": [
            {
                "Accord Sedan, Accord Hybrid": 9
            },
            {
                "Accord Sedan, Pilot": 9
            },
            {
                "Accord Sedan, CR-V": 1
            },
            {
                "Accord Sedan, Element": 0.5
            },
            {
                "Accord Sedan, Odyssey": 5
            },
            {
                "Accord Hybrid, Pilot": 1
            },
            {
                "Accord Hybrid, CR-V": 0.1111111111111111
            },
            {
                "Accord Hybrid, Element": 0.1111111111111111
            },
            {
                "Accord Hybrid, Odyssey": 0.14285714285714285
            },
            {
                "Pilot, CR-V": 0.1111111111111111
            },
            {
                "Pilot, Element": 0.1111111111111111
            },
            {
                "Pilot, Odyssey": 0.14285714285714285
            },
            {
                "CR-V, Element": 0.5
            },
            {
                "CR-V, Odyssey": 5
            },
            {
                "Element, Odyssey": 6
            }
        ],
        "computed": null
    }
}
```

## Using AHPy

### Terminology

describe "target", 'element', 'value' 'weight'

https://www.edx.org/course/valoracion-de-futbolistas-con-el-metodo-ahp

### The Compare Object

The Compare class computes the priority vector and consistency ratio of a positive reciprocal matrix, created using an input dictionary of pairwise comparison values. Optimal values are computed for any missing pairwise comparisons. Compare objects can also be [linked together to form a hierarchy](#compareadd_children) representing the decision problem: global problem elements are then derived by synthesizing all levels of the hierarchy.

`Compare(name, comparisons, precision=4, random_index='dd', iterations=100, tolerance=0.0001, cr=True)`

- `name`: *str (required)*, the name of the Compare object
  - This property is used to link a child object to its parent and must be unique

- `comparisons`: *dict (required)*, the elements and values to be compared, provided in one of two forms:
  1. A dictionary of pairwise comparisons, in which each key is a tuple of two elements and each value is their pairwise comparison value
      - `{('a', 'b'): 3, ('b', 'c'): 2, ('a', 'c'): 5}`
      - **The order of the elements in the key matters: the comparison `('a', 'b'): 3` means "a is moderately more important than b"**
  2. A dictionary of measured values, in which each key is a single element and each value is that element's measured value
      - `{'a': 1.2, 'b': 2.3, 'c': 3.4}`
      - Given this form, AHPy will automatically create a consistent priority vector of normalized values

- `precision`: *int*, the number of decimal places to take into account when computing both the priority vector and the consistency ratio of the Compare object
  - The default precision value is 4

- `random_index`: *'dd'* or *'saaty'*, the set of random index estimates used to compute the priority vector's consistency ratio
  - 'dd' uses estimates from Donegan, H.A. and Dodd, F.J., 'A Note on Saaty's Random Indexes,' *Mathematical and Computer Modelling*, 15:10, 1991, pp. 135-137 (DOI: [10.1016/0895-7177(91)90098-R](https://doi.org/10.1016/0895-7177(91)90098-R))
    - 'dd' supports the computation of consistency ratios for matrices less than or equal to 100 &times; 100 in size
  - 'saaty' uses estimates from Saaty, T., *Theory And Applications Of The Analytic Network Process*, Pittsburgh: RWS Publications, 2005, p. 31
    - 'saaty' supports the computation of consistency ratios for matrices less than or equal to 15 &times; 15 in size
  - The default random index is 'dd'

- `iterations`: *int*, the stopping criterion for the algorithm used to compute the Compare object's priority vector
  - If a priority vector has not been determined after this number of iterations, the algorithm stops and the last principal eigenvector to be computed is assigned as the priority vector
  - The default number of iterations is 100

- `tolerance`: *float*, the stopping criterion for the cycling coordinates algorithm used to compute the optimal value of missing pairwise comparisons
  - The algorithm stops when the difference between the norms of two cycles of coordinates is less than this value
  - The default tolerance value is 0.0001

- `cr`: *bool*, whether to compute the priority vector's consistency ratio
  - Set `cr=False` to compute the priority vector of a matrix when a consistency ratio cannot be determined due to the size of the matrix
  - The default value is True

### Compare Properties

`consistency_ratio`
`local_weights`
`global_weights`
same as local weight if only object in hierarchy
`target_weights`

may not add up to 1 due to rounding

### Compare.add_children()

Compare objects can be linked together to form a hierarchy representing the decision problem. To link Compare objects together into a hierarchy, call `add_children()` on the Compare object intended to form the *upper* level (the *parent*) and include as an argument a list or tuple of one or more Compare objects intended to form its *lower* level (the *children*). **In order to properly synthesize the levels of the hierarchy, the `name` of each child object MUST appear as an element in its parent object's input `comparisons` dictionary.**

`Compare.add_children(children)`

- `children`: *list* or *tuple (required)*, the Compare objects that will form the lower level of the current Compare object

```python
>>> child1 = ahpy.Compare(name='child1', ...)
>>> child2 = ahpy.Compare(name='child2', ...)

>>> parent = ahpy.Compare(name='parent', comparisons={('child1', 'child2'): 5})
>>> parent.add_children([child1, child2])
```

The global and target weights of the Compare objects in a hierarchy are updated as the hierarchy is constructed: each time `add_children()` is called, the parent object's global weight is set to 1.0 and all of its descendants' global weights are updated accordingly. For this reason, the order in which the hierarchy is constructed is important. While it is possible to construct the hierarchy in any order, **it is best practice to construct the hierarchy beginning with the Compare objects on the lowest level and working up**. This will insure that the global weights of each lower level are always properly computed as the hierarchy is built.

The precision of the target weights are also updated as the hierarchy is constructed: each time `add_children()` is called, the precision of the parent object's target weights is set to equal the lowest precision of its child objects. Because lower precision propagates up through the hierarchy, the final target weights will always have the same level of precision as the hierachy's least precise Compare object. This also means that it is possible for the precision of a Compare object's target weights to be different from the precision of its local and global weights.

### Compare.complete()

**If the hierarchy is *not* constructed beginning with the Compare objects on the lowest level and working up, the final step of construction MUST be to call `complete()` on the Compare object at the hierarchy's highest level**. This will insure that both the target weights and the global weights of each Compare object in the hierarchy are correctly computed.

### Compare.report()

A report on the details of each Compare object is available. To return the report as a dictionary, call `report()` on the Compare object; to simultaneously print the information to the console in JSON format, set `show=True`.

`Compare.report(show=False)`

- `show`: *bool*, whether to print the report to the console in JSON format
  - The default value is False

The keys of the report take the following form:

- `name`: *str*, the name of the Compare object
- `weight`: *float*, the global weight of the Compare object in the hierarchy
- `target`: *dict*, the target weights of the elements in the lowest level of the hierarchy; each key is a single element and each value is that element's computed target weight; **if the global weight of the Compare object is less than 1.0, this value will be `None`**
    - `{'a': 0.5, 'b': 0.5}`
- `weights`: *dict*, the weights of the Compare object's elements
  - `local`: *dict*, the local weights of the Compare object's elements; each key is a single element and each value is that element's computed local weight
    - `{'a': 0.5, 'b': 0.5}`
  - `global`: *dict*, the global weights of the Compare object's elements; each key is a single element and each value is that element's computed global weight
    - `{'a': 0.25, 'b': 0.25}`
- `consistency_ratio`: *float*, the consistency ratio of the Compare object
- `random_index`: *'Donegan & Dodd' or 'Saaty'*, the random index used to compute the consistency ratio
- `elements`: *dict*, the elements compared by the Compare object
  - `count`: *int*, the number of elements compared by the Compare object
  - `names`: *list*, the names of the elements compared by the Compare object
- `children`: *dict*, the children of the Compare object; if the Compare object has no children, this value will be `None`
  - `count`: *int*, the number of the Compare object's children
  - `names`: *list*, the names of the Compare object's children
- `comparisons`: *dict*, the comparisons of the Compare object
  - `count`: *int*, the number of comparisons made by the Compare object, **not counting reciprocal comparisons**
  - `input`: *dict*, the comparisons input to the Compare object; this is identical to the input `comparisons` dictionary
  - `computed`: *dict*, the comparisons computed by the Compare object; each key is a tuple of two elements and each value is their computed pairwise comparison value; if the Compare object has no computed comparisons, this value will be `None`
    - `{('c', 'd'): 0.730297106886979}, ...}`

### Missing Pairwise Comparisons

When a Compare object is initialized, the elements forming the keys of the input `comparisons` dictionary are permuted. Permutations of elements that do not contain a value within the input `comparisons` dictionary are then optimally solved for using the cyclic coordinates algorithm described in Bozóki, S., Fülöp, J. and Rónyai, L., 'On optimal completion of incomplete pairwise comparison matrices,' *Mathematical and Computer Modelling*, 52:1–2, 2010, pp. 318-333 (DOI: [10.1016/j.mcm.2010.02.047](https://doi.org/10.1016/j.mcm.2010.02.047)).

The following example demonstrates this functionality using the matrix below. We first compute the target weights and consistency ratio for the complete matrix, then repeat the process after removing the **(c, d)** comparison marked in bold.

||a|b|c|d|
|-|:-:|:-:|:-:|:-:|
|a|1|1|5|2|
|b|1|1|3|4|
|c|1/5|1/3|1|**3/4**|
|d|1/2|1/4|**4/3**|1|

```python
>>> comparisons = {('a', 'b'): 1, ('a', 'c'): 5, ('a', 'd'): 2, ('b', 'c'): 3, ('b', 'd'): 4, ('c', 'd'): 3/4}

>>> complete = ahpy.Compare('complete', comparisons)
>>> print(complete.target_weights)
{'b': 0.3917, 'a': 0.3742, 'd': 0.1349, 'c': 0.0991}
>>> print(complete.consistency_ratio)
0.0372

>>> del comparisons[('c', 'd')]

>>> missing_cd = ahpy.Compare('missing_cd', comparisons)
>>> print(missing_cd.target_weights)
{'b': 0.392, 'a': 0.3738, 'd': 0.1357, 'c': 0.0985}
>>> print(missing_cd.consistency_ratio)
0.0372
```