# lefschetz-family


## Description
This repository contains the topological data computed in [arxiv:2505.07685](https://arxiv.org/abs/2505.07685).
It contains two files `differential_operators.txt` and `topological_data.txt`.
The file `differential_operators.txt` contains lines x
The generation of the data relies on the the `lefschetz-family` package available at [https://github.com/ericpipha/lefschetz-family](https://github.com/ericpipha/lefschetz-family), please follow the installation instructions given there.
The list of operators was obtained from [The Calabi-Yau database](https://cydb.mathematik.uni-mainz.de/), see also [The Calabi-Yau cluster](https://cycluster.mpim-bonn.mpg.de/).

## How to load the data?

The topological data can be loaded in SageMath using
```python3
topological_invariants = {}
file = open("/path/to/repository/CYO_topological_data/topological_data.txt", "r")
for line in file:
    line = line.replace("\n", "").split(", ")
    key = line[0].replace("'","")
    N, M, chi, c2H, H3, alpha, delta, sigma = line[1:]
    topological_invariants[key] = {
        "N": int(N),
        "M": int(M),
        "chi": int(chi),
        "c2H": int(c2H),
        "H3": int(H3),
        "alpha": int(alpha),
        "delta": int(delta),
        "sigma": int(sigma),
    }
file.close()
```
This creates a dictionary object for which the keys are the new number of the operators, and the items are themselves dictionary containing the topological data, with keys `N`, `M`, `chi`, `c2H`, `H3`, `alpha`, `delta` and `sigma`.

The differential operators can be loaded in SageMath once OreAlgebra is installed, using
```python3
from ore_algebra import OreAlgebra
R.<t> = QQ[]
S.<Dt> = OreAlgebra(R)
operators = {}
file = open("/path/to/repository/CYO_topological_data/operators.txt", "r")
for line in file:
    line = line.replace("\n", "").split(", ")
    key = line[0].replace("'","")
    L = S(line[1])
    operators[key] = L
file.close()
```
This creates a dictionary object for which the keys are the new number of the operators and the items are the operators.


## I have an operator I would like to compute the invariants of. How can i do this?
You can do this using the `CalabiYauOperator` class of `lefschetz-family`.
```python3
from lefschetz-family import CalabiYauOperator
L = <your operator as an OreAlgebra element>
CYO = CalabiYauOperator(L, nbits=nbits, basepoint=basepoint)
```
where `nbits` is the working precision you want (`400` by default), and `basepoint` is a rational number that is not a singularity of the operator (picked automatically by default).
For now this only works for Calabi-Yau operators of order four.

This creates a `CalabiYauOperator` object, which has the following methods:
- `period_matrix`: deduces a period matrix at the basepoint for which the monodromy is integral. There are cases where the monodromy is over a number field. In this case the given period matrix is such that all the rational monodromy matrices are integral.
- `paths`: computes a generating set of the fundamental group of the projective line punctured at the singularities of the 
- `monodromy_from_periods`: computes the monodromy representation from the periods, given by the $4\times 4$ matrices of monodromy along the loops of `CYO.paths`
- `gamma_class_from_periods`: computes the change of basis between the scaled Frobenius basis of solutions of the operator at the maximal unipotent monodromy point 0 and the period matrix. This matrix is defined over a polynomial ring over Q of c, and c represents $\zeta(3)/(2\pi i)^3$.
- `periods_from_gamma_class`: conversely, computes the period at the basepoint from a given Gamma class.
- `cleanup`: takes a Gamma class matrix, and computes a change of basis that puts it in the standard form given in Conjecture 1 of [arxiv:2505.07685](https://arxiv.org/pdf/2505.07685).

An example of usage is
```python3
os.environ["SAGE_NUM_THREADS"] = '8' # lefschetz-family makes usage of parallelism to speed up the computation. Put here the number of cores you want to use.

from ore_algebra import OreAlgebra
R.<t> = QQ[]
S.<Dt> = OreAlgebra(R)
operators = {}
file = open("/path/to/repository/CYO_topological_data/operators.txt", "r")
for line in file:
    line = line.replace("\n", "").split(", ")
    key = line[0].replace("'","")
    L = S(line[1])
    operators[key] = L
file.close()

from lefschetz-family import CalabiYauOperator
CYO = CalabiYauOperator(operators["3.7"], nbits=800)
GC = CYO.gamma_class_from_periods(CYO.period_matrix)
GC = CYO.cleanup(GC) * GC
print("Gamma class:")
print(GC, end="\n\n") # prints the Gamma class
print("Monodromy matrices:")
for M in CYO.monodromy_from_periods(CYO.periods_from_gamma_class(GC)):
    print(M, end="\n\n") # prints the monodromy representation. All the entries are integers.
```

## Contact
For any question, bug or remark, please contact [eric.pichon@mis.mpg.de](mailto:eric.pichon@mis.mpg.de).