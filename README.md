# SMILES Generator using DeepSMILES LSTM and Reinforcement Learning with SMILES Predictor

The first part of this project involves a Generator for generating specified SMILES (Simplified Molecular Input Line Entry System) strings using a combined deep learning model. The model leverages deepSMILES, a more compact representation of SMILES and augment it with the SMILES Predictor MLP to enhance the generation process of the LSTM (Long Short Term Model). By combining the base trained LSTM Model with the classification of the SMILES Predictor as Reinforment Learning approach to improve the output of the LSTM and ensure similarity to a target class of molecules.


# SMILES Predictor using a MLP and Morgan Fingerprints, Molecule Descriptors for classification

The second part of this project involves a Predictor. The machine learning model is a Multi-Layer Perceptron (MLP) designed to predict whether a given molecule is an AXL kinase inhibitor based on its SMILES representation. The model leverages various molecular descriptors and Morgan fingerprints to classify the molecules.




## Table of Contents

1. [Introduction](#introduction)
1. [Setup and Installation](#setup-and-installation)
1. [Requirements](#requirements)
2. [Project Structure](#project-structure)
3. [Workflow](#workflow)
4. [SMILES Generator](#smiles-generator)
5. [SMILES Predictor](#smiles-predictor)




## Introduction

The combined approach makes it possible to generate not only a large number of syntactically correct and chemically valid SMILES, but also those that have a high probability of possessing relevant chemical properties. This is done by initially modelling a large number of SMILES data with the SMILES Generator (phase 1) and then fine-tuning them by means of value-driven optimisation using the SMILES Predictor (phase 2). In addition, generated molecules can then be validated separately with the SMILES Predictor for a final quality check (phase 3). This method can be particularly useful in drug design to discover new, potentially effective molecules.




## Setup and Installation

**Conda**

   - Please check out the [Conda Documentation](https://github.com/conda/conda-docs).

   - To execute all tasks in one single conda environment the `GeneratorPredictor.yaml` contains all required packages and the corresponding channels
   

     > Note: If you want to update your current environment manually you should add the following **conda packages**:
       >
       > ```bash   
       > - python
       > - cudatoolkit
       > - numpy
       > - pandas
       > - matplotlib
       > - scikit-learn
       > - rdkit
       > - tqdm
       > - pytorch
       > - torchvision
       > - torchmetrics
       > - deepsmiles
       > ```


   - Otherwise navigate to the location of the pulled AMA.yaml file and execute `conda env create -f GeneratorPredictor.yaml`


   - To activate the created conda environmentconda and getting started execute `conda activate GeneratorPredictor`


**Clone the repository:**
    ```bash
    git clone https://github.com/Schockwav3/SMILESGeneratorPredictor.git
    cd smiles-generation
    ```


## Requirements

- Set up all required data

- **Phase 1 SMILES Generator**

   - You need the input **chembl_smiles.txt** containing the 1.8 million small molecule compounds from the ChEMBL database. This is the dataset what I used to lead the SMILES Predictor to train the vocabular and the basic structure of molecules. It is already in the repository. If you want to use a different dataset for training feel free to change it in your project and update the `file_path`. 
   
> Note: The defined vocabulary is adjusted to the components of the molecules that occur most frequently in this data set. I have written and added a script `Vocabulary_Creator` that analyses other datasets and outputs which vocabulars are most common. With this information you can extend the vocabulary if necessary. 

   - Once the network has been pretrained on the basic dataset, it will be saved under the `save_model_path` and can then be further trained for Transfer Learning or phase 2.


> Note: It is also possible to use **Transfer Learning** to a second more specific dataset of SMILES. To do this, simply save the model after the first training and activate `use_pretrained_model = True`, adjust the parameters, define the second / new dataset as new `file_path` and run the code again.


- **Phase 2 SMILES Generator**

   - First you need the pretrained SMILES Generator model from phase 1, simply enter the path under `trained_lstm_path` in the second part of the code.

   - Then you need the pretrained SMILES Predictor model which you will also need in phase 3 for classification. Simply pause here and prepare the learning for the predictor model. Afterwards simply enter the path under `trained_mlp_path` in the second part of the code.

   - Once the network has been finally trained, it will be saved under `save_model_path` in the second part of the code. The fully trained SMILES Generator model can then be used to generate specific SMILES.


- **Phase 3 SMILES Predictor**

   -  You need two files as input. One file **smiles_data** containing different generic molecules enter the path under `smiles_data_path` and one file with targets **smiles_axl_inhibitors** that the model should learn to predict. In this case, we want to train the model to distinguish whether a molecule is an AXL kinase inhibitor or not. Update the path under `smiles_axl_inhibitors_path`.

    Target = 1: AXL kinase inhibitor
    Target = 0: Other molecules

> Note: You should search for as much target SMILES for your prediction class input as possible. In my example I found a total of 4564 on ChEMBL and NIH. For the generic smiles data I took a ratio of around 1:2 so a total of 10812 random non target SMILES. 


   - Once the network has been trained, it will be saved under `save_model_path`. The trained SMILES Predictor model can then be used to classify SMILES.




## Project Structure

This will give you a complete overview of the **SMILES GeneratorPredictor** project structure, all existing scripts in the repository and all required files:

![Project Structure](https://github.com/Schockwav3/SMILESGeneratorPredictor/blob/main/Pictures/project_structure.png)




## Workflow

This will give you a complete overview of the **SMILES GeneratorPredictor** Workflow:

<img src="https://github.com/Schockwav3/SMILESGeneratorPredictor/blob/main/Pictures/workflow.png" width="600" height="1360">




## SMILES Generator using DeepSMILES LSTM and Reinforcement Learning with SMILES Predictor


**Breakdown of the Code**


### Device Selection

The primary purpose of device selection is to determine whether the computations will be performed on a CPU or a GPU. The choice of device can significantly impact the training and inference speed of deep learning models, especially those involving large datasets and complex architectures.

- **Checking GPU Availability:** The code starts by checking if a CUDA-capable GPU is available on the machine. This is done using the function `torch.cuda.is_available()`.

- **If a GPU is Available:** If a CUDA-compatible GPU is found, the device is set to cuda using `torch.device("cuda")`. This setting indicates that the model and its computations will be transferred to the GPU, which can provide significant speed-ups due to its parallel processing capabilities.

- **If a GPU is Not Available:** If no compatible GPU is detected, the device is set to cpu using `torch.device("cpu")`. This means that all computations will be performed on the CPU.




### Define the Vocabulary

The FixedVocabulary class defines a fixed set of tokens for encoding SMILES sequences.

```bash  
           'PAD':      0,       # Padding token (for filling batches of unequal length)
           'UNK':      1,       # Undefined token (for unknown elements)
           '^':        2,       # Start token
           '$':        3,       # End token
           '3':        4,
           '4':        5,
           '5':        6,
           '6':        7,
           '7':        8,
           '8':        9,
           '9':        10,
           '%10':      11,
           '%11':      12,
           '%12':      13,
           '%13':      14,
           '%14':      15,
           '%15':      16,
           '%16':      17,
           '%17':      18,
           '%18':      19,
           '%19':      20,
           '%20':      21,
           '%21':      22,
           '%22':      23,
           '%23':      24,
           '%24':      25,
           ')':        26,
           '=':        27,
           '#':        28,
           '.':        29,
           '-':        30,
           '/':        31,
           '\\':       32, 
           'n':        33,
           'o':        34,
           'c':        35,
           's':        36,
           'N':        37,
           'O':        38,
           'C':        39,
           'S':        40,
           'F':        41,
           'P':        42,
           'I':        43,
           'B':        44,
           'Br':       45,
           '[C@]':     47,
           '[C@H]':    48,
           '[C@@]':    50,
           '[nH]':     51,
           '[O-]':     52,
           '[N+]':     53,
           '[n+]':     54,
           '[Na+]':    55,
           '[S+]':     56,
           '[Br-]':    57,
           '[I-]':     59,
           '[N-]':     60,
           '[Si]':     61,
           '[2H]':     62,
           '[K+]':     63,
           '[Se]':     64,
           '[P+]':     65,
           '[C-]':     66,
           '[se]':     67,
           '[Cl+3]:':  68,
           '[Li+]:':   69,      
```

> [!TIP]
If you need to update or change the FixedVocabulary you can use the sript in /src/Vocabulary_Creator.ipynb to analyze a file with SMILES and see which Tokens are used and how many of them are included to create a updated Vocabulary but for most use cases this Vocabulary should be fine.




### Tokanizer

- The DeepSMILESTokenizer class handles the transformation of SMILES into deepSMILES and performs tokenization and untokenization.

- The DeepSMILESTokenizer class uses several regular expressions to tokenize deepSMILES strings. Each regex pattern is designed to  match specific components of a SMILES string. Below are the regex patterns used and their purposes:

    - `brackets` groups characters within square brackets together, which can represent charged atoms or specific configurations in the SMILES syntax.

    - `2_ring_nums` matches numbers up to two digits preceded by a percent sign ("%"), used to denote ring closures in molecules with more than 9 rings.

    - `brcl` matches the halogen atoms bromine ("Br") and chlorine ("Cl"), ensuring they are recognized as unique tokens in the SMILES string. They are essential in drug molecules.


 ```bash
       "brackets": re.compile(r"(\[[^\]]*\])"),
       "2_ring_nums": re.compile(r"(%\d{2})"),
       "brcl": re.compile(r"(Br|Cl)")
 ```




### Define the LSTM Model (RNN)

- The LSTM base model is designed to handle the generation and manipulation of SMILES representations using an RNN (Recurrent Neural Network) architecture with LSTM (Long Short-Term Memory) cells.

- The architecture of the LSTM includes an `**embedding layer** which converts the input sequences into dense vectors, **LSTM layers** which processes these vectors sequentially and returns a sequence of outputs and a final **linear layer** at the end transforms the LSTM outputs back to the size of the vocabulary so that the probabilities for the next token can be calculated.

- Training process: 
    - During training, the model receives a SMILES sequence and learns to predict the next token in the sequence. The training loss is calculated by measuring the difference between the predicted and actual tokens.

- Sequence generation:
    - During generation, the model begins with a start token (^) and progressively predicted the next token until an end token ($) is reached or a maximum sequence length is reached.
    - `generate_deepsmiles(num_samples, max_length)`: Generates a specified number of deepSMILES sequences up to a maximum length, used for creating new molecular representations.
    - `convert_deepsmiles_to_smiles(deep_smiles_list)`: Converts a list of deepSMILES sequences back to SMILES format, making the output interpretable in a chemical context.

- The input parameters allow users to configure the model according to the complexity of the dataset and the computational resources available. The model's capability to load pretrained weights also facilitates fine-tuning (phase 2) and Transfer Learning, making it adaptable to new tasks with minimal retraining.

- Key Input Parameters:

   - `voc_size (int)`: The size of the vocabulary, which dictates the number of unique tokens the model can understand.

   - `layer_size (int)`: The number of units in each LSTM layer, determining the model's capacity.

   - `num_layers (int)`: The number of LSTM layers stacked in the model, affecting the depth and representational power.

   - `embedding_layer_size (int)`: The size of the embedding vectors that represent input tokens, influencing the richness of token representations.

   - `dropout (float)`: The dropout rate applied to prevent overfitting by randomly setting some LSTM outputs to zero during training.

   - `layer_normalization (bool)`: A flag indicating whether to apply layer normalization, which helps stabilize and accelerate training.




### Define Trainer

- The SmilesTrainer class is used to streamline the training process of the LSTM model. It handles data loading, model training, validation, and testing, as well as monitoring the training progress through loss and accuracy metrics. The class can utilize a pretrained model for fine-tuning (phase 2), making it versatile for different stages of model development. The plotting functions provide a clear visualization of the training dynamics, allowing for easy identification of issues such as overfitting or underfitting.

- NLLLoss:
    - `nn.NLLLoss()` stands for the **"Negative Log Likelihood Loss "**. It is a loss function that is often used in classification problems. It calculates the negative log-likelihood of the correct class. If the probability of the correct class is low, the log value becomes negative and large, resulting in a high loss. The loss is minimised by training the model to maximise the probability of correct classes.

- Calculate predictions:
    - `outputs.argmax(dim=-1)` selects the class with the highest log probability for each position in the sequence. The result is a tensor where each position contains the predicted class.

- Determine correct predictions:
    - `(predictions == targets).float()` compares the predictions with the actual target classes and produces a tensor containing 1 if the prediction is correct and 0 if it is incorrect.

- Calculate accuracy:
    - `correct.mean()` calculates the average of the correct predictions, which corresponds to the accuracy, i.e. the proportion of correctly predicted tokens.

- These methods are crucial for monitoring and optimising the training process as they provide insights into the performance of the model. Loss helps to determine the direction for updating the model parameters, while accuracy is a direct metric for the model's performance in terms of correct predictions.

- The division into the three data sets is crucial to ensure a fair and reliable evaluation of model performance. Without this, the model could be over-optimised on the training data and subsequently perform poorly on new data. 
    - `train` is used to train the model. The model learns patterns, relationships and structures in the data and adjusts its parameters to improve predictions.
    - `validation` is used to evaluate the model during training. They help to assess the performance of the model independently of the training data.
    - `test` is only used to evaluate the final model performance after the training has been completed.

- By plotting the loss and accuracy metrics over the training epochs, you can gain deeper insights into the behaviour of the model during the training process, evaluating model performance and react accordingly. 

- Key Input Parameters:

    - `epochs (int)`: The number of epochs (full passes through the training dataset) to train the model.

    - `learning_rate (float)`: The learning rate for the optimizer, controlling how much to adjust the model's weights with respect to the loss gradient.

    - `batch_size (int)`: The number of samples per batch to be loaded by the DataLoader.

    - `use_pretrained_model (bool)`: Indicates whether to use a pretrained model for the fine-tuning (phase 2) process.

    - `load_model_path (str)`: The file path to load a pretrained model's weights.

    - `save_model_path (str)`: The file path to save the trained model's weights.




### Dataset

- The SMILESDataset class is designed to preprocess SMILES data for the machine learning task. It efficiently handles data augmentation, tokenization, and encoding to prepare input-target pairs for model training. 

- Data augmentation enhances the diversity of the dataset, potentially improving model robustness. For each SMILES in the list, if `augment=True`, it generates multiple random valid SMILES representations of the same molecule using `randomize_smiles(self, smile: str, num_random: int)` method. The model can learn to recognize the molecule regardless of the specific SMILES string used.

- `load_data`: reads a file containing SMILES strings (one per line) and returns a list of these strings.

- `split_data`: splits a given dataset into three subsets: **training=0.7**, **validation=0.15**, and **test=0.15** sets.

- Key Input Parameters:

    - `augment (bool)`: A flag indicating whether to apply data augmentation to the SMILES strings.

    - `augment_factor (int)`: The number of augmented versions to generate for each SMILES string.




## SMILES Predictor using a MLP and Morgan Fingerprints, Molecule Descriptors for validation


## Descriptors

The following descriptors are calculated for each molecule:

- **Morgan Fingerprints**: A type of molecular fingerprint used for similarity searching, based on the circular substructures around each atom, and encoded as binary vectors. Morgan fingerprints are particularly useful in capturing the local environment of atoms within a molecule. They are generated by considering circular neighborhoods of varying radii around each atom, and then hashing these neighborhoods into fixed-length binary vectors.
- **AlogP**: A measure of the lipophilicity (fat-loving nature) of a molecule, representing its ability to dissolve in fats, oils, lipids, and non-polar solvents.
- **Polar Surface Area**: The total area of a molecule that is polar (i.e., capable of hydrogen bonding), influencing drug absorption and transport properties.
- **HBA (Hydrogen Bond Acceptors)**: The number of hydrogen bond acceptor sites within a molecule, affecting its solubility and interaction with biological targets.
- **HBD (Hydrogen Bond Donors)**: The number of hydrogen bond donor sites within a molecule, influencing solubility and biological interactions.
- **Bioactivities (pki Value)**: A measure of the potency of a molecule in inhibiting a specific biological target, expressed as the negative logarithm of the inhibition constant (Ki).
- **Chi0**: A molecular connectivity index representing the molecule's overall structure.
- **Kappa1**: A shape index representing the molecule's flexibility.
- **TPSA (Topological Polar Surface Area)**: The surface area of a molecule contributed by polar atoms, influencing drug absorption and permeability.
- **MolLogP**: The logarithm of the partition coefficient between water and octanol, representing a molecule's hydrophobicity.
- **PEOE_VSA1 to PEOE_VSA14**: A series of descriptors representing the van der Waals surface area of a molecule, divided into regions of different partial charges, which influence molecular interactions.
- **Molecular Weight**: The total mass of a molecule, influencing its distribution and elimination from the body.
- **NumRotatableBonds**: The number of bonds in a molecule that can rotate, affecting its flexibility and interaction with biological targets.
- **NumAromaticRings**: The number of aromatic (ring-shaped) structures within a molecule, influencing its stability and interaction with biological targets.
- **FractionCSP3**: The fraction of sp3 hybridized carbons in a molecule, representing the degree of saturation and three-dimensionality.
- **Polarizability**: A measure of how easily the electron cloud around a molecule can be distorted, influencing its interaction with electric fields and other molecules.
- **MolVolume**: The volume occupied by a molecule, affecting its density and interaction with biological environments.
- **MolWt**: Another term for molecular weight, indicating the total mass of a molecule.
- **HeavyAtomCount**: The number of non-hydrogen atoms in a molecule, influencing its size and reactivity.
- **NHOHCount**: The number of nitrogen, hydrogen, and oxygen atoms in a molecule, which can affect its polarity and reactivity.
- **NOCount**: The number of nitrogen and oxygen atoms in a molecule, influencing its hydrogen bonding and reactivity.
- **NumHeteroatoms**: The number of atoms in a molecule that are not carbon or hydrogen, affecting its reactivity and interaction with biological targets.
- **NumRadicalElectrons**: The number of unpaired electrons in a molecule, which can influence its reactivity and stability.
- **NumValenceElectrons**: The number of electrons in the outer shell of a molecule's atoms, determining its reactivity and bonding behavior.
- **RingCount**: The number of ring structures within a molecule, affecting its stability and interaction with biological targets.
- **BalabanJ**: A topological index representing the overall shape and branching of a molecule.
- **BertzCT**: A complexity index representing the structural complexity of a molecule.
- **Chi1**: A molecular connectivity index representing the molecule's overall structure, similar to Chi0 but focusing on different aspects.
- **Chi0n**: A molecular connectivity index representing the molecule's overall structure, focusing on nitrogen atoms.
- **Chi0v**: A molecular connectivity index representing the molecule's overall structure, focusing on valence electrons.
- **Chi1n**: A molecular connectivity index representing the molecule's overall structure, focusing on the first level of nitrogen atoms.
- **Chi1v**: A molecular connectivity index representing the molecule's overall structure, focusing on the first level of valence electrons.
- **Kappa2**: A shape index representing the molecule's flexibility and three-dimensionality.
- **Kappa3**: A shape index representing the molecule's overall three-dimensional structure.
- **HallKierAlpha**: An index representing the overall branching and complexity of a molecule's structure.


## Target Variable

The target variable indicates whether a molecule is an AXL kinase inhibitor or not:

- `Target = 1`: AXL kinase inhibitor
- `Target = 0`: Other molecules


## Steps to Create the Training Dataset

1. **Read SMILES Data**:
    - Two files are used: one (`smiles.txt`) containing various molecules, and another (`axl_inhibitors.txt`) with known AXL kinase inhibitors.

2. **Calculate Descriptors**:
    - For each molecule, various molecular descriptors are calculated.

3. **Set Target Variable**:
    - For molecules from `smiles.txt`, set `Target` to 0.
    - For molecules from `axl_inhibitors.txt`, set `Target` to 1.

## Example

Given the following data:

- `smiles.txt`:
    ```
    CCO
    CCC
    COC
    ```

- `axl_inhibitors.txt`:
    ```
    C1=CC=CC=C1
    C2=CCN=C(C)C2
    ```

The descriptors and targets are assigned as follows:

- `smiles.txt`:
    ```
    CCO: Descriptors + Target = 0
    CCC: Descriptors + Target = 0
    COC: Descriptors + Target = 0
    ```

- `axl_inhibitors.txt`:
    ```
    C1=CC=CC=C1: Descriptors + Target = 1
    C2=CCN=C(C)C2: Descriptors + Target = 1
    ```

## Goal of the Model

The model aims to utilize the descriptors to learn whether a molecule is an AXL kinase inhibitor (`Target = 1`) or not (`Target = 0`).


## Training and Validation

1. **Train-Test Split**:
    - The data is split into training and test datasets. The training set is used to train the model, and the test set is used to evaluate the model's performance.

2. **Model Architecture**:
    - A Multi-Layer Perceptron (MLP) is used to process the descriptors and learn to classify molecules as AXL kinase inhibitors or not.

3. **Training the Model**:
    - The model is trained on the training dataset, adjusting its weights to improve predictions.

4. **Validation**:
    - After training, the model is validated on the test dataset to assess its accuracy and performance.




## Results and Visualization

- The training and test losses are plotted to visualize the training progress.
- The accuracy for each label (AXL kinase inhibitors and other molecules) is plotted for both training and test datasets.
- Confusion matrices are plotted to visualize the model's performance on the training and test datasets.


## Saving and Loading the Model

- The trained model is saved to a file (`model_predictor.pth`) and can be loaded for further predictions or evaluations.

















## Results

The models generate SMILES strings based on the training data and validate their performance using known AXL kinase inhibitors. The results are plotted for loss and accuracy over epochs.

## References

- RDKit: Open-source cheminformatics software.
- DeepSMILES: An alternative compact representation of SMILES.
- PyTorch: An open-source machine learning library.




- `chembl_smiles.txt`: Contains 1.8 million molecule SMILES strings from Chembl Database for the base training of the LSTM
    - `ChemblDatensatzAXLKinase.txt`: Contains known AXL kinase inhibitors from Chembl / NIB Database for the training of the Siamse Network