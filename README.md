# Topological data for Gamma class formula of Calabi-Yau operators of order four

## Description
This repository contains the topological data computed in [arxiv:2505.07685](https://arxiv.org/abs/2505.07685).
It contains two files `differential_operators.txt` and `topological_data.txt` (which really are `csv` files).
Each line of `topological_data.txt` contains, in order, the new number of the operator and the invariants `N`, `M`, `chi`, `c2H`, `H3`, `alpha`, `delta` and `sigma`.

The generation of the data relies on the the `lefschetz_family` package available at [https://github.com/ericpipha/lefschetz-family](https://github.com/ericpipha/lefschetz-family), please follow the installation instructions given there.

The list of operators was obtained from [The Calabi-Yau database](https://cydb.mathematik.uni-mainz.de/), see also [The Calabi-Yau cluster](https://cycluster.mpim-bonn.mpg.de/).

> [!TIP]
> The topological invariants are not always unique. It is possible that a rational change of basis preserves integral monodromy. We give an example below

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
        "N": ZZ(N),
        "M": ZZ(M),
        "chi": ZZ(chi),
        "c2H": ZZ(c2H),
        "H3": ZZ(H3),
        "alpha": ZZ(alpha),
        "delta": ZZ(delta),
        "sigma": ZZ(sigma),
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
You can do this using the `CalabiYauOperator` class of `lefschetz_family`.
```python3
from lefschetz_family import CalabiYauOperator
L = <your operator as an OreAlgebra element>
CYO = CalabiYauOperator(L, nbits=nbits, basepoint=basepoint)
```
where `nbits` is the working precision you want (`400` by default), and `basepoint` is a rational number that is not a singularity of the operator (picked automatically by default).
For now this only works for Calabi-Yau operators of order four.

This creates a `CalabiYauOperator` object, which has the following methods:
- `period_matrix`: deduces a period matrix at the basepoint for which the monodromy is integral. There are cases where the monodromy is over a number field. In this case the given period matrix is such that all the rational monodromy matrices are integral.
- `paths`: computes a generating set of the fundamental group of the projective line punctured at the singularities of the 
- `monodromy_from_periods(period_matrix)`: computes the monodromy representation from the periods, given by the $4\times 4$ matrices of monodromy along the loops of `CYO.paths`
- `gamma_class_from_periods(period_matrix)`: computes the change of basis between the scaled Frobenius basis of solutions of the operator at the maximal unipotent monodromy point 0 and the period matrix. This matrix is defined over a polynomial ring over $\mathbb Q[c]$, and c represents $\zeta(3)/(2\pi i)^3$.
- `periods_from_gamma_class(gamma_class)`: conversely, computes the period at the basepoint from a given Gamma class.
- `cleanup(gamma_class)`: takes a Gamma class matrix, and computes a change of basis that puts it in the standard form given in Conjecture 1 of [arxiv:2505.07685](https://arxiv.org/pdf/2505.07685).
> [!CAUTION]
> The change of basis acts on the left of the Gamma class matrix, but it acts by its transpose on the right on the period matrices.
- `saturate(list_of_monodromy_matrices)`: from a set of rational monodromy matrices, produces a change of basis on the period matrix (acting on the right) that makes the monodromy matrices integral

An example of usage is
```python3
os.environ["SAGE_NUM_THREADS"] = '8' # lefschetz_family makes usage of parallelism to speed up the computation. Put here the number of cores you want to use.

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

from lefschetz_family import CalabiYauOperator
CYO = CalabiYauOperator(operators["3.7"], nbits=800)
GC = CYO.gamma_class_from_periods(CYO.period_matrix)
GC = CYO.cleanup(GC) * GC
print("Gamma class:")
print(GC, end="\n\n") # prints the Gamma class
print("Monodromy matrices:")
for M in CYO.monodromy_from_periods(CYO.periods_from_gamma_class(GC)):
    print(M, end="\n\n") # prints the monodromy representation. All the entries are integers.
```

Comparing this data to `topological_invariants["3.7"]`, one sees that there is a discrepancy. 
This is because there is a sublattice of the homology that is preserved under monodromy, as shown by the following computation.
```python3
monodromy_matrices = CYO.monodromy_from_periods(CYO.periods_from_gamma_class(GC) * diagonal_matrix([1,1,1,1/3])) # this is the *a priori* rational monodromy in the new basis
change_basis_saturation = CYO.saturate(monodromy_matrices) # we make the monodromy integral again
print(CYO.gamma_class_from_periods(CYO.periods_from_gamma_class(GC) * diagonal_matrix([1,1,1,1/3]) * change_basis_saturation)) # this is the new Gamma class matrix
```

## I want to compute the gamma class from topological invariants
Use the following code:
```python3
R.<c> = QQ[]
N, M, chi, c2H, H3, alpha, delta, sigma = <your data here>
GC = matrix([
    [chi*c - alpha/2*c2H/24 -delta/2, M*c2H/24, alpha/2*H3/2, M*H3/6],
    [c2H/24, N*sigma/2, -H3/2, 0],
    [1,0,0,0],
    [alpha/2*N/M, N, 0, 0]
])
```


## Contact
For any question, bug or remark, please contact [eric.pichon@mis.mpg.de](mailto:eric.pichon@mis.mpg.de).
