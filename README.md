# Bdot
## Fast Matrix Multiply on Pretty Big Data

[![Build Status](https://travis-ci.org/tailwind/bdot.svg?branch=master)](https://travis-ci.org/tailwind/bdot)

Bdot does big dot products (by making your RAM bigger on the inside). It's based on [Bcolz](https://github.com/Blosc/bcolz/) and includes transparent disk-based storage.

![Bigger on the Inside](https://31.media.tumblr.com/dcd82ee9cc541ef6774572e9110de082/tumblr_inline_n3eq30Vjhh1rnbe7i.gif)


Supports `matrix . vector` and `matrix . matrix` for most common numpy numeric data types (`numpy.int64`, `numpy.int32`, `numpy.float64`, `numpy.float32`)

## Install
```bash
pip install bdot
```

or build from source (requires bcolz >= 0.9.0)

```bash
python setup.py build_ext --inplace
python setup.py install
```

## Usage

### Matrix . Vector

Multiply a matrix (`carray`) with a vector (`numpy.ndarray`), returns a vector (`numpy.ndarray`)

```python
import bdot
import numpy as np

matrix = np.random.random_integers(0, 12000, size=(300000, 100))
bcarray = bdot.carray(matrix, chunklen=2**13, cparams=bdot.cparams(clevel=2))

v = bcarray[0]

result = bcarray.dot(v)
expected = matrix.dot(v)

# should return True
(expected == result).all()

```

### Matrix . Matrix

Multiply a matrix (`carray`) with the transpose of a matrix (`carray`), returns a matrix (`carray`)

```python
import bdot
import numpy as np

matrix = np.random.random_integers(0, 120, size=(1000, 100))
bcarray1 = bdot.carray(matrix, chunklen=2**9, cparams=bdot.cparams(clevel=2))
bcarray2 = bdot.carray(matrix, chunklen=2**9, cparams=bdot.cparams(clevel=2))

# calculates bcarray1 . bcarray2.T (transpose)
result = bcarray1.dot(bcarray2)
expected = matrix.dot(matrix.T)

# should return True
(expected == result).all()

```
### Save Result to Disk (Experimental)
Save really big results directly to disk

```python
# create correctly sized container (helper method, not required)
output = bcarray1.empty_dot(bcarray2, rootdir='/path/to/bcolz/output')

# generate results directly on disk
bcarray1.dot(bcarray2, out=output)

# make sure the last bits get written
output.flush()
```

This method can also be used to get `carray` output for `ndarray` vector input, just leave off the `rootdir` parameter in `empty_dot`, or create your own `carray` container.
 
## Test

```python
nosetests bdot
```

## Simple Benchmarks

Benchmarks were done on data structures generated by the above code, are very informal, and vary a bit across data sets.

### Space

* `numpy` ~229MB
* `bdot` ~64MB

compression ratio: 3.5

### Time

* `numpy` ~33 ms
* `bdot` ~48 ms

percent performance: 68%

## Goals

This project has three goals, each slightly more fantastic than the last:

1. Allow computation on (compressed) data which is (~5-10x) larger than RAM at approximately the same speed as `numpy.dot`


2. Allow computation on (slightly compressed) data at speeds that improve on `numpy.dot`


3. Allow computation on (compressed) data which resides on disk at some sizable percentage (~50-30%) of the speed of `numpy.dot`


So far, the first goal has been met.


## Acknowledgements

This library wouldn't be possible without all the talented people who worked hard to create [Bcolz](https://github.com/Blosc/bcolz/) (and the libraries on which it's based). Initial code was also heavily influenced by [Bquery](https://github.com/visualfabriq/bquery).

Awesome TARDIS can be found [here](https://youtu.be/dUBxHd3bMhg?t=1m5s)
