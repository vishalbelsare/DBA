# DBA: Dynamic Time Warping Barycenter Averaging

DBA computes the average of a set of time series under Dynamic Time Warping (DTW). Unlike a simple arithmetic mean, which averages point-by-point and blurs out temporal variations, DBA first aligns each series to the current average using DTW, then updates each position as the mean of all values aligned to it. This produces an average that preserves the shape of the underlying pattern, even when series are shifted or warped in time.

**Arithmetic mean** — washes out the pattern because series are misaligned in time:

![arithmetic mean](https://raw.githubusercontent.com/fpetitjean/DBA/master/images/arithmetic.png)

**DBA** — recovers the shared pattern by aligning before averaging:

![DBA](https://raw.githubusercontent.com/fpetitjean/DBA/master/images/DBA.png)

## How it works

1. **Initialization**: find the medoid of the dataset (the series with the lowest sum of squared DTW distances to all others). For large datasets (>50 series), a random subset is sampled for efficiency.
2. **Iterative refinement**: repeat for *n* iterations:
   - For each series, compute the DTW alignment path to the current average.
   - For each position in the average, collect all values aligned to it across all series.
   - Update each position as the mean of its aligned values.

The algorithm converges to a local minimum of the sum of squared DTW distances — a meaningful centroid for time series clustering, classification, and data augmentation.

## Quick start (Python)

```python
import numpy as np
from DBA import performDBA

# series is a list of 1-D numpy arrays (can have different lengths)
series = [np.sin(np.linspace(0, 2*np.pi, 200) + phase) for phase in np.random.uniform(0, 1, 20)]

average = performDBA(series, n_iterations=10)
```

For **multivariate** time series (channels-first format, shape `(n_channels, length)`):

```python
from DBA_multivariate import performDBA

# series is a list of 2-D numpy arrays with shape (n_channels, length)
average = performDBA(series, n_iterations=10)
```

## Quick start (Java)

```java
double[][] sequences = /* your time series, can have different lengths */;
double[] average = DBA.performDBA(sequences, 10);
```

With a Sakoe-Chiba warping window:

```java
int w = 50; // warping window size
double[] average = DBAWarpingWindow.performDBA(sequences, w);
```

## Implementations

| File | Language | Multivariate | Warping window | Notes |
|------|----------|:---:|:---:|-------|
| `DBA.py` | Python (numpy) | | | **Primary implementation** — start here |
| `DBA_multivariate.py` | Python (numpy) | x | | Channels-first format `(n_channels, length)` |
| `cython/` | Cython | | x | Compiled extension for speed |
| `DBA.java` | Java | | | |
| `DBAWarpingWindow.java` | Java | | x | |
| `DBA.m` | Matlab | | | Max length 1000 |

All implementations support **variable-length** time series within the same dataset.

## Research papers

This code supports three research papers:

* **[Pattern Recognition 2011](http://francois-petitjean.com/Research/Petitjean2011-PR.pdf)**: A global averaging method for Dynamic Time Warping — introduces DBA
* **[ICDM 2014](http://francois-petitjean.com/Research/Petitjean2014-ICDM-DTW.pdf)**: Dynamic Time Warping Averaging of Time Series allows Faster and more Accurate Classification
* **[ICDM 2017](http://francois-petitjean.com/Research/ForestierPetitjean2017-ICDM.pdf)**: Generating synthetic time series to augment sparse datasets

When using this repository, please cite:

```bibtex
@article{Petitjean2011-DBA,
  title={A global averaging method for dynamic time warping, with applications to clustering},
  author={Petitjean, Fran{\c{c}}ois and Ketterlin, Alain and Gan{\c{c}}arski, Pierre},
  journal={Pattern Recognition},
  volume={44},
  number={3},
  pages={678--693},
  year={2011},
  publisher={Elsevier}
}

@inproceedings{Petitjean2014-ICDM,
  title={Dynamic time warping averaging of time series allows faster and more accurate classification},
  author={Petitjean, Fran{\c{c}}ois and Forestier, Germain and Webb, Geoffrey I and Nicholson, Ann E and Chen, Yanping and Keogh, Eamonn},
  booktitle={Data Mining (ICDM), 2014 IEEE International Conference on},
  pages={470--479},
  year={2014},
  organization={IEEE}
}

@inproceedings{Forestier2017-ICDM,
  title={Generating synthetic time series to augment sparse datasets},
  author={Forestier, Germain and Petitjean, Fran{\c{c}}ois and Dau, Hoang Anh and Webb, Geoffrey I and Keogh, Eamonn},
  booktitle={Data Mining (ICDM), 2017 IEEE International Conference on},
  pages={865--870},
  year={2017},
  organization={IEEE}
}
```

## License

GNU General Public License v3.0
