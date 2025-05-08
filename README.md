# NeuralMie and TAMie

This is the code repository for the NeuralMie aerosol optics emulator and the TAMie Mie optics code. The NeuralMie emulator is a neural network physics emulator that closely approximates the bulk scattering properties of populations of small particles. It was developed for use in climate and weather models to simulate the optical properties of atmospheric aerosol populations. TAMie is a fast Python-based Mie scattering code that can be used to calculate the scattering properties of individual particles, modeled as either homogeneous or coated spheres, and was used to generate the training data for NeuralMie. There is a [corresponding paper](https://doi.org/10.5194/gmd-2024-30) that describes and tests the algorithm in detail [1].


## Using TAMie

The entire TAMie scattering code is contained in the TAMie.py file. There are two primary subroutines:

`Qe, Qs, g = sphere(m, x)` Simulates Mie scattering for a homogeneous sphere. This function takes the complex refractive index of the particle (m) and the size parameter (x = 2&pi;r / &lambda;) and returns the extinction (Q<sub>e</sub>) and scattering (Q<sub>s</sub>) efficiency and the asymmetry parameter (g). Either sign convention can be used for the imaginary part of the refractive index. The absorption efficiency can be computed from these outputs as Q<sub>a</sub> = Q<sub>e</sub>-Q<sub>s</sub>.

`Qe, Qs, g = coreshell(mc, ms, xc, xs)` This is a Python implementation of Toon and Ackerman Mie scattering algorithm for coated spheres [2]. It produces the same outputs as the 'sphere' function and takes the refractive index and size parameter of the core (m<sub>c</sub> and x<sub>c</sub>) and the refractive index and size parameter of the shell (m<sub>s</sub> and x<sub>s</sub>) as arguments.

TAMie has only two dependencies:<br />
`numpy` (developed on version 1.23.5)<br />
`numba` (developed on version 0.56.4)

The code can be run without numba by removing the import statement and `@njit` function decorators but this will result in significantly slower performance.

The subroutine `qqg(an, bn, x)` takes Mie coefficients and a size parameter as arguments and determines the outputs for the 'sphere' and 'coreshell' functions. This subroutine can potentially be modified to add functionality, e.g. to return backscattering efficiency or absorption efficiency directly.

An example of calling TAMie from a Python script:

```python
from TAMie import sphere

m = 1.5 + 1j*0.01  #an example refractive index
x = 10             #an example size parameter

Qe, Qs, g = sphere(m,x)
```


## Using NeuralMie

NeuralMie consists of two pre-trained neural networks that emulate bulk optics for a population of either log-normally distributed homogeneous spheres or coated spheres. We have provided the models in two different save formats:

`sphere.h5` - Saved Keras version of the model for scattering by homogeneous spheres.<br />
`sphere.txt` - The sphere model saved in Fortran Keras bridge format [3].<br />
`coreshell.h5` - Saved Keras version of the model for scattering by concentric spheres.<br />
`coreshell.txt` - The core-shell scattering model saved in Fortran Keras bridge format.

There are two Python scripts provided that show examples of how to load and inference the models using Keras. It is important that the exact pre- and post-processing procedures shown in these scripts be followed when deploying the model. They also include several test cases and code to apply a Rayleigh approximation when appropriate.

`sphere_inference_demo.py` <br />
`coreshell_inference_demo.py`




## Training NeuralMie

The remaining scripts in the repository are used to generate scattering data and train NeuralMie. These are not necessary to use NeuralMie or TAMie, and are included as a supplement to [1].

`bulk_optics.py` - Contains subroutines for calculating the bulk optical properties of a log-normally distributed aerosol population based on the optical properties of individual particles calculated with TAMie.

`create_datasets.py` - This script generates training data for NeuralMie by generating random hypothetical aerosol populations and calling the `bulk_optics.py` code to determine their optical properties.

`neural_networks.py` - Contains functions for defining neural networks, generating neural networks with randomized hyperparameters for hyperparameter search, several metrics and loss functions used during training, and subroutines to import and prepare the training data and postprocess the neural network outputs.

`train_final_ann.py` - Script for training the final versions of the neural networks.

`train_random_ann.py` - Script for performing random hyperparameter search.

`utils.py` - Contains information about the range of plausible refractive indices and wavelengths that might occur in EAM / MAM and contains functions for generating random training samples.


## References

[[1](https://doi.org/10.5194/gmd-2024-30)] Geiss, A. and P.-L. Ma: NeuralMie (v1.0): An Aerosol Optics Emulator, Geosci. Model Dev. Discuss. [preprint], in review, 2024.

[[2](https://opg.optica.org/ao/abstract.cfm?uri=ao-20-20-3657)]  Toon, O. B. and T. P. Ackerman: Algorithms for the calculation of scattering by stratified spheres, Appl. Opt. 20, 3657-3660, 1981. 

[[3](https://github.com/scientific-computing/FKB)]   Ott, J., M. Pritchard, N. Best, E. Linstead, M. Curcic, and P. Baldi: A Fortran-Keras Deep Learning Bridge for Scientific Computing, arXiv:2004.10652, 2020.




## Disclaimer

This material was prepared as an account of work sponsored by an agency of the United States Government.  Neither the United States Government nor the United States Department of Energy, nor Battelle, nor any of their employees, nor any jurisdiction or organization that has cooperated in the development of these materials, makes any warranty, express or implied, or assumes any legal liability or responsibility for the accuracy, completeness, or usefulness or any information, apparatus, product, software, or process disclosed, or represents that its use would not infringe privately owned rights.
Reference herein to any specific commercial product, process, or service by trade name, trademark, manufacturer, or otherwise does not necessarily constitute or imply its endorsement, recommendation, or favoring by the United States Government or any agency thereof, or Battelle Memorial Institute. The views and opinions of authors expressed herein do not necessarily state or reflect those of the United States Government or any agency thereof.

<p align="center">
PACIFIC NORTHWEST NATIONAL LABORATORY<br />
<i>operated by</i><br />
BATTELLE<br />
<i>for the</i><br />
UNITED STATES DEPARTMENT OF ENERGY<br />
<i>under Contract DE-AC05-76RL01830</i><br />
</p>
