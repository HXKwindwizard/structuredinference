# structuredInf
Structured Inference Networks for Deep Kalman Filters 

## Goal
The goal of this package is to provide a black box algorithm inference algorithm to learn varying models of time-series data. 
Learning of the underlying model is performed using a recognition or  [inference network.](https://arxiv.org/abs/1401.4082)

## Model
The figure below describes a simple model of time-series data.
This method would be a good fit to perform inference and learning within your model when:
* You have enough training examples to learn the generative model
* You would like to have a method for fast posterior inference at train and test time 
* Your temporal generative model has Gaussian latent variables (mean/variance can be a nonlinear function of previous timestep's variables).

![DKF](images/dkf.png?raw=true "Deep Kalman Filter")

The code learns the model by optimizing the variational lower bound.

![ELBO](images/ELBO.png?raw=true width="500" height="70" "Evidence Lower Bound")

*Generative Model* 

* The latent variables z1...zT and the observations x1...xT describe the generative process for the data.
* The figure depicts a state space model for time-varying data. 
* The emission and transition functions may be pre-specified to have a fixed functional form or be learned as a function parameterized by a deep neural network

*Inference Model* 

The box q(z1..zT | x1...xT) represents the inference network. There are several supported inference networks within this package. 
* Inference implemented with a bi-directional LSTM
* Inference implemented with an LSTM conditioned on observations in the future
* Inference implemented with an LSTM conditioned on observations from the past

## Installation

### Requirements
This package has the following requirements:

python2.7

[Theano](https://github.com/Theano/Theano)
Used for automatic differentiations

[theanomodels] (https://github.com/clinicalml/theanomodels) 
Wrapper around theano that takes care of bookkeeping, saving/loading models etc. Clone the github repository
and add it to your PATH environment variable so that it is accessible by this package. 

[pykalman] (https://pykalman.github.io/) 
[Optional: For running baseline UKFs/KFs]

An NVIDIA GPU w/ atleast 6G of memory is recommended.

Once the requirements have been met, clone this repository and it's ready to run. 

### Folder Structure
The following folders contain code to reproduct the results reported in our paper:
* expt-synthetic, expt-polyphonic: Contains code and instructions for reproducing results. 
* baselines/: Contains to run some of the baseline algorithms on the synthetic data
* ipynb/: Ipython notebooks for evaluation and building plots

The main files of interest are:
* parse_args_dkf.py: Contains the list of arguments that the model expects to be present. Looking through it is useful to understand the different knobs available to tune the model. 
* stinfmodel/dkf.py: Contains the code to construct the inference and generative model. The code is commented and should be readable.
* stinfmodel/evaluate.py: Contains code to evaluate the Deep Kalman Filter's performance during learning.
* stinfmodel/learning.py: Code for performing stochastic gradient ascent in the Evidence Lower Bound. 

## Dataset

We use numpy tensors to store the datasets with binary numpy masks to allow batch sizes comprising sequences of variable length. We train the models using mini-batch gradient descent on -ELBO. 

### Format 

The code to run on polyphonic and synthetic datasets has already been created in the theanomodels repository. See theanomodels/datasets/load.py for how the dataset is created and loaded. 

The datasets are stored in three dimensional numpy tensors. 
To deal with datapoints
of different lengths, we use numpy matrices comprised of binary masks. There may be different choices
to manipulate data that you may adopt depending on your needs and this is merely a guideline.

```
assert type(dataset) is dict,'Expecting dictionary'
dataset['train'] # N_train x T_train_max x dim_observation : training data
dataset['test']  # N_test  x T_test_max  x dim_observation : validation data
dataset['valid'] # N_valid x T_valid_max x dim_observation : test data
dataset['mask_train'] # N_train x T_train_max : training masks
dataset['mask_test']  # N_test  x T_test_max  : validation masks
dataset['mask_valid'] # N_valid x T_valid_max : test masks
dataset['data_type'] # real/binary
dataset['has_masks'] # true/false
```

During learning, we select a minibatch of these tensors to update the weights of the model. 


### Running on different datasets
To run the models on different datasets, create a file to load the dataset into a format that is similar to the above and
follow the setup in expt-polyphonic/train.py to create the training script. 

**See the folder expt-template for an example of how to create and run the code on your data**



## References: 
```
@article{krishnan2015deep,
  title={Deep Kalman Filters},
  author={Krishnan, Rahul G and Shalit, Uri and Sontag, David},
  journal={arXiv preprint arXiv:1511.05121},
  year={2015}
}
```
