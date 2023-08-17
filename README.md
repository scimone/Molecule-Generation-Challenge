# Molecule Generation Challenge

This repository contains code and documentation for participating at a molecule generation challenge. The challenge is to generate a set of new molecules based on a training dataset that contains existing molecules represented as SMILES strings. The goal is to generate molecules with validity, novelty, and uniqueness, all scoring above 0.9. The primary evaluation metric used is the Fréchet ChemNet Distance (FCD), which measures the distance between the generated molecules and the molecules from the training dataset.
The training dataset is not included in this repository due to copyright reasons.

## Approach
Use the individual padded molecules as training inputs for a next character LSTM.

## 1. Preprocessing

The original string from `smiles_train.txt` is split into the respective molecules. These are canonicalized and filtered for validity. Then a start and end token are added to each molecule. For the dictionary, the SmilesPE atomwise tokenizer is used, which tokenizes the strings into atoms instead of distinct characters. Additionally, the start, stop and a padding character are added to the dictionary. Afterwards the SMILES strings are split into train and validation
data, encoded, and padded to the size of the largest molecule in the dataset.
The input for the model is the padded molecule from the first to second-last token. The label is the same molecule, this time from the second to the last token.

## 2. Model

The Molecule Generator Network architecture consists of an embedding layer, two LSTM layers with a hidden size of 1000, and a linear output layer. The embedding size corresponds to the size of the dictionary (100), and a dropout rate of 0.3 is applied. Sequences are not packed before entering the LSTM due to the computational overhead introduced by the packing process, which negates the benefits of saving computations on padded entries.

## 3. Training

The cross-entropy loss function is utilized for training, with the "ignore_index" parameter set to 0 to exclude padding characters from the loss computation. The model is trained for 20 epochs using a learning rate of `1e-4` and a batch size of `256`, resulting in a validation loss of `0.5`.

## 4. SMILES Generation

For SMILES generation, the molecules are generated character by character (atom by atom), initiated with a prime sequence. Generation proceeds until either the stop token or the maximum molecule length is reached. A softmax operation is applied to the model's output, and the next token in the sequence is sampled from the top 4 highest probabilities. A novel approach is taken for prime sequence selection: instead of using only the start token, a molecule from the training data is randomly selected, and its first three tokens (including the start token) are used as the prime sequence. This approach improves the FCD score significantly. The generated molecules are canonicalized and filtered for validity before being saved to a text file.
