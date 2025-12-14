This is a multi-class classification task to predict the tumor stage in patients using multiple-modalities such as clinical data, RNA expressions, and DNA methylation patterns.
It includes incremental fusion and abalation experiments to understand the importance and synergy among the modalities, VAE, and contrastive learning methods to learn their representations to train an XGBoost model.

## Data:
The Cancer Genome Atlas (TCGA) breast cancer patients data is used for this project. TCGA is one of the biggest cancer genomics datasets out there, with molecular profiles from thousands of tumors.
The modalities explored includes:
•	RNA seq data measuring expression levels of 60,664 genes basically which genes are turned on or off in the tumor
•	DNA methylation data from 14,164 sites on the genome these are chemical modifications that control gene activity without changing the DNA sequence
•	Clinical information including tumor stage (I, II, III, or IV)

## Methods:
  * ### Data pre-processing:
     Downloaded multi-gigabyte files from TCGA, matched patient samples across different data types using their barcode identifiers, and cleaned up the messy stage annotations patients sometimes had multiple diagnoses recorded, which were reconciled. After filtering for quality and matching samples across modalities, the data went from 1,098 clinical records and nearly 1,000 molecular profiles down to final 491 patients with complete data.
    To reduce the dimensionality, with 60,664 RNA features and 14,164 methylation features, including about 75,000 total measurements per patient which was way too many for our sample size of 491. Utilized PCA to reduce each modality down to 256 dimensions while keeping most of the variance. This gave a more manageable 512 features total when combined. The data is split as 80/20 for training and testing, using stratified sampling to maintain the stage proportions in both sets.
  * ### Baseline approaches:
    To understand which modality is more predictive, single modality baselines were built. The PCA reduced RNA data (256 dimensions) are used to train an XGBoost classifier. The same process is done with just
    methylation data. XGBoost is a gradient boosted tree model that works well on tabular data like this. For the multi omics baseline, the RNA and methylation PCA features are concatenated into one 512 dimensional vector to apply another round of PCA to get back down to 256 dimensions to train XGBoost. This tests whether just mashing the data together provides any benefit.
  * ### Variational autoencoder:
    The deep learning attempt used a VAE to learn a joint representation of both modalities. The idea was that forcing all the information through a tight bottleneck using 4 dimensions which would make the model
    learn a compressed representation that captures the essential relationships between RNA and methylation. The VAE has an encoder that compresses the 512 dimensional input down to 4 dimensional latent codes and
    a decoder that tries to reconstruct the original input from those codes. The VAE model is trained for 50 epochs, optimizing a loss function that combines reconstruction error with a regularization term that
    keeps the latent space well behaved. After training, the 4 dimensional latent codes are extracted for each patient and used those as features for XGBoost classification.The hypothesis was that these learned
    features might capture nonlinear relationships that simple PCA misses.
  * ### Contrastive learning:
    As the VAE didn't give much improvement, a different approach inspired by recent work in computer vision is used. Contrastive learning tries to align embeddings from different modalities by pulling together
    measurements from the same patient while pushing apart measurements from different patients. Separate neural network encoders for RNA and methylation are built, each mapping from 256 dimensions down to 64.
    The loss function encourages RNA embeddings and methylation embeddings from the same patient to be close together in the 64-dimensional space. The contrastive learning method is trained this for 100 epochs
    using InfoNCE loss with temperature scaling. After training, both the RNA and methylation embeddings are extracted for each patient and concatenated them which came to 64+64=128 dimensions total for downstream
    classification. The idea was that if there are meaningful cross modal relationships like certain RNA patterns that consistently co occur with certain methylation patterns contrastive learning should discover
    them.


