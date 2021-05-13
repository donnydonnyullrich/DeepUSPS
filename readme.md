## Disclaimer
This is the Code publication for the paper 
[1]: Nguyen, Dax et al: DeepUSPS: Deep Robust Unsupervised Saliency Prediction With Self-Supervision (NeurIPS 2019)
This work uses datasets, networks and pseudolabels supplied by traditional machine learning methods from past research in the CV community. 
In particular, credit goes to the following publications:

The Network used is designed by
[2]: Yu2017, Dilated Residual Networks
Original implementation: https://github.com/fyu/drn
Similar results for our method can be achieved with the DeepLab-ResNet pretrained on ImageNet (not implemented in this version). 

For training we use the MSRA-B dataset
[3]: WangDRFI2017, Salient Object Detection: A Discriminative Regional Feature Integration Approach (https://mmcheng.net/msra10k/)

and pseudolabels from the methods
[4]: Jiang2013, Saliency detection via absorbing markov chain
[5]: Zou2015, Hierarchy-associated rich features for salient object detection
[6]: Li2013, Saliency detection via dense and sparse reconstruction
[7]: Zhu2014, Saliency optimization from robust background detection



## Data and preparation
To install requirements:
pip install -r requirements.txt

The DeepUSPS directory contains 4 subfolders.

### 1. Code
The Code base.

### 2. Data
The Data folder contains the MSRA-B dataset:
- 01_img; with the images. (*.jpg)
- 02_gt; with the ground truth saliency masks. (*.png)
- 03_mc, 04_hs, 05_dsr, 06_rbd; with the saliency predictions of the handcrafted methods. (*.png)

### 3. Parameters
Directory with normalization parameters in file info.json, and split of dataset into training, validation and test data (train_names.txt, val_names.txt, test_names.txt).

### 4. Pretrained_Models
subfolder drn_pretraining: models pretrained on cityscapes.
subfolder DeepUSPS_trained_model: model trained with DeepUSPS pipeline (network: drn_d_105, size: 432 x 432), and baseline model trained on Ground Truth saliency (network: drn_d_105, size: 432 x 432)

Additionally, when training a model, a directory Doc/ will be created, in which detailed information about the training process is saved.




## Code Base

The main Code for training and testing is contained in **DeepUSPS.py**.
Additionally, there are several libraries that do not need to be modified for most purposes. They contain:

- DataClass.py:			The class definition used for handling the different types of data for the dataloader.
- Par_CRF.py:			Functions for applying deep crf methods to the predictions.
- data_transforms.py:		Modified transformations for input data.
- drn.py: 			The backbone of our tool, the powerfull drn network by [2].
- utily.py: 			Miscellaneous helper functions.




## Training (Unsupervised)

To start the training pipeline as proposed in [1], call the functions train_unsupervised in DeepUSPS.py.
As a quick start, you call the program via

python DeepUSPS.py train -r {1} -s {2} --arch {3} --batch-size {4}

{1}: root directory; insert the path to DeepUSPS folder
{2}: size; images are resized to size x size. Use 432 to reproduce our results.
{3}: network depth; use drn_d_105 for to reproduce our results with 105 layer network, and drn_d_22 for a shallow but faster network (in this case, learning rates might need to be adjusted in train_unsupervised() function)
{4}: batch size; choose 10 or 20 if enough gpu memory is available, else try 5, 4 or 2

additional arguments:

iter-size:		Gradient steps are performed every iter-size iteration. This way, the Code can make up for small batch-sizes. In train_unsupervies, this option is set sucht that effective batch size is 20 in phase 1 and 100 in phase 2.
pretrained:		If the pretrained model on cityscapes is not loaded automatically, download it manually and set the pretrained option with the appropriate path.




## Testing

To test the model on the MSRA-B dataset, run the command

python DeepUSPS.py test -r {1} -s {2} --arch {3} --batch-size {4} --resume {5}

{1}-{3}:	Use root directory, size and arch as specified in training.
{4}:		Use batch-size 1 if you want detailed results for individual images, else use larger batch size.
{5}:		Set path to the saved model from unsupervised training that you want to evaluate.

Results should be a little better than in validation phase while training, since at test time a deep CRF is applied by default. This is not done in validation, in order to reduce training time.
