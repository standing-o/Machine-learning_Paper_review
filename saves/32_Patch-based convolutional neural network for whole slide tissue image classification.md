## Patch-based convolutional neural network for whole slide tissue image classification
- Author: Hou et al.
- Journal: Proceedings of the IEEE conference on computer vision and pattern recognition
- Year: 2016
- Link: https://openaccess.thecvf.com/content_cvpr_2016/papers/Hou_Patch-Based_Convolutional_Neural_CVPR_2016_paper.pdf

### Abstract
- WSI is currently computationally impossible. The differentiation of cancer subtypes is based on cellular-level visual features observed on image patch scale. Therefore, we argue that in this situation, training a patch-level classifier on image patches will perform better than or similar to an image-level classifier. 
- The challenge becomes how to intelligently combine patch-level classification results and model the fact that not all patches will be discriminative.
- We propose to train a decision fusion model to aggregate patch-level predictions given by patch-level CNNs, which to
the best of our knowledge has not been shown before. 
  - Furthermore, we formulate a novel `Expectation-Maximization (EM)` based method that automatically locates discriminative patches robustly by utilizing the spatial relationships of patches. 


### Introduction
-  Applying CNN directly for WSI classification has several drawbacks. First, extensive image downsampling is required by
which most of the discriminative details could be lost. Second, it is possible that a CNN might only learn from one of the multiple discriminative patterns in an image, resulting in data inefficiency. 
  - Discriminative information is encoded in high resolution image patches. Therefore, one solution is to train a CNN on high resolution image patches and predict the label of a WSI based on patch-level predictions.
- The ground truth labels of individual patches are unknown, as only the image-level ground truth label is given.
  - This complicates the classification problem. Because tumors may have a mixture of structures and texture properties, patch-level labels are not necessarily consistent with the image-level label. 
  - More importantly, when aggregating patch-level labels to an image-level label, simple decision fusion methods such as voting and max-pooling are not robust and do not match the decision process followed by pathologists.
- We propose using a patch-level CNN and training a decision fusion model as a two-level model.
    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/47f88e28-14eb-4dfa-97a3-ed876e2d5038' width=50%>

- The first-level (patch-level) model is an `Expectation Maximization (EM)` based method combined with CNN that outputs patch-level predictions. In particular, we assume that there is a hidden variable associated with each patch extracted from an image that indicates whether the patch is discriminative (i.e. the true hidden label of the patch is the same as the true label of the image).
- Initially, we consider all patches to be discriminative. We train a CNN model that outputs the cancer type probability of each input patch. We apply spatial smoothing to the resulting probability map and select only patches with higher probability values as discriminative patches. 
  - We iterate this process using the new set of discriminative patches in an EM fashion. In the second-level (image-level), histograms of patch-level predictions are input into an image-level multi-class logistic regression or SVM model that predicts the image-level labels.


### EM-based method with CNN
- We model a high resolution image as a bag and patches extracted from it as instances. We have a ground truth label for the whole image but not for the individual patches.
- We model whether an instance is discriminative or not as a hidden binary variable.
- $X = (X_1, ..., X_N)$: dataset containing N bags. Each bag $X_i = (X_{i,1},..., X_{i, N_i})$ consists of $N_i$ instances.
  - Assuming the bags are independent and i.i.d., the X and the hidden variables H are generated by the following generative model:
    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/8924cd23-abf1-4e05-9f9c-a791c19d6091' width=50%>
  - $H = (H_1, ..., H_N)$ is the hiddle variable and $H_{i,j}$ is the hidden variable that indicates whether instance $x_{i,j}$ is discriminative for label $y_i$ of bag $X_i$.

    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/6af8eeff-6ff9-4fee-9222-1b1c7234e387' width=50%>

    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/e314d12a-b98e-42be-8ae6-a14bff318905' width=50%>


### Discriminative patch selection
- Patches $x_{i,j}$ that have $P(H_{i,j} | X)$ larger than a threshold $T_{i,j}$ are considered discriminative and are selected to continue training CNN.
- It is reasonable to assume that $P(H_{i,j} | X)$ is correlated with $P(y_i | x_{i,j}; \theta)$ i.e., patches with lower $P(y_i | x_{i,j}; \theta)$ tends to have lower probability $x_{i,j}$ to be discriminative.
  - However, a hard-to-classify patch, or a patch close to the
decision boundary may have low $P(y_i | x_{i,j}; \theta)$ as well. These patches are informative and should not be rejected.
  - Therefore, to obtain a more robust $P(H_{i,j} | X)$, we apply the following two steps:
    - First, we train two CNNs on two different scales in parallel. $P(y_i | x_{i,j}; \theta)$ is the averaged prediction of the two CNNs.
    - Second, we simply denoise the probability map $P(y_i | x_{i,j}; \theta)$ of each image with a Gaussian kernel to compute $P(H_{i,j} | X)$.
    - This use of spatial relationships yields more robust discriminative patch identification.
- Choosing a thresholding scheme carefully yields significantly better performance than a simpler thresholding scheme.
  - We obtain the threshold Ti,j for $P(H_{i,j} | X)$ as follows: We note $S_i$ as the set of $P(H_{i,j} | X)$ values for all $x_{i,j}$ of the i-th image and $E_c$ as the set of $P(H_{i,j} | X)$ values for all $x_{i,j}$ of the c-th class. 
  - We introduce the image-level threshold $H_i$ as the $P_1$-th percentile of $S_i$ and the class-level threshold $R_i$ as the $P_2$-th percentile of $E_c$, where $P_1$ and $P_2$ are predefined. The threshold $T_{i,j}$ is defined as the minimum value between $H_i$ and $R_i$. 
  - There are two advantages of our method: 
    - By using the image-level threshold, there are at least $1 − P_1$ percent of patches that are considered discriminative for each image. 
    - By using the class-level threshold, the thresholds can be easily adapted to classes with different prior probabilities.


### Image-level decision fusion model
- We input all patch-level predictions into a multi-class logistic regression or SVM that outputs the image-level label. 
  - This decision level fusion method is more robust than max-pooling. 
  - Moreover, this method can be thought of as a Count-based Multiple Instance (CMI) learning method with two-level learning which is a more general MIL assumption than the Standard Multiple Instance (SMI) assumption.
  - For example, a WSI of the “mixed” glioma, Oligoastrocytoma should be recognized when two single glioma subtypes (Oligodendroglioma and Astrocytoma) are jointly present on the slide possibly on non-overlapping regions. 
- Because the patch-level model is never perfect and probably biased, an image-level decision fusion model may learn to correct the bias of patch-level decisions.
- Because it is unclear at this time whether strongly discriminative features for cancer subtypes exist at whole slide scale, we fuse patch-level predictions without the spatial relationship between patches. 
  - In particular, the class histogram of the patch-level predictions is the input to a linear multi-class logistic regression model or an SVM with Radial Basis Function (RBF) kernel. 
  - Because a WSI contains at least hundreds of patches, the class histogram is very robust to miss-classified patches. 
  - To generate the histogram, we sum up all of the class probabilities given by the patch-level CNN. 
  - Moreover, we concatenate histograms from four CNNs models: CNNs trained at two patch scales for two different numbers of iterations. We found in practice that using multiple histograms is robust.


### Experiments
- The dataset of WSIs used in the experiments part of the public Cancer Genome Atlas (TCGA) dataset. 
- The typical resolution of a WSI in this dataset is 100K by 50K pixels.

#### Patch extraction and segmentation
- To train the CNN model, we extract patches of size 500×500 from WSIs. To capture structures at multiple scales, we extract patches from 20X (0.5 microns per pixel) and 5X (2.0 microns per pixel) objective magnifications. 
- We discard patches with less than 30% tissue sections or have too much blood. 
  - We extract around 1000 valid patches per image per scale.
  - We select a random 400×400 sub-patch from each 500×500 patch. 

#### Experimental Setup
- Tested algorithms are:
  - 1. `CNN-Vote`: CNN followed by voting (average-pooling). We use all patches extracted from a WSI to train the patch-level CNN. There is no second-level model. Instead, the predictions of all patches vote for the final predicted label of a WSI.
  - 2. `CNN-SMI`: CNN followed by max-pooling. Same as `CNN-Vote` except the final predicted label of a WSI equals to the predicted label of the patch with maximum probability over all other patches and classes.
  - 3. `CNN-Fea-SVM`: We apply feature fusion instead of decision level fusion. In particular, we aggregate the outputs of the second fully connected layer of the CNN on all patches by 3-norm pooling. Then an SVM with RBF kernel predicts the image-level label.
  - 4. `EM-CNN-Vote/SMI`, `EM-CNN-Fea-SVM`: EM-based method with `CNN-Vote`, `CNN-SMI`, `CNN-Fea-SVM` respectively. We train the patch-level `EM-CNN` on discriminative patches identified by the E-step. Depending on the dataset, the discriminative threshold $P_1$ for each image ranges from 0.18 to 0.25; the discriminative threshold $P_2$ for each class ranges from 0.05 to 0.28. In each M-step, we train the CNN on all the discriminative patches for 2 epochs.
  - 5. `EM-Finetune-CNN-Vote/SMI`: Similar to `EM-CNNVote/SMI` except that instead of training a CNN from scratch, we fine-tune a pretrained 16-layer CNN model by training it on discriminative patches.
  - 6. `CNN-LR`: CNN followed by logistic regression. Same as `CNN-Vote` except that we train a second-level multi-class logistic regression to predict the image-level label. One tenth of the patches in each image is held out from the CNN to train the second-level multi-class logistic regression.
  - 7. `CNN-SVM`: CNN followed by SVM with RBF kernel instead of logistic regression.
  - 8. `EM-CNN-LR/SVM`: EM-based method with `CNN-LR` and `CNN-SVM` respectively.
  - 9. `EM-CNN-LR w/o spatial smoothing`: We do not apply Gaussian smoothing to estimate $P(H | X)$. Otherwise similar to `EM-CNN-LR`.
  - 10. `EM-Finetune-CNN-LR/SVM`: Similar to `EM-CNNLR/SVM` except that instead of training a CNN from scratch, we fine-tune a pretrained 16-layer CNN model by training it on discriminative patches.
  - 11. `SMI-CNN-SMI`: CNN with max-pooling at both discriminative patch identification and image-level prediction steps. For the patch-level CNN training, in each WSI only one patch with the highest confidence is considered discriminative.
  - 12. `NM-LBP`: We extract Nuclear Morphological features and rotation invariant Local Binary Patterns from all patches. We build a Bag-of-Words (BoW) feature using k-means followed by SVM with RBF kernel, as a non-CNN baseline.
  - 13. `Pretrained-CNN-Fea-SVM`: Similar to `CNN-Fea-SVM`. But instead of training a CNN, we use a pretrained 16-layer CNN model to extract features from patches. Then we select the top 500 features according to accuracy on the training set.
  - 14. `Pretrained-CNN-Bow-SVM`: We build a BoW model using k-means on features extracted by the pretrained CNN, followed by SVM.

- **Glioma classification results**
    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/0a1ef3d6-9291-4c39-a500-408c6944b662' width=50%>

- **NSCLC classification results**
    <img src='https://github.com/standing-o/Machine_Learning_Paper_Review/assets/57218700/d01e79e0-e31e-4df7-a0a1-9262871837b7' width=50%>

