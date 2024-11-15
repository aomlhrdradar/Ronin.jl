# Ronin.jl

Ronin.jl (Random forest Optimized Nonmeteorological IdentificatioN) contains a julia implementation of the algorithm described in [DesRosiers and Bell 2023](https://journals.ametsoc.org/view/journals/aies/aop/AIES-D-23-0064.1/AIES-D-23-0064.1.xml) for removing non-meteoroloigcal gates from airborne radar scans. Care has been taken to ensure relative similarity to the form described in the manuscript, but some changes have been made in the interest of computational speed. 

A key part of the process is computing necessary derived parameters from the raw radar moments, which may be custom-specified in a parameters file. Many of the relevant functions for these calculations are contained within [Ronin.jl](./src/Ronin.jl). 

  <br> 

  ___
  # Acknowledgments 
  
  Much of the data used to train the models in this repository is the product of arduous manual editing of radar scans. ELDORA data is provided by the authors of [Bell, Lee, Wolff, & Cai 2013](https://journals.ametsoc.org/view/journals/apme/52/11/jamc-d-12-0283.1.xml?tab_body=fulltext-display). NOAA P3 TDR Data is courtsey of Dr. Paul Reasor, Dr. John Gamache, and Kelly Neighbour. As mentioned above, the code is adapted from the original work of Dr. Alex DesRosiers. 
___
# Getting Started:
## Setting up the environment (CSU)
After cloning the repository, start Julia using Ronin as the project directory, either by calling 
```
julia --project=Ronin
```
from the parent directory of `Ronin` or modifying the `JULIA_PROJECT` environment variable. <br>
Then, enter package mode in the REPL by pressing `]`.<br>
<br><br>
Next, run `instantiate` to download the necessary dependencies. This should serve both to download/install dependencies and precompile the Ronin package. Now, exit package using the dlete key. To ensure that everything was installed properly, run `using Ronin` on the Julia REPL. No errors or information should print out if successful. Run `add iJulia` if you will be viewing the code in a Jupyter notebook and need access to the Jupyter kernel.
> Guide adaped from https://github.com/mmbell/Scythe.jl/tree/main
>
## Setting up the environment (Derecho)
### Getting Julia
export JULIA_DEPOT_PATH=$SCRATCH/julia <br>
curl -fsSL https://install.julialang.org | sh
<br>


Now, exit package mode using the delete key. To ensure that everything was installed properly, run `using Ronin` on the Julia REPL. No errors or information should print out if successful. 
> Guide adapted from https://github.com/mmbell/Scythe.jl/tree/main

## Example notebook 
<br>

If you're looking to jump right in, check out [Ronin Example Notebook](./Ronin_example.ipynb) - it contains everything you need to get up and running.
<br><br><br>

___
## Guide: Processing new data, training, and evaluating a new model
___
  <br>
  
The first step in training a new random forest model is determining which portions of the data will be used for training, testing, and validation. A helpful function here is `split_training_testing!` - this can be used to automatically split a collection of scans into a training directory and a testing directory. In order for the script to be configured properly, the variables relating to the different paths must be modified by the user - this is shown in the example notebook. 
<br> <br>The current configuration is consistent with the 80/20 training/testing split described in the manuscript, as well as to have an equal number of scans from each "case" represented in the testing set. It is expected that the script would work for different training/testing splits, but this has not yet been tested. <br><br>

Once the training and testing scans have been placed into separate directories, data processing may begin. `calculate_features` will be the primary function utilized here. The script processes a directory (or single scan) of scans, and outputs the calculated features into an .h5 file, with the desired features specified by the user in a text file. <br><br>

For the case where training scans are located within `/cfradials/training/`, the desired features to be calculated are specified in `features.txt`, and you wish to output the input features to `training_set.h5`, invoke the function as 
```
calculate_features("/cfradials/training", "features.txt", "training_set.h5")
```
If you wish to remove a validation set from the training dataset, utilize `remove_validation` 
<br><br>


Finally, we can train a model to process our data. To do so, utilize `train_model`. If training data is contained within `training_set.h5`, and you wish to name your trained model `trained_model.jld2`, invoke as follows. It's recommended to end the model name in `.jld2` as this is the method used to serialzied it to disk. 
```
train_model("training_set.h5", "trained_model.jld2")
```
<b>NOTE: This may take on the order of 20-30 minutes if running on the entire ELDORA set.</b><br><br>
This script also includes the option to verify the model on the training set and output the results to a separate h5 file. If you wish to do this, execute the same as above, but include the keyword argument `verify=true`<br><br>
## Evaluating the model <br>
Now - let's apply the trained model on a set of data. The useful function here is `QC_scan`.  In order, pass it arguments of the input location, the configuration file, and the path to the trained model. For this reason, it's important to <b>keep the configuration file used to calculate input features in a known location</b>. 

The function will calculate the necessary input features, apply the Random Forest model, and apply the resulting prediction the fields specified by keyword argument `VARIABLES_TO_QC`. These new variables will then be written back out into the specified netcdf file under the field name concatenated with keyword argument `QC_suffix`. If this name is already in use in the NetCDF, it will be overwritten. 
___

## Notes on data conventions
_______
Some important data convetions to make note of: 

* **Meteorological Data is referred to by 1 or `true`**
* **Non-Meteorological Data is referred to by 0 or `false`**
* **ELDORA**scan variable names: 
    * Raw Velocity: **VV**
    * QC'ed Velocity (Used for ground truth): **VG**
    * Raw Reflectivity: **ZZ**
    * QC'ed Reflectivity (Used for ground truth): **DBZ**
    * Normalized Coherent Power/Signal Quality Index: **NCP**
* **NOAA TDR** scan variable names: 
    * Raw Velocity: **VEL**
    * QC'ed Velocity (Used for ground truth): **VE**
    * Raw Reflectivity: **DBZ**
    * QC'ed Reflectivity: **DZ**
    * Normalized Coherent Power/Signal Quality Index: **SQI**


