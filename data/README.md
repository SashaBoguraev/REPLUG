# Data Overview

For this project, we use AllenAI's C4 dataset. It can be 
accessed [here](https://huggingface.co/datasets/allenai/c4). In particular we use 
the standard, english only version of the dataset. 

# Loading Data

We provide cells to load the dataset in our Jupyter Notebook in the 
`code/` folder.

# Nuance Compared to Original Paper

The original paper uses 36M sequences of data from [The 
Pile](https://pile.eleuther.ai/) for their datastore, and 800K sequences 
for training. In comparison, we use a fraction of this many due to Colab 
Memory limitation. Further, we do not use The Pile, as it has been taken 
down due to copyright. We use C4 instead due to its relative similarity in 
content 
to The Pile. 
 
