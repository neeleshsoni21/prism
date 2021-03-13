# PrISM : Annotating Precision for Integrative Structure Models
PrISM is a package for visualizing regions of high and low precision in ensembles of integrative models. It uses autoencoders, a class of deep-learning networks, to annotate bead-wise precision in a scalable and efficient manner for ensembles of large macromolecular assemblies.

## Installation
PrISM requires [IMP](http://integrativemodeling.org) to be installed.

It also requires a list of python packages that can be installed with the following types of commands:

```
pip3 install --user -r install_requires.txt
or
pip install -r install_requires.txt
```

## Running PriSM

### Inputs
There are three ways to run PrISM, based on the inputs at hand.

1. `NPZ`: Runs PrISM directly on the output of the IMP analysis pipeline, after running `imp-sampcon`, the module to test sampling exhaustiveness and cluster sampled models generated using [IMP](http://integrativemodeling.org). For each output cluster, the superposed bead coordinates are stored in an `.npz` file (e.g., `cluster.0.superposed.npz` for cluster 0), which is passed as input to PriSM.  

2. `RMF`: Runs PrISM on a directory with a set of superposed integrative models in RMF format.
Note that PrISM assumes that the models are structurally superposed and does not perform the superposition before calculating precision.
The RMF format is commonly used to store integrative models generated by [IMP](http://integrativemodeling.org).

3. `PDB`: Runs PrISM on a directory with a set of superposed integrative models in PDB format.
Note that PrISM assumes that the models are structurally superposed and does not perform the superposition before calculating precision.
The PDB format is commonly used to store atomistic integrative models.

### Training Parameters
Parameters for training the deep neural network are specified in the YAML file ([Sample test_config.yml](src/test_config.yaml)).
A brief description of each is provided in the comments below.

```
model_desc: Default PrISM hyperparameters established on benchmarks.
model_params:
  encoder_depth: 2 # number of layers in the encoder network
  decoder_depth: 3 # number of layers in the decoder network
  latent_dim: 64 # number of neurons in the latent space
lr: 1e-3 # learning rate, controls how much the model is changed in response to error each time
batch_size: 64 # number of inputs trained at once, larger is better. This depends on the available system memory.
max_epochs: 50 # number of steps to train the network.
```

### Run command examples

Run PrISM like this, initially:

```
$IMP/build/setup_environment.sh python run_prism.py
```

Here `$IMP` is the path to local installation of IMP (if compiled from source). If IMP has been installed using a binary installer, the `$IMP/build/setup_environment.sh` argument may be skipped.

To change params and run: One may choose to increase the size of the network if a large assembly is being modeled.



To run on a selected subunit only:








### Using the output
The output precision values are stored in a text file ``.

`color_precision.py`
