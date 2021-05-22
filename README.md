# PrISM : Annotating Precision for Integrative Structure Models
PrISM is a package for visualizing regions of high and low precision in ensembles of integrative models. It uses autoencoders, a class of deep-learning networks, to annotate bead-wise precision in a scalable and efficient manner for ensembles of large macromolecular assemblies.

<img width="684" alt="Screenshot 2021-05-05 at 10 08 18 AM" src="https://user-images.githubusercontent.com/8314735/117098728-e8effa00-ad8c-11eb-8ed6-04485b2f9d36.png">

## Installation
PrISM requires [IMP](http://integrativemodeling.org) to be installed.
Ensure that the `sampcon` module is also installed (if installing IMP from source: clone `sampcon` from [github](https://github.com/salilab/imp-sampcon/) into its module directory and recompile IMP).

It also requires a list of python packages that can be installed with the following types of commands:

```
pip3 install --user -r install_requires.txt
or
pip install -r install_requires.txt
```

## Running PrISM

### Input formats
There are three ways to run PrISM, based on the inputs at hand.

1. `NPZ`: Runs PrISM directly on the output of the IMP analysis pipeline, after running `sampcon`(https://github.com/salilab/imp-sampcon/), the module to test sampling exhaustiveness and cluster sampled models generated using [IMP](http://integrativemodeling.org). For each output cluster, the superposed bead coordinates are stored in an `.npz` file (e.g., `cluster.0.superposed.npz` for cluster 0), which is passed as input to PriSM.  

2. `RMF`: Runs PrISM on a directory with a set of superposed integrative models in RMF format. The RMF format is commonly used to store integrative models generated by [IMP](http://integrativemodeling.org).

There are options to select by *subunit, resolution, and select specific protein/domains* (similar to options in `sampcon`).

Note that PrISM assumes that the models are structurally superposed and does not perform the superposition before calculating precision.

3. `PDB`: Runs PrISM on a directory with a set of superposed integrative models in PDB format. The PDB format is commonly used to store atomistic integrative models. Molecules in the PDB are converted to a bead representation with one residue per bead, with beads centered at the respective C-alpha coordinates (side-chain fluctuations are not considered).

By default *all protein chains* are selected.

Note that PrISM assumes that the models are structurally superposed and does not perform the superposition before calculating precision.

### Training Parameters
Parameters for training the deep neural network are specified in the YAML file ( [Sample test_config.yml](src/test_config.yml) ).
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

### Run command

Use `src/run_prism.py`to generate the precision for the given input. Use the `--help` option to generate descriptions of arguments.

## Run command examples

### Example 1. PrISM on NPZ file
To run PrISM on an NPZ file, use the following type of command. Here, we assume you are in the `example/1AVX_npz` directory:

```
$IMP/build/setup_environment.sh python ../../src/run_prism.py  --input 1AVX_all_models_coordinates.npz --output_dir output/ --type npz --config ../../src/test_config.yml --gpu 1
```

Here `$IMP` is the path to local installation of IMP (if compiled from source). If IMP has been installed using a binary installer, the `$IMP/build/setup_environment.sh` argument may be skipped.

This runs the training on the GPU on the given NPZ input file and generates an output `precision.txt` file.

To run the same command on the CPU instead, set the gpu flag to 0.

### Example 2.  PrISM on NPZ file with new hyperparameters

One can change the default size of the network or other learning parameters in the YAML file and rerun.
But the expensive step of input pre-processing need not be performed if PrISM has already been run once on the system. To skip input generation one can run as follows.

```
$IMP/build/setup_environment.sh python ../../src/run_prism.py  --input 1AVX_all_models_coordinates.npz --skip_input_generation 1 --output_dir output/ --type npz --config ../../src/test_config.yml --gpu 1
```

### Example 3. PrISM on RMF file
Here's an example with an RMF file. Assumes you are in `example/1AVX_rmf` directory.

```
$IMP/build/setup_environment.sh python ../../src/run_prism.py  --input rmfs/ --output_dir output/ --type rmf --config ../../src/test_config.yml --gpu 1
```

### Example 4. PrISM on RMF file and selected subunits

Here's another example with RMF and assuming you want to calculate precision on a given subunit only (perhaps because the rest of the subunits were fixed during modeling), and at a given resolution (bead-size in terms of number of residues per bead).

Here precision is calculated on the protein `B` on `1-residue` beads.

```
$IMP/build/setup_environment.sh python ../../src/run_prism.py  --input rmfs/ --output_dir output/ --type rmf --config ../../src/test_config.yml --gpu 1 --subunit B --resolution 1
```

More than one selected subunit can also be specified using the selection file as follows
```
$IMP/build/setup_environment.sh python ../../src/run_prism.py  --input rmfs/ --output_dir output/ --type rmf --config ../../src/test_config.yml --gpu 1 --selection selection.txt
```
**test this**

### Example 5. PrISM on PDB file

Here's an example with PDB files. Assumes you are in `example/1AVX_pdb` directory. By default all chains are selected and Calpha atoms are selected.

```
$IMP/build/setup_environment.sh python ../../src/run_prism.py  --input pdbs/ --output_dir output/ --type pdb --config ../../src/test_config.yml --gpu 1
```

## Getting the precision-colored model from PrISM
The output precision values are stored in a text file in the output directory called `precision.txt` with precision per bead (for NPZ/RMF input) or per residue (for PDB input). The annotated precision can be visualized by coloring the beads of a representative model (e.g., the cluster center model) using the following command.  

For `NPZ` input, the `-su`, `-r`, and `-sn` options should be **identical** to what was passed in the sampcon step `exhaust.py`.
For `RMF` input, the `-su`, `-r`, and `-sn` options should be **identical** to what was passed in the previous step to `run_prism.py`.

The representative model is specified by the `-i` option.

The `-o` option specifies the name of the output precision-colored RMF file.

### Example 1. NPZ / RMF input  
For e.g. in `example/1AVX_npz` or `example/1AVX_rmf3`

```
$IMP/build/setup_environment.sh python ../../src/color_precision.py -su B -r 1 -pf precision.txt -i cluster_center_model.rmf3 -o precision_colored_cluster_center_model.rmf3
```

For e.g. in `example/rnapol_npz`

```
$IMP/build/setup_environment.sh python ../../src/color_precision.py -sn selection.txt -r 1 -pf precision.txt -i cluster_center_model.rmf3 -o precision_colored_cluster_center_model.rmf3
```
(**test this**)

### Example 2. PDB input
For e.g. in `example/1AVX_pdb`   

```
$IMP/build/setup_environment.sh python ../../src/color_precision.py -pf precision.txt -i representative_model.pdb -o precision_colored_representative_model.rmf3
```

Note that we still visualize the output in an RMF file. This is because beads can be colored to show precision in RMFs. PDBs on the other hand do not store color information for residues, and one would need an additional Chimera script for coloring residues.  

## Visualizing the precision-colored model

The output RMF file, `precision_colored_cluster_center_model.rmf3` can be visualized in UCSF Chimera. It has a flat hierarchy with each bead named in the format `PROTEIN:COPY NUMBER:START_RESIDUE-END_RESIDUE` (RMF) or `PROTEIN:RESIDUE` (PDB).

1. It may be helpful to view this file along with the representative model simultaneously
2. One can hide/select a set of beads from this hierarchy using the RMF viewer.
3. [rmfalias](https://www.cgl.ucsf.edu/chimera/docs/UsersGuide/midas/rmfalias.html) might be helpful for selecting/unselection sets of beads.

## Tips to improve the usability of PrISM output

- In cases where an assembly has both fixed subunits and moving subunits, the superposition step in `sampcon` where each model is superposed to the cluster center produces a slight distortion (1-2 A) in the coordinates of the fixed subunit. This may result in a confusing PrISM output. To alleviate this, run `sampcon` and PrISM in the selection mode, specifying the moving subunits only in the selection.  
