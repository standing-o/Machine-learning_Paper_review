## Deep learning for identifying metastatic breast cancer
- Author: Wang et al.
- Journal: arXiv preprint arXiv
- Year: 2016
- Link: https://arxiv.org/abs/1606.05718



### Abstract
- Our team won both competitions in the grand challenge, obtaining an area under the receiver operating curve (AUC) of 0.925 for the task of whole slide image classification and a score of 0.7051 for the tumor localization task.

### Introduction
- Here, we present a deep learning-based approach for the identification of cancer metastases from whole slide images of breast sentinel lymph nodes. 
  - Our approach uses millions of training patches to train a deep convolutional neural network to make patch-level predictions to discriminate tumor-patches from normal-patches. 
  - We then aggregate the patch-level predictions to create tumor probability heatmaps and perform post-processing over these heatmaps to make predictions for the slide-based classification task and the tumor-localization task. 
  - Our system won both competitions at the Camelyon Grand Challenge 2016, with performance approaching human level accuracy. 


### Dataset
- The Camelyon16 dataset consists of a total of 400 whole slide images (WSIs) split into 270 for training and 130 for testing.
- The ground truth data for the training slides consists of a pathologist’s delineation of regions of metastatic cancer on WSIs of sentinel lymph nodes. 


### Method
#### Image Pre-processing
- We adopt a threshold based segmentation method to automatically detect the background region. 
  - In particular, we first transfer the original image from the RGB color space to the HSV color space, then the optimal threshold values in each channel are computed using the Otsu algorithm, and the final mask images are generated by combining the masks from H and S channels. 

#### Cancer metastasis detection framework
- During model training, the patch-based classification stage takes as input whole slide images and the ground
truth image annotation, indicating the locations of regions of each WSI containing metastatic cancer. 
- We randomly extract millions of small positive and negative patches from the set of training WSIs. 
  - If the small patch is located in a tumor region, it is a tumor / positive patch and labeled with 1, otherwise, it is a normal / negative patch and labeled with 0. 
  - Following selection of positive and negative training examples, we train a supervised classification model to discriminate between these two classes of patches, and we embed all the prediction results into a heatmap image. 
  - In the heatmap-based post-processing stage, we use the tumor probability heatmap to compute the slide-based evaluation and lesion-based evaluation scores for each WSI.

    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/745618db-12d6-4e33-80d8-2378cfe4457c' width=80%>

#### Patch-based Classification Stage
- During training, this stage uses as input 256x256 pixel patches from positive and negative regions of the WSIs and trains a classification model to discriminate between the positive and negative patches. 
- In our experiments, we evaluated a range of magnification levels, including 40×, 20× and 10×, and we obtained the best performance with 40× magnification. 
- After generating tumor-probability heatmaps using GoogLeNet across the entire training dataset, we noted that a significant proportion of errors were due to false positive classification from histologic mimics of cancer. 
  - To improve model performance on these regions, we extract additional training examples from these difficult negative regions and retrain the model with a training set enriched for these hard negative patches.

    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/347e56c7-2f5d-42ee-8f63-e19966af8ba8' width=80%>


#### Post-processing of tumor heatmaps to compute slide-based and lesion-based probabilities
- After completion of the patch-based classification stage, we generate a tumor probability heatmap for each WSI. 
  - On these heatmaps, each pixel contains a value between 0 and 1, indicating the probability that the pixel contains tumor.
- We now perform post-processing to compute slide-based and lesion-based scores for each heatmap.
- **Slide-based Classification**
  - For the slide-based classification task, the post-processing takes as input a heatmap for each WSI and produces as output a single probability of tumor for the entire WSI. 
  - Given a heatmap, we extract 28 geometrical and morphological features from each heatmap, including the percentage of tumor region over the whole tissue region, the area ratio between tumor region and the minimum surrounding convex region, the average prediction values, and the longest axis of the tumor region. 
  - We compute these features over tumor probability heatmaps across all training cases, and we build a random forest classifier to discriminate the WSIs with metastases from the negative WSIs. 


- **Lesion-based Detection**
  - For the lesion-based detection post-processing, we aim to identify all cancer lesions within each WSI with few false
positives.
  - To achieve this, we first train a deep model (D-I) using our initial training dataset that is enriched for tumor-adjacent negative regions. 
  - This model (D-II) produces fewer false positives than D-I but has reduced sensitivity. 
  - In our framework, we first threshold the heatmap produced from D-I at 0.90, which creates a binary heatmap. 
  - We then identify connected components within the tumor binary mask, and we use the central point as the tumor location for each connected component. 
  - To estimate the probability of tumor at each of these (x, y) locations, we take the average of the tumor probability predictions generated by D-I and D-II across each connected component. 

    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/0b482e73-0929-4b3c-9a57-5ade99f13675' width=50%>

