---
layout: article
title: A new dataset for measuring the performance of blood vessel segmentation methods under distribution shifts
authors:
  - name: "Matheus Viana da Silva"
  - name: "Natália de Carvalho Santos"
  - name: "Julie Ouellette"
  - name: "Baptiste Lacoste"
  - name: "Cesar H. Comin"
    orcid: "0000-0003-1207-4982"

date: 2025-04-10                   
year: 2025                         
journal: Plos One
volume: 20
issue: 5
pages: e0322048
doi: 10.1371/journal.pone.0322048
abstract: |
  Creating a dataset for training supervised machine learning algorithms can be a demanding task. This is especially true for blood vessel segmentation since one or more specialists are usually required for image annotation, and creating ground truth labels for just a single image can take up to several hours. In addition, it is paramount that the annotated samples represent well the different conditions that might affect the imaged tissues as well as possible changes in the image acquisition process. This can only be achieved by considering samples that are typical in the dataset as well as atypical, or even outlier, samples. We introduce VessMAP, an annotated and highly heterogeneous blood vessel segmentation dataset acquired by carefully sampling relevant images from a large non-annotated dataset containing fluorescence microscopy images. Each image of the dataset contains metadata information regarding the contrast, amount of noise, density, and intensity variability of the vessels. Prototypical and atypical samples were carefully selected from the base dataset using the available metadata information, thus defining an assorted set of images that can be used for measuring the performance of segmentation algorithms on samples that are highly distinct from each other. We show that datasets traditionally used for developing new blood vessel segmentation algorithms tend to have low heterogeneity. Thus, neural networks trained on as few as four samples can generalize well to all other samples. In contrast, the training samples used for the VessMAP dataset can be critical to the generalization capability of a neural network. For instance, training on samples with good contrast leads to models with poor inference quality. Interestingly, while some training sets lead to Dice scores as low as 0.59, a careful selection of the training samples results in a Dice score of 0.85. Thus, the VessMAP dataset can be used for the development of new active learning methods for selecting relevant samples for manual annotation as well as for analyzing the robustness of segmentation models to distribution shifts of the data.
keywords:
  - Blood vessel dataset
  - Fluorescence microscopy
  - Distribution shift
mathjax: true         
bibtex: |
  @article{viana2025new,
    title={A new dataset for measuring the performance of blood vessel segmentation methods under distribution shifts},
    author={Viana da Silva, Matheus and de Carvalho Santos, Nat{\'a}lia and Ouellette, Julie and Lacoste, Baptiste and Comin, Cesar H},
    journal={Plos one},
    volume={20},
    number={5},
    pages={e0322048},
    year={2025},
    publisher={Public Library of Science San Francisco, CA USA}
  }
---

# Introduction

The performance of neural networks has dominantly been measured using metrics such as classification or segmentation accuracy, precision, recall, and the area under the ROC curve. However, recent studies have shown the dangers of only considering such globally-averaged metrics (Shit et al. 2020; Mosinska et al. 2018; <span class="nocase">da Silva et al.</span> 2022) that provide only an aggregated, summarized, view of the performance of machine learning algorithms on datasets with sometimes millions of images. Such an approach may hide important biases of the model (<span class="nocase">da Silva et al.</span> 2022). For instance, for medical images, a 95% accuracy is usually considered a good performance. But what about the remaining 5%? It is usually unrealistic to expect models to reach 100% accuracy, but the samples that are not correctly processed by a neural network may hide important biases of the model. These concerns led to the definition of new approaches and metrics that can aid the interpretation of black box models (Chaddad et al. 2023).

For medical image segmentation, the detection of relevant structures is usually only the first step of a more elaborate procedure for measuring relevant properties such as size (<span class="nocase">de P. Mendes et al.</span> 2021), regularity (Pérez-Beteta et al. 2018), length (<span class="nocase">Paetzold et al.</span> 2021; Freitas-Andrade et al. 2022), and curvature (Krestanova et al. 2020; Freitas-Andrade et al. 2022) of the imaged structures. Therefore, systematic segmentation mistakes might lead to undetected errors when characterizing samples for clinical diagnoses (<span class="nocase">Reinke et al.</span> 2021) and research purposes (Shit et al. 2020). An important cause of such systematic errors can be the presence of samples with characteristics that occur with low frequency in a dataset. This can happen due to additional, unexpected, noise during image acquisition, variations in tissue staining, image artifacts, or even the presence of structures that are anatomically different than what was expected. Assuming for illustration purposes that the data is normally distributed, a machine learning model having good performance around the peak of the distribution will tend to have good average accuracy measured for the whole dataset, even if it cannot correctly classify or segment images that are around the tail of the distribution (Gupta et al. 2019), which might be important for downstream analyses.

Segmenting the vasculature in tissue samples tends to be particularly challenging since the appearance of blood vessels can change significantly depending on tissue preparation and imaging protocols. In addition, in most cases, the global shape of the vasculature can be very different among the samples. We argue that blood vessel segmentation methodologies should have good performance, or even be directly optimized, on both prototypical and atypical samples. This focus can lead to models that are more robust to samples located in a sparsely populated region of the feature space of the dataset. In addition, it might also lead to models that generalize better to out-of-distribution samples as well as to new datasets. With these aspects in mind, we create a new dataset which we call the *Feature-Mapped Cortex Vasculature Dataset* (VessMAP). The dataset is designed to be as heterogeneous as possible by including samples having very different characteristics from each other. To this end, we use a simple and intuitive sampling methodology to select a subset of 100 images from a non-annotated base dataset containing 18279 image patches. The selected samples were then manually annotated with pixel-wise accuracy.

The dataset allows the creation of training and validation splits with images having different characteristics, such as contrast and blood vessel density. We show that different splits of the VessMAP dataset lead to very different training and validation results. As illustrated in Figure <a href="#fig:training_set_examples" data-reference-type="ref" data-reference="fig:training_set_examples">[fig:training_set_examples]</a>, the performance on the validation set can be dissimilar depending on the samples used for training a neural network. Thus, we expect the dataset to be useful for the development of new segmentation algorithms that are robust under distribution shifts of the data as well as for the validation of novel few-shot and active learning approaches.

The main contributions of this work are listed as follows:

- A new dataset, VessMAP, is made available to aid the development of blood vessel segmentation algorithms;

- It is shown that VessMAP has high variability compared to other popular datasets in the literature, leading to respective large variations of performance when training data is scarce;

- Specific splits of the dataset are provided for testing techniques that improve the generalizability of blood vessel segmentation algorithms.

# Related works

Table <a href="#tab:datasets" data-reference-type="ref" data-reference="tab:datasets">1</a> shows a summary of the main blood vessel datasets used in the literature as well as some recently published datasets. Most of the datasets have images from the retina. Few datasets are associated with microscopy images. More importantly, to our knowledge, none of the datasets were specifically designed to maximize the diversity of the samples. The diversity on some datasets tends to come as a proxy from the inclusion of healthy and abnormal tissue. For instance, samples in the DRIVE dataset contain diabetic retinopathy, which generates abnormal characteristics in blood vessels and image artifacts such as exudates that are not related to blood vessels. Still, most blood vessels tend to have a well-defined geometry and texture in all samples of the dataset. Thus, it becomes a simple task for a segmentation algorithm to generalize to new unseen samples from the same dataset. It is not surprising that many methods can reach an accuracy larger than 0.94 on the DRIVE dataset (Kovács and Fazekas 2022).

| Dataset | Anatomical Region | Imaging Technique |
|:---|:---|:---|
| DRIVE (Staal et al. 2004) |  |  |
| STARE (Hoover et al. 2000) |  |  |
| CHASEDB1 (Fraz et al. 2012a) |  |  |
| HRF (Odstrcilik et al. 2013) |  |  |
| INSPIRE-AVR (Niemeijer et al. 2011) |  |  |
| IMAGERET (Tomi Kauppi et al. 2007; T. Kauppi et al. 2007) |  |  |
| MESSIDOR (<span class="nocase">Decencière et al.</span> 2014) |  |  |
| VICAVR (*<span class="nocase">VICAVR dataset</span>*, n.d.) |  |  |
| ROC (Niemeijer et al. 2010) |  |  |
| DRIONS DB (Carmona et al. 2008) |  |  |
| DR HAGIS (Holm et al. 2017) |  |  |
| RET-TORT (Grisan et al. 2008) |  |  |
| WIDE (Estrada et al. 2015) |  |  |
|  |  | ultra-wide field-of-view fluorescein angiogram |
| IOSTAR (Zhang et al. 2016) |  |  |
| RC-SLO (Zhang et al. 2016) |  |  |
|  | aorta; cerebral, coronary, aortofemoral and pulmonary arteries |  |
| VESSEL12 (<span class="nocase">Rudyanto et al.</span> 2014) | human lung |  |
| 3D-IRCADb-01 (Soler et al. 2010) | human liver |  |
| ASOCA (Gharleghi et al. 2022) | coronary arteries | cardiac CTA |
| Vascular Synthesizer(Hamarneh and Jassi 2010) | synthetic vessels | \- |
| VesSAP (Todorov et al. 2020) |  |  |
| TubeMap (Kirst et al. 2020) |  |  |
| Di Diovanna et al. (Di Giovanna et al. 2018) |  |  |
| BvEM (Wan et al. 2024) |  | volume electron microscopy |
| OCTA (Glandorf et al. 2024) |  | optical coherence microscopy |
| DeepVess (Haft-Javaherian 2019) |  |  |
| MiniVess (Poon et al. 2023) |  |  |
|  | mouse brain, heart, and bladder vasculature |  |
| SMILE-UHURA (Chatterjee et al. 2024) |  | MRA |
| TopCoW (Yang et al. 2024) |  |  |
| DeepVesselNet (Tetteh et al. 2020) | human and rat brain |  |
| MSD8 (Antonelli et al. 2022) | human liver | computed tomography |
| HR-Kidney (Kuo et al. 2023) | mouse kidney | X-ray |
| HiP-CT (Yagis et al. 2024) | human kidney | computed tomography |

Summary of important blood vessel datasets on the literature. \*CTA: Computed tomography angiography. MRA: Magnetic resonance angiography. $`\mu`$CTA: Micro-computed tomography angiography. {#tab:datasets}

Regarding microscopy images, all datasets found by our survey include very few samples. Usually, very large 3D volumes are annotated in a semi-supervised fashion. They contain large amounts of vessels, but represent a single individual and image acquisition procedure. Therefore, most vessels have similar appearance and it becomes difficult to measure the generalization capability of segmentation methods. With these limitations in mind, we created a dataset that was specifically designed to include blood vessels having very different characteristics.

The creation of the dataset involved the application of a method for selecting relevant samples for annotation. A concept that is similar to the used methodology is the so-called *coreset* (Yu et al. 2024). The aim of a coreset is to select a subset of samples that can optimally represent the whole dataset. Many different methodologies and criteria were developed for defining relevant coresets (Zheng et al. 2019; Adhikari et al. 2021; Guo et al. 2022). Indeed, the subset defined by our methodology can be associated with a coreset, but in our case, the aim of the methodology and the approach used differs markedly from the usual definition of a coreset. The aim of our methodology is not focused on accurately representing the underlying distribution of the data or preserving the accuracy of a machine learning algorithm, but on providing a relevant dataset for training machine learning algorithms while avoiding the underrepresentation of atypical samples. In addition, many coreset methodologies use a surrogate neural network to estimate latent features or to estimate a degree of uncertainty about each sample, while our methodology is more general in the sense that any set of features obtained from the samples can be used. Furthermore, many related studies consider a similarity metric for selecting relevant samples (Zheng et al. 2019; Adhikari et al. 2021), which is a degenerate metric and, therefore cannot provide a full representation of the data distribution.

# Materials and methods

In the following, we describe the methodology used for creating the VessMAP dataset. The methodology is illustrated in Figure <a href="#fig:flowchart" data-reference-type="ref" data-reference="fig:flowchart">[fig:flowchart]</a> and can be divided into three steps: (a) acquisition of the base data from different experiments; (b) characterization of the base dataset according to important morphometry features; (c) selection of samples that uniformly covers the mapped feature space. The base non-annotated dataset used for selecting relevant samples for VessMAP is described in Section <a href="#sec:base_data" data-reference-type="ref" data-reference="sec:base_data">3.1</a>. The sampling methodology used for selecting appropriate samples from the base dataset is described in Section <a href="#sec:sampling" data-reference-type="ref" data-reference="sec:sampling">3.2</a>.

## Blood vessel microscopy base images

We start from a collection of 2637 confocal microscopy images of mouse brain vasculature. The images were acquired under different experimental conditions in different works published in the literature (Lacoste et al. 2014; Gouveia et al. 2017; Ouellette et al. 2020). Conditions include control animals, animals that have suffered a deletion of chromosome 16p11.12, animals that have experienced sense deprivation or sense hyperarousal, samples from stroke regions, and also from different stages of mouse development. The images have sizes from $`1376\times 1104`$ to $`2499\times 2005`$ pixels, totaling around 3.8GB of data.

The dataset is interesting because it has a considerable variety of characteristics of blood vessels. In addition, the images represent samples obtained from hundreds of different animals and experimental conditions. This makes it an excellent dataset for training machine learning algorithms for blood vessel segmentation. However, training supervised algorithms requires the manual annotation of the blood vessels.

After annotating a few samples, we estimated that each image in the dataset takes roughly 12 hours to fully annotate. Therefore, it is unfeasible to annotate the whole dataset, and a subset of samples needs to be selected. Our objective was to select a diverse set of samples containing both prototypical and atypical samples, so that it would be possible to create useful training and validation splits for quantifying the performance of segmentation algorithms under challenging distribution shifts between the splits. To this end, a sampling methodology was developed to select appropriate samples.

## Sampling methodology

Each image in the base dataset may include illumination inhomogeneities, changes in contrast, different levels of noise, as well as blood vessels having distinct characteristics (e.g., caliber, tortuosity, etc). Thus, from the original dataset, we generated a new set of images, each having a size of 256$`\times`$<!-- -->256 pixels. These smaller images were generated by extracting 256$`\times`$<!-- -->256 patches from the original images. As shown in Figure <a href="#fig:janelas" data-reference-type="ref" data-reference="fig:janelas">[fig:janelas]</a>, seven regions were extracted from each image. The seven regions were extracted in key areas of each image, with four windows in each of the corners of the image, a central window, and two windows at random positions. The latter two may overlap with the other windows. Windows that did not contain a satisfactory number of blood vessel segments were removed. The total size of the resulting dataset is 18279 images. This new dataset was used in the remainder of the sampling procedure.

The methodology developed to sample relevant images has three steps: 1) dataset mapping to a feature space, 2) generation of a discrete representation of the feature space, and 3) selection of points from the feature space representation. We explain each of these steps in the following sections.

### Dataset mapping

We represent the dataset as $`D = \{\delta_1, \delta_2, ..., \delta_n\}`$ where $`n`$ is the number of samples. Given a function $`f: \delta_i \to \vec{p}_i`$ that maps a sample $`\delta_i`$ to a vector $`\vec{p}_i`$ with dimension $`d`$, the dataset is mapped to a feature space as a $`n \times d`$ matrix, which we call $`D_{\textrm{mapped}}`$. The function $`f`$ represents a set of characteristics measured from the samples. Each line of matrix $`D_{\textrm{mapped}}`$ therefore represents the features of a sample $`f(\delta_i)`$.

Given that the images from our base dataset were used in previous works, each sample has a respective segmentation that was obtained using a semi-supervised methodology. This methodology is based on the adaptive thresholding of the original images, where the threshold was selected manually for each image. The full details of the segmentation procedure are described in (Freitas-Andrade et al. 2022). Using the semi-supervised segmentation, the following features were used to characterize the samples: blood vessel contrast, level of Gaussian noise, blood vessel density, and medial line heterogeneity.

The blood vessel contrast is related to the average difference in intensity between the vessels and the background of the image. The greater the contrast, the easier it is to detect the vessels. It can be measured using the original image of the vessels and the respective semi-supervised segmentation containing the pixels belonging to the vessels. The contrast was calculated as

``` math
\begin{equation}
    C = \frac{\bar{I}_v}{\bar{I}_f},
\end{equation}
```
where $`\bar{I}_v`$ and $`\bar{I}_f`$ are the mean intensities of, respectively, the pixels belonging to the blood vessels and the background of the image.

The signal-to-noise level of the images can be estimated in different ways. We investigated different definitions and used the method that was the most compatible with a visual inspection of the images. The method proposed in (Donoho and Johnstone 1994) was used. It assumes a noise with normal distribution and uses wavelets to identify the most likely standard deviation of the noise component. To prevent the method from capturing vessel variation, only the background of the image was used for the estimation.

Blood vessel density is defined as the total length of blood vessels in an image divided by the image area. To do this, we first applied a skeletonization algorithm to extract the medial lines of the vessels (Palàgyi and Kuba 1998). The total length of vessels was then calculated as the sum of the arc-lengths of all vessel segments.

The last metric, which we call medial line heterogeneity, measures the illumination changes in the vessel lumen. To calculate this metric, we first blurred the image using a Gaussian filter with unit standard deviation to remove extreme values. The medial line heterogeneity was then calculated as the standard deviation of the pixel values along the medial lines of this blurred image. The medial lines considered are the same ones used for the blood vessel density metric.

We observed that the medial line heterogeneity tended to be correlated with the average intensity of the blood vessels. In order to remove this dependency, the medial line heterogeneity, as well as the average intensity of the medial lines, were calculated for all images in the dataset. Then, a straight line fit $`h_m = a*m + b`$ was applied to the calculated values, where $`m`$ is the average intensity and $`h_m`$ is the expected medial line heterogeneity associated with $`m`$. Next, a normalized medial line heterogeneity was defined as $`\tilde{h} = h - h_m`$, where $`h`$ is the medial line heterogeneity calculated for an image.

These specific features were used because they can significantly impact the quality of the morphometry assessment of the cortex vasculature. For example, images with low contrast and/or high noise levels are expected to be more challenging to be accurately segmented. Conversely, images with a larger amount of blood vessels and high medial line heterogeneity provide intricate topology and texture to segmentation algorithms. Other features, such as blood vessel tortuosity, the density of branching points (bifurcations), and additional noise estimators, were considered. However, we disregarded strongly correlated features for our final data selection. The four remaining metrics mapped the base dataset to a 4-D feature space. As mentioned before, the dataset contains 18279 images. Hence, the whole dataset was mapped to a matrix $`D_{\textrm{mapped}}`$ having size $`18279\times 4`$.

### Feature space discretization

A regular grid was defined in the 4-D feature space, and each data point was mapped to the nearest point in this grid. Figure <a href="#fig:data_mapping" data-reference-type="ref" data-reference="fig:data_mapping">[fig:data_mapping]</a> illustrates this procedure. For creating the grid, it is useful to first normalize the values of $`D_{\textrm{mapped}}`$ to remove differences in the scale of the features. Thus, each feature was normalized to have zero mean and unit variance. Then, the discretization was done by defining a scale $`\nu`$ that sets the size of each grid cell, and calculating

``` math
\begin{equation}
    \displaystyle D_{\textrm{grid}} = \left\lfloor\frac{D_{\textrm{mapped}}}{\nu}\right\rfloor,
\end{equation}
```
where $`\lfloor . \rfloor`$ represents the floor function. As shown in Figure <a href="#fig:data_mapping" data-reference-type="ref" data-reference="fig:data_mapping">[fig:data_mapping]</a>(c), this operation ensures that each value of $`D_{\textrm{grid}}`$ lies within a regular grid. Note that, as a consequence of undersampling, we expect multiple data points to fall in the same grid position; this is one of the key properties of the method that will allow a uniform sampling of the data. A value of $`\nu=10`$ was used since we observed that it provided a good balance between grid sparsity and variability.

After the feature space discretization, we generated a sparse set of points representing an estimation of the possible values that can be obtained in the feature space. We call this set the *sampling set* of the feature space. This procedure works as follows. A 4-dimensional discrete hypersphere $`S`$ with radius $`r`$ (in grid units) centered on each data point is defined. This hypersphere is translated to each data point position. The union of the calculated hypersphere positions of all points defines the sampling set $`D_{\textrm{sset}}`$. The general appearance of $`D_{\textrm{sset}}`$ is depicted by the blue points of Figure <a href="#fig:drawing" data-reference-type="ref" data-reference="fig:drawing">[fig:drawing]</a>. The hypersphere radius used for creating the VessMAP dataset was $`r=4`$.

### Uniform selection of points

The final step of the method is to select the samples to be manually annotated. The samples are selected by first drawing a set of points from the sampling set $`D_{\textrm{sset}}`$. As illustrated in Figure <a href="#fig:drawing" data-reference-type="ref" data-reference="fig:drawing">[fig:drawing]</a>, we draw from $`D_{\textrm{sset}}`$ $`k`$ points with uniform probability (green dots in Figure <a href="#fig:drawing" data-reference-type="ref" data-reference="fig:drawing">[fig:drawing]</a>). For each point drawn, the closest data sample is identified using the Euclidean distance. If the same data sample is obtained more than once, a new point is drawn from $`D_{\textrm{sset}}`$ until $`k`$ unique data samples are obtained. The final set of data samples (orange stars in Figure <a href="#fig:drawing" data-reference-type="ref" data-reference="fig:drawing">[fig:drawing]</a>) is represented as $`D_{\textrm{sampled}}`$.

A uniform sampling of $`D_{\textrm{sset}}`$ allows the selection of prototypical and atypical samples from the dataset with equal probability. Nevertheless, a single realization of the sampling may lead to distortions, such as the selection of many samples at similar regions of the space or the creation of large regions with no samples selected. This is due to random fluctuations in the sampling process. To amend this, we define a metric called Farthest Unselected Point (FUS) that punishes sampled subsets with large gaps between the selected points.

Let $`D_{\textrm{sampled}}`$ be the set of sampled data points from $`D_{\textrm{grid}}`$, and $`\neg D_{\textrm{sampled}}`$ the set of points from $`D_{\textrm{grid}}`$ that were not selected in the sampled subset. For each data point in $`\neg D_{\textrm{sampled}}`$, the Euclidean distance to the closest point in $`D_{\textrm{sampled}}`$ is obtained. The FUS metric is defined as the largest calculated distance among all points in $`\neg D_{\textrm{sampled}}`$. Sampled subsets leading to low values of the FUS metric should be preferred since it avoids the creation of large regions of the feature space with no samples. In our experiments, we found that minimizing FUS for 1000 different subsets covered a good amount of subset possibilities.

We decided to select $`k = 100`$ images for annotation. Also, to avoid data leakage, an additional restriction that prevented the selection of samples from the same image was used.

# Results

## Dataset heterogeneity

The sampling approach used to generate the VessMAP dataset should lead to a heterogeneous set of samples. It is difficult to properly measure the heterogeneity of the dataset because it would involve the estimation of the probability density function of the original data, which is not a trivial task and can be influenced by the choice of parameter values. However, it is clear that the method should naturally lead to a uniform selection of the samples. This is so because the set $`D_{\textrm{grid}}`$ (defined in Section <a href="#sec:feature_space_discretization" data-reference-type="ref" data-reference="sec:feature_space_discretization">3.2.2</a>) represents an estimation of the domain of the probability density function of the data, and this domain is sampled uniformly.

One approach to illustrate the characteristics of the sampled images is displayed in Figure <a href="#fig:metrics_histogram" data-reference-type="ref" data-reference="fig:metrics_histogram">[fig:metrics_histogram]</a>, which shows histograms of the four considered features for both the full dataset and the sampled subset. The histograms of individual features are not expected to be uniform since they represent a projection of the original data into one dimension. Still, it can be seen that the histograms of the sampled set tend to represent a slightly flattened version of the histograms of the original data, indicating that a larger priority is being given to atypical samples when compared to the original distribution.

A more robust way of visually checking the sampled subset is to visualize the data using Principal Component Analysis (PCA). Using PCA, the original 4-D data can be projected into 2-d with optimal preservation of the variance (Gewers et al. 2021). Figure <a href="#fig:pca" data-reference-type="ref" data-reference="fig:pca">[fig:pca]</a> shows the PCA projection of the data. The four plots included in the figure represent the same projection, but the points are colored according to the different features used to characterize the images. The selected samples are shown in red. It can be noticed that the sampling methodology selects a subset of images that uniformly covers the distribution of the data. Furthermore, as also suggested by the histograms in Figure <a href="#fig:metrics_histogram" data-reference-type="ref" data-reference="fig:metrics_histogram">[fig:metrics_histogram]</a>, the sampling was capable of covering the full range of values of every considered feature.

The subset of images selected by the method (the VessMAP dataset) is shown in Figure <a href="#fig:sampled_images" data-reference-type="ref" data-reference="fig:sampled_images">[fig:sampled_images]</a>. The subset indeed contains a heterogeneous set of images covering many different values of the considered features (e.g., low contrast, high vessel density, etc). For instance, some of the samples in the dataset come from animals who suffered hemorrhagic strokes. These samples are very different from the typical samples contained in the base dataset, and they would be largely underrepresented if a sampling following the data distribution was performed.

We manually labeled each of the 100 images and made the dataset publicly available (Silva et al. 2023). To account for inter-annotator variability, 20 samples were labeled by two annotators. The Dice similarity score between the two annotators is 0.8780. We identified that most disagreement between annotators lies in delineating the vessel borders, resulting in mildly different blood vessel calibers. With that in mind, we also calculated the centerline Dice (clDice) (Shit et al. 2020) between both annotations, which provides a metric of how well the annotators agreed about the topology of the blood vessels. A clDice of 0.9556 was obtained, indicating a good agreement between annotators regarding the preservation of continuities and bifurcations. As a comparison, the annotations of the DRIVE dataset’s (one of the most used blood vessel segmentation datasets) test set have a Dice similarity of 0.7881 and a clDice of 0.7634.

The VessMAP repository includes manually annotated binary labels, their skeletons (calculated by the Palágyi-Kuba algorithm (Palàgyi and Kuba 1998)), and the metrics for each sample (as described in Section <a href="#sec:dataset_mapping" data-reference-type="ref" data-reference="sec:dataset_mapping">3.2.1</a>) – which were calculated using the manual annotations. We verified that the metrics calculated from the manual annotations have a strong correlation with the metrics calculated using the labels obtained from the semi-supervised segmentation algorithm. This evidences the quality of the algorithm in providing useful metrics to map the dataset into a feature space. We expect the VessMAP dataset to be useful for future studies regarding the influence of image and tissue characteristics on the generalization capability of segmentation algorithms.

## Neural network performance on VessMAP splits

Many current methods for semantic segmentation of biological images involve neural networks (Xu et al. 2024). Convolutional Neural Networks (CNNs) have been successfully used for segmenting biological structures for many years, especially after the adoption of encoder/decoder architectures, such as the original U-Net (Ronneberger et al. 2015) and its variants (Isensee et al. 2021; Galdran et al. 2022b; Fhima et al. 2024). Recently, the emergence of the Transformer (Vaswani et al. 2017) architecture allowed remarkable performance in tasks such as the development of Large Language Models (**OpenAI2024?**; <span class="nocase">Grattafiori et al.</span> 2024), speech processing (Radford et al. 2022), multimodal learning (Zhang et al. 2024), and drug discovery (Raza et al. 2023). Regarding image processing, Transformers don’t make assumptions about the relationship between the pixels of an image. This lack of inductive bias makes it harder to train a model from scratch in scenarios of scarce data, and it is usually necessary to pre-train a Vision Transformer (ViT) (Dosovitskiy et al. 2021) in large datasets such as ImageNet (Deng et al. 2009). Since we aim to evaluate the VessMAP performance using small training sets, we chose to use CNNs, which tend to perform better than ViTs for medical image segmentation with limited data (Shamshad et al. 2023).

To evaluate the potential of VessMAP to generate data splits that are challenging for neural networks, we generated eight different splits based on the features used for creating the dataset: blood vessel density, contrast, medial line heterogeneity, and noise estimation. For each feature, we selected 20 of the samples with the lowest and highest values and trained a segmentation CNN using two configurations: (i) training with samples that have the lowest feature values –lowest split– and evaluating with samples that have the highest feature values –highest split–, and (ii) training with the highest split and evaluating with the lowest split. We chose to use 20 images because it is a similar number to common split sizes used for well-known blood vessel datasets, such as DRIVE (Staal et al. 2004), STARE (Hoover et al. 2000), and CHASEDB1 (Fraz et al. 2012b). The idea behind this experiment is to test whether we can use VessMAP to generate splits that challenge the generalization capability of CNNs.

For this experiment, we used the CNN architecture illustrated in Figure <a href="#fig:resunetv2" data-reference-type="ref" data-reference="fig:resunetv2">[fig:resunetv2]</a>. This architecture encodes the input data through a series of residual blocks (He et al. 2016), concatenates the resulted feature vector with the activations from the first convolution operation (similar to a U-Net (Ronneberger et al. 2015)), and decodes the feature vector with a single residual block. For each training/evaluation split, we trained the network for 1000 epochs. The training was carried out using the Cross-Entropy as the loss function, the Adam optimizer (Kingma 2014), and a polynomial learning rate scheduler (power = 0.9) – which decays the initial learning rate (0.01) almost linearly.

Figure <a href="#fig:splits" data-reference-type="ref" data-reference="fig:splits">[fig:splits]</a> presents the loss curves of the eight training setups (two split configurations for each metric). We evaluate the distance between the training and validation loss curves, $`\delta`$, as a metric of how well the CNN generalized for out-of-distribution data. Only the first 200 epochs are plotted because $`\delta`$ did not change significantly during the remaining epochs. We define $`\delta`$ as the difference between the training loss and the validation loss at a specific epoch. It is also worth noting that the loss curves were smoothed using an exponentially weighted moving average in order to reduce the natural variance of the loss values and obtain a more precise $`\delta`$ value. Here, we calculate $`\delta`$ at epoch 150.

For the splits using the contrast feature, when training with samples having low contrast (Figure <a href="#fig:splits" data-reference-type="ref" data-reference="fig:splits">[fig:splits]</a>(b)), the network generalizes well for new data having high contrast. This behavior can be attributed to the fact that high-contrast images are less challenging and, if the network learns how to properly segment low-contrast images, it tends to handle well high-contrast images. The opposite behavior occurs when we invert the training and validation sets. When training with high-contrast images (Figure <a href="#fig:splits" data-reference-type="ref" data-reference="fig:splits">[fig:splits]</a>(a)), the CNN could not generalize towards low-contrast data, yielding a negative $`\delta`$. The same behavior can be observed for the blood vessel density splits. Notice that a negative $`\delta`$ indicates that using specifically low-density samples as the training set yields low generalization towards more dense images. The exact opposite happens when training with denser samples. It can also be noticed that the training splits using the highest noise and medial line heterogeneity values resulted in similar training and validation loss curves. This indicates that, although these splits are unbalanced regarding feature values, the samples are still diverse enough to allow good generalization. The results of the experiments with different splits are summarized in Table <a href="#tab:deltas" data-reference-type="ref" data-reference="tab:deltas">2</a>.

| Experiment         | Train Loss | Valid Loss | $`\delta`$  |
|:-------------------|:-----------|:-----------|:------------|
| **Contrast (h)**   | **0.2853** | **0.5359** | **-0.2507** |
| Contrast (l)       | 0.3831     | 0.1661     | 0.2107      |
| Density (h)        | 0.3652     | 0.2309     | 0.1343      |
| **Density (l)**    | **0.2610** | **0.5575** | **-0.2966** |
| Noise (h)          | 0.2717     | 0.2535     | 0.0182      |
| Noise (l)          | 0.3710     | 0.2243     | 0.1467      |
| **Skel. het. (h)** | **0.2934** | **0.2992** | **-0.0057** |
| Skel het. (l)      | 0.3696     | 0.2514     | 0.1182      |

$`\delta`$ values for the experiments with different splits generated using the VessMAP metadata. Each experiment name depicts the feature used to split the dataset, followed by (h) if the training set had the largest values of the feature or (l) if the training set had the lowest values. Negative $`\delta`$ values suggest poor generalization and were marked as bold. \*Skel. het: skeleton heterogeneity. {#tab:deltas}

Considering that the VessMAP images are diverse, another approach for generating challenging training and validation splits is to select samples that are far apart in the feature space. To do so, it is first necessary to identify a distance threshold above which the training and validation sets can be considered to be adequately separated in the feature space. We calculated this threshold by generating 10000 random splits of 20 training and validation images and obtaining the smallest Euclidean distance between all pairs of points of the two sets for each split. Then, we analyzed the histogram of the calculated distances and considered that two sets are far apart if their distance is larger than a threshold of $`t=0.7`$, which corresponded to approximately $`2.4\%`$ of the randomly drawn splits.

One of the identified splits containing highly distinct samples is highlighted in blue and red in Figure <a href="#fig:sampled_images" data-reference-type="ref" data-reference="fig:sampled_images">[fig:sampled_images]</a>. By using the same CNN and hyperparameters as the previous experiments, an average validation Dice score (Dice 1945) of $`0.824\pm0.008`$ was obtained for 100 training runs using the identified split. When the training and validation sets were swapped, a Dice score of $`0.892\pm0.006`$ was obtained. This result is in agreement with our previous experiment depicted in Figure <a href="#fig:splits" data-reference-type="ref" data-reference="fig:splits">[fig:splits]</a>, as the images of the training set present high contrast, low density, and low medial line heterogeneity.

For reference, we ran a similar experiment on the retinography images from the DRIVE, STARE, and CHASEDB1 datasets. For the DRIVE dataset, a Dice of $`0.803\pm0.002`$ was obtained using the official split of the dataset, and a Dice of $`0.791\pm0.003`$ was obtained when the training and validation sets were swapped. Since the STARE dataset does not have an official training and validation split, we trained the network for 100 randomly drawn splits and calculated the average performance difference between each split and its swapped counterpart. An average Dice difference of $`0.029\pm0.024`$ was obtained, with a maximum difference of 0.12. This same approach was applied to the CHASEDB1 dataset, where an average Dice difference of $`0.01\pm0.008`$ was obtained, with a maximum difference of 0.03. Note that the CHASEDB1 and DRIVE datasets got similar results regarding the performance difference between splits. The difference in Dice values obtained for the splits of the VessMAP dataset, $`0.892-0.824=0.068`$, was significantly larger than the maximum values obtained for the CHASEDB1 and DRIVE datasets.

Interestingly, our experiments show that the STARE dataset contains the split with the largest performance difference among all datasets. Indeed, some samples of the STARE dataset have very distinct appearances when compared to the typical characteristics of the dataset. Thus, in addition to VessMAP, STARE also seems to be a suitable dataset for evaluating network generalizability on blood vessel segmentation tasks. Nevertheless, the higher number of images in VessMAP compared to STARE and the feature metadata enables the definition of training setups with a greater number of training/validation splits.

It is worth mentioning that the Dice values obtained in our experiments with the fundus images are slightly lower than the state-of-the-art results for these datasets (Cervantes et al. 2023). This is mainly because no preprocessing and data augmentation were applied in order to match the training setup applied to VessMAP.

The heterogeneity of the VessMAP dataset is particularly useful for developing robust segmentation models when the training data is scarce. To show this, we ran a series of experiments using only four images for the training set. This replicates situations where, for instance, an active learning method suggests a small set of images for annotation, or on interactive segmentation scenarios where only a small set of blood vessels might be annotated. A neural network was trained on 4 randomly selected samples from the VessMAP dataset and the performance was measured on the remaining 96 samples. The same process was repeated 100 times using different sets of samples. To show that the overall results of our analyses are not dependent on a specific network architecture or training parameters, we replicated the same model and training approach used in (Galdran et al. 2022a). Specifically, the $`\phi_{3,8}`$ U-Net model containing 6 convolution layers in the encoder was used. The training protocol was also replicated with the exception of the cyclical learning rate scheduler, which was replaced by a polynomial scheduler. The batch size was also changed from 4 to 2 since the training set has only four samples.

For each image of the dataset, we measured the Dice scores obtained for trainings runs in which the image was not included in the training set. The result for all images is shown in Figure <a href="#fig:boxplots" data-reference-type="ref" data-reference="fig:boxplots">[fig:boxplots]</a>(a). It is clear that, for most images, the samples used for training the network have a large influence on the quality of the segmentation. The training set can lead to either very good segmentations or to segmentations that are of very poor quality.

For comparison, we repeated the same experiments for the DRIVE, STARE, and CHASEDB1 datasets. The results are shown in Figures <a href="#fig:boxplots" data-reference-type="ref" data-reference="fig:boxplots">[fig:boxplots]</a>(b)-(d). The variation observed for these datasets is much smaller compared to the VessMAP dataset. That is, four training samples are usually enough to obtain good and robust performance on the remaining samples. Thus, methods developed to work on scarce data annotation regimes might trivially result in good, low-biased performance when tested on these datasets. The same trend was observed for the area under the ROC curve (AUC) and average precision performance metrics (Figures <a href="#fig:boxplots_roc_auc" data-reference-type="ref" data-reference="fig:boxplots_roc_auc">[fig:boxplots_roc_auc]</a> and <a href="#fig:boxplots_avg_precision" data-reference-type="ref" data-reference="fig:boxplots_avg_precision">[fig:boxplots_avg_precision]</a> of the supplementary material).

To quantify the performance variation observed, the difference between the highest and lowest Dice score obtained for each sample was calculated. The average difference for all samples was then calculated for each dataset. The values are shown in Table <a href="#tab:dice_diff" data-reference-type="ref" data-reference="tab:dice_diff">3</a> and confirm the high performance variations observed for VessMAP.

| Dataset  | Average Dice difference | Std. dev. Dice difference |
|:---------|:------------------------|:--------------------------|
| VessMAP  | 0.55                    | 0.19                      |
| DRIVE    | 0.13                    | 0.13                      |
| STARE    | 0.29                    | 0.18                      |
| CHASEDB1 | 0.11                    | 0.06                      |

Influence of the training set on the generalizability of a model. For each dataset, models were trained on 100 different training sets containing 4 images each and were validated on the remaining images. The difference between the maximum and minimum Dice scores obtained for each sample across all runs was calculated and averaged over all samples. The standard deviation of the differences is also shown to provide a reference regarding the degree of variation observed among samples. {#tab:dice_diff}

In Figure <a href="#fig:predictions" data-reference-type="ref" data-reference="fig:predictions">[fig:predictions]</a> we show example segmentations obtained for the sample having the median standard deviation of Dice scores among all runs, that is, a sample with a typical variation of Dice scores observed in the dataset. The training set can lead to many missing blood vessels, to an oversegmentation of the vessels, to the presence of spurious holes as well as to discontinuities on the vessels. For many other samples, we also observed a large number of false positives.

With these analyses, we suggest two applications of the VessMAP metadata. First, one can generate splits that challenge the generalization capacity of a neural network, yielding negative $`\delta`$ during training. This kind of split can be used to test or develop new approaches to handle datasets having very distinct samples. In a similar fashion, splits that have positive $`\delta`$ can be used for developing new active learning methods, where it is useful to identify challenging samples for training networks so as to obtain low validation loss. In both situations, an ideal model should converge the training and validation loss curves, resulting in $`\delta \approx 0`$.

To aid the development of such methods, we provide official splits of the dataset containing training sets that lead to vastly distinct inference performances. The splits are shown in Table <a href="#tab:splits" data-reference-type="ref" data-reference="tab:splits">4</a>. For calculating the performance of each split, each of the 100 training runs was repeated 5 times using different seeds for the random number generator used during training.

|         | Training set              | Dice score         |
|:--------|:--------------------------|:-------------------|
| Split 1 | 4404, 11828, 16295, 7344  | $`0.846\pm 0.007`$ |
| Split 2 | 12943, 8493, 12618, 9284  | $`0.800\pm 0.014`$ |
| Split 3 | 7083, 6887, 14778, 2287   | $`0.752\pm 0.025`$ |
| Split 4 | 12877, 15577, 12960, 9593 | $`0.702\pm 0.024`$ |
| Split 5 | 8284, 9284, 11411, 9452   | $`0.653\pm 0.008`$ |
| Split 6 | 9710, 2643, 11111, 8196   | $`0.589\pm 0.073`$ |

Relevant training splits of the VessMAP dataset. Each row shows the average Dice score obtained on the remaining 96 samples when training a neural network using the four images indicated in the training set column. The standard deviation obtained across five repetitions of the training runs is also shown. {#tab:splits}

# Conclusion

Annotating appropriate images from a larger dataset for training machine learning algorithms is an important task. This is because the usual approach is to use as many images as possible. While this approach is relevant for general classification problems, for medical image segmentation, where image annotation can be very costly, the images used must be carefully selected in order to ensure good coverage of different tissue appearances and imaging variations. In addition, it is important that the annotated images do not lead to biases in downstream tasks related to tissue characterization. For instance, training segmentation algorithms mostly on prototypical images can lead to incorrect measurements on samples having unusual properties (e.g., very bright or very noisy).

We used an intuitive sampling methodology that evenly selects, as best as possible, both typical and atypical vascular image samples for creating VessMAP, a dataset containing a heterogeneous set of samples representing many possible variations of image noise and contrast as well as blood vessel density and intensity variance. One important characteristic of the dataset is that it provides an intuitive uniform grid in the feature space that can be used for further analyses. For example, one can study the accuracy of a segmentation model on different regions of the grid to identify regions where samples are not being correctly segmented. A robust algorithm should provide good segmentation no matter if a sample is too noisy, bright or dark, if it has low or high contrast, or any other variation on relevant image properties. The dataset is being made available together with the metadata containing the features used for creating the dataset.

We showed that different splits of the dataset can lead to largely distinct validation performances. The heterogeneity is particularly noticeable when training data is scarce. For many popular blood vessel datasets, the vasculature has similar characteristics throughout all samples. Thus, while they can be used for testing novel approaches for segmenting blood vessels, they are not ideal for quantifying the robustness of methods under small distribution shifts regarding sample characteristics and vessel geometry. Our analyses showed that VessMAP displays stronger appearance changes, with an average Dice score change of $`0.55`$, depending on the samples used for training. This result contrasts with the average Dice score difference of $`0.29`$ observed for the STARE dataset, the most heterogeneous dataset identified in the experiments after VessMAP.

One drawback of VessMAP is that the samples are relatively easy to segment. When training with more than 20 samples, the validation performance tends to be good and has little dependence on the training set. Thus, the usefulness of the dataset lies mostly in tasks with very limited training data. Another important consideration is that the features used to create the dataset are not necessarily related to the underlying conditions affecting the tissue samples (e.g., wild type, mutations, stroke, development stage) or to the acquisition process of the samples. Thus, obtaining good performance on the VessMAP dataset is important but not sufficient to conclude that a model is not biased on downstream tasks.

We expect that the dataset will be useful for studies regarding data distribution shifts as well as few-shot, interactive segmentation and active learning methods. We suggest two specific applications. Observing the official splits shown in Table <a href="#tab:splits" data-reference-type="ref" data-reference="tab:splits">4</a>, it is clear that among the 100 samples, training on samples 4404, 11828, 16295, and 7344 (split 1) led to robust models displaying good segmentation accuracy on the remaining samples (Dice score of 0.846). The same is not true for most of the other samples in the dataset. An active learning method should be able to automatically identify these four samples since they lead to a very low annotation effort to segment the whole dataset with good accuracy.

Another interesting application is the automatic identification of segmentation mistakes. A common scenario in real applications is the following. A new dataset is provided and needs to be segmented for downstream analyses. Since manually annotating blood vessel samples is time-consuming, only a fraction of the samples are manually annotated. A segmentation model is then trained on the annotated samples and applied to the remaining images. But how do we verify that the annotated samples were enough to provide good accuracy on downstream analyses for the remaining data? Looking back at Table <a href="#tab:splits" data-reference-type="ref" data-reference="tab:splits">4</a>, if the manually annotated samples are those of split 6, the performance of the model is known to be poor (Dice score of 0.589). Thus, one can develop additional heuristics to identify where the model is making mistakes. For instance, an interesting prospect is to analyze the topology of the vasculature and automatically identify missing segments, spurious branches and unrealistic connectivity patterns. The official splits of the vessMAP dataset allow a systematic comparison between methods developed by different research groups.

Interestingly, the splits in Table <a href="#tab:splits" data-reference-type="ref" data-reference="tab:splits">4</a> represent different degrees of difficult for such methods. Split 6, with a Dice score of 0.589, should lead to clearly unrealistic connectivity patterns. However, the difference between splits 1 and 2 is likely more subtle, and automatically identifying segmentation mistakes in split 2 that are not on split 1 should be more challenging.

The VessMAP dataset might also be used for testing the performance of more general methods that were not developed specifically for segmenting blood vessels. Many large-scale biomedical datasets have been created in recent years (Yang et al. 2023; <span class="nocase">Cheng et al.</span> 2023; <span class="nocase">Wang et al.</span> 2023; <span class="nocase">Wasserthal et al.</span> 2023; <span class="nocase">D’Antonoli et al.</span> 2024). Fluorescence microscopy samples are relatively uncommon in such datasets. Thus, the VessMAP dataset can be useful as an additional imaging modality for quantifying the performance of general methods.

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-Adhikari2021" class="csl-entry">

Adhikari, Bishwo, Esa Rahtu, and Heikki Huttunen. 2021. “Sample Selection for Efficient Image Annotation.” *2021 9th European Workshop on Visual Information Processing (EUVIP)*, 1–6. <https://doi.org/10.1109/EUVIP50544.2021.9484022>.

</div>

<div id="ref-Antonelli2022" class="csl-entry">

Antonelli, Michela, Annika Reinke, Spyridon Bakas, et al. 2022. “The Medical Segmentation Decathlon.” *Nature Communications* 13 (1): 4128. <https://doi.org/10.1038/s41467-022-30695-9>.

</div>

<div id="ref-CarmonaDRIONSDB" class="csl-entry">

Carmona, Enrique J., Mariano Rincón, Julián García-Feijoó, and José M. Martínez-de-la-Casa. 2008. “Identification of the Optic Nerve Head with Genetic Algorithms.” *Artificial Intelligence in Medicine* 43 (3): 243–59. https://doi.org/<https://doi.org/10.1016/j.artmed.2008.04.005>.

</div>

<div id="ref-Cervantes2023" class="csl-entry">

Cervantes, Jair, Jared Cervantes, Farid García-Lamont, Arturo Yee-Rendon, Josué Espejel Cabrera, and Laura Domínguez Jalili. 2023. “A Comprehensive Survey on Segmentation Techniques for Retinal Vessel Segmentation.” *Neurocomputing* 556: 126626. https://doi.org/<https://doi.org/10.1016/j.neucom.2023.126626>.

</div>

<div id="ref-Chaddad2023" class="csl-entry">

Chaddad, Ahmad, Jihao Peng, Jian Xu, and Ahmed Bouridane. 2023. “Survey of Explainable AI Techniques in Healthcare.” *Sensors* 23 (2). <https://doi.org/10.3390/s23020634>.

</div>

<div id="ref-Chatterjee2024" class="csl-entry">

Chatterjee, Soumick, Hendrik Mattern, Marc Dörner, et al. 2024. *SMILE-UHURA Challenge – Small Vessel Segmentation at Mesoscopic Scale from Ultra-High Resolution 7T Magnetic Resonance Angiograms*. <https://arxiv.org/abs/2411.09593>.

</div>

<div id="ref-cheng2023sam" class="csl-entry">

<span class="nocase">Cheng, Junlong, Jin Ye, Zhongying Deng, et al.</span> 2023. “Sam-Med2d.” *arXiv Preprint arXiv:2308.16184*.

</div>

<div id="ref-d2024totalsegmentator" class="csl-entry">

<span class="nocase">D’Antonoli, Tugba Akinci, Lucas K Berger, Ashraya K Indrakanti, et al.</span> 2024. “TotalSegmentator MRI: Sequence-Independent Segmentation of 59 Anatomical Structures in MR Images.” *arXiv Preprint arXiv:2405.19492*.

</div>

<div id="ref-DaSilva2022" class="csl-entry">

<span class="nocase">da Silva, Matheus V., Julie Ouellette, Baptiste Lacoste, and Cesar H. Comin</span>. 2022. “An Analysis of the Influence of Transfer Learning When Measuring the Tortuosity of Blood Vessels.” *Computer Methods and Programs in Biomedicine* 225: 107021. https://doi.org/<https://doi.org/10.1016/j.cmpb.2022.107021>.

</div>

<div id="ref-DePMendes2021" class="csl-entry">

<span class="nocase">de P. Mendes, Rodrigo, Xin Yuan, Elizabeth M. Genega, Xiaoyin Xu, Luciano da F. Costa, and Cesar H. Comin</span>. 2021. “Gland Context Networks: A Novel Approach for Improving Prostate Cancer Identification.” *Computerized Medical Imaging and Graphics* 94: 101999. https://doi.org/<https://doi.org/10.1016/j.compmedimag.2021.101999>.

</div>

<div id="ref-DecenciereMESSIDOR" class="csl-entry">

<span class="nocase">Decencière, Etienne, Xiwei Zhang, Guy Cazuguel, et al.</span> 2014. “FEEDBACK ON a PUBLICLY DISTRIBUTED IMAGE DATABASE: THE MESSIDOR DATABASE.” *Image Analysis and Stereology* 33 (3): 231–34. <https://doi.org/10.5566/ias.1155>.

</div>

<div id="ref-Deng2009" class="csl-entry">

Deng, J., W. Dong, R. Socher, L.-J. Li, K. Li, and L. Fei-Fei. 2009. “ImageNet: A Large-Scale Hierarchical Image Database.” *CVPR09*.

</div>

<div id="ref-DiGiovanna2018" class="csl-entry">

Di Giovanna, Antonino Paolo, Alessandro Tibo, Ludovico Silvestri, et al. 2018. “Whole-Brain Vasculature Reconstruction at the Single Capillary Level.” *Scientific Reports* 8 (1): 12573.

</div>

<div id="ref-Dice1945" class="csl-entry">

Dice, Lee R. 1945. “<span class="nocase">Measures of the Amount of Ecologic Association Between Species</span>.” *Ecology* 26 (3): 297–302. <https://doi.org/10.2307/1932409>.

</div>

<div id="ref-Donoho1994" class="csl-entry">

Donoho, David L, and Iain M Johnstone. 1994. “<span class="nocase">Ideal spatial adaptation by wavelet shrinkage</span>.” *Biometrika* 81 (3): 425–55. <https://doi.org/10.1093/biomet/81.3.425>.

</div>

<div id="ref-Dosovitskiy2021" class="csl-entry">

Dosovitskiy, Alexey, Lucas Beyer, Alexander Kolesnikov, et al. 2021. *An Image Is Worth 16x16 Words: Transformers for Image Recognition at Scale*. <https://arxiv.org/abs/2010.11929>.

</div>

<div id="ref-EstradaWIDE" class="csl-entry">

Estrada, Rolando, Michael J. Allingham, Priyatham S. Mettu, Scott W. Cousins, Carlo Tomasi, and Sina Farsiu. 2015. “Retinal Artery-Vein Classification via Topology Estimation.” *IEEE Transactions on Medical Imaging* 34 (12): 2518–34. <https://doi.org/10.1109/TMI.2015.2443117>.

</div>

<div id="ref-Fhima2024" class="csl-entry">

Fhima, Jonathan, Jan Van Eijgen, Marie-Isaline Billen Moulin-Romsée, et al. 2024. “LUNet: Deep Learning for the Segmentation of Arterioles and Venules in High Resolution Fundus Images.” *Physiological Measurement* 45 (5): 055002. <https://doi.org/10.1088/1361-6579/ad3d28>.

</div>

<div id="ref-FrazCHASEBD1" class="csl-entry">

Fraz, Muhammad Moazam, Paolo Remagnino, Andreas Hoppe, et al. 2012a. “An Ensemble Classification-Based Approach Applied to Retinal Blood Vessel Segmentation.” *IEEE Transactions on Biomedical Engineering* 59 (9): 2538–48. <https://doi.org/10.1109/TBME.2012.2205687>.

</div>

<div id="ref-Fraz2012" class="csl-entry">

Fraz, Muhammad Moazam, Paolo Remagnino, Andreas Hoppe, et al. 2012b. “An Ensemble Classification-Based Approach Applied to Retinal Blood Vessel Segmentation.” *IEEE Transactions on Biomedical Engineering* 59 (9): 2538–48. <https://doi.org/10.1109/TBME.2012.2205687>.

</div>

<div id="ref-FreitasAndrade2022" class="csl-entry">

Freitas-Andrade, Moises, Cesar H. Comin, Matheus Viana da Silva, Luciano F. Fontoura Da Costa, and Baptiste Lacoste. 2022. “<span class="nocase">Unbiased analysis of mouse brain endothelial networks from two- or three-dimensional fluorescence images</span>.” *Neurophotonics* 9 (3): 031916. <https://doi.org/10.1117/1.NPh.9.3.031916>.

</div>

<div id="ref-galdran2022state" class="csl-entry">

Galdran, Adrian, André Anjos, José Dolz, Hadi Chakor, Hervé Lombaert, and Ismail Ben Ayed. 2022a. “State-of-the-Art Retinal Vessel Segmentation with Minimalistic Models.” *Scientific Reports* 12 (1): 6174.

</div>

<div id="ref-Galdran2022" class="csl-entry">

Galdran, Adrian, André Anjos, José Dolz, Hadi Chakor, Hervé Lombaert, and Ismail Ben Ayed. 2022b. “State-of-the-Art Retinal Vessel Segmentation with Minimalistic Models.” *Scientific Reports* 12 (1): 6174.

</div>

<div id="ref-Gewers2021" class="csl-entry">

Gewers, Felipe L., Gustavo R. Ferreira, Henrique F. De Arruda, et al. 2021. “Principal Component Analysis: A Natural Approach to Data Exploration.” *ACM Comput. Surv.* (New York, NY, USA) 54 (4). <https://doi.org/10.1145/3447755>.

</div>

<div id="ref-GharleghiASOCA" class="csl-entry">

Gharleghi, Ramtin, Dona Adikari, Katy Ellenberger, et al. 2022. “Automated Segmentation of Normal and Diseased Coronary Arteries – the ASOCA Challenge.” *Computerized Medical Imaging and Graphics* 97: 102049. https://doi.org/<https://doi.org/10.1016/j.compmedimag.2022.102049>.

</div>

<div id="ref-Glandorf2024" class="csl-entry">

Glandorf, Lukas, Bastian Wittmann, Jeanne Droux, et al. 2024. “Bessel Beam Optical Coherence Microscopy Enables Multiscale Assessment of Cerebrovascular Network Morphology and Function.” *Light: Science & Applications* 13 (1): 307. <https://doi.org/10.1038/s41377-024-01649-1>.

</div>

<div id="ref-Gouveia2017" class="csl-entry">

Gouveia, Ayden, Matthew Seegobin, Timal S. Kannangara, et al. 2017. “The aPKC-CBP Pathway Regulates Post-Stroke Neurovascular Remodeling and Functional Recovery.” *Stem Cell Reports* 9 (6): 1735–44. https://doi.org/<https://doi.org/10.1016/j.stemcr.2017.10.021>.

</div>

<div id="ref-Meta2024" class="csl-entry">

<span class="nocase">Grattafiori, Aaron, Abhimanyu Dubey, Abhinav Jauhri, et al.</span> 2024. *The Llama 3 Herd of Models*. <https://arxiv.org/abs/2407.21783>.

</div>

<div id="ref-ForacchiaRETTORT" class="csl-entry">

Grisan, Enrico, Marco Foracchia, and Alfredo Ruggeri. 2008. “A Novel Method for the Automatic Grading of Retinal Vessel Tortuosity.” *IEEE Transactions on Medical Imaging* 27 (3): 310–19. <https://doi.org/10.1109/TMI.2007.904657>.

</div>

<div id="ref-Guo2022" class="csl-entry">

Guo, Chengcheng, Bo Zhao, and Yanbing Bai. 2022. “DeepCore: A Comprehensive Library for Coreset Selection in Deep Learning.” In *Database and Expert Systems Applications*, edited by Christine Strauss, Alfredo Cuzzocrea, Gabriele Kotsis, A. Min Tjoa, and Ismail Khalil. Springer International Publishing.

</div>

<div id="ref-Gupta2019" class="csl-entry">

Gupta, Agrim, Piotr Dollár, and Ross Girshick. 2019. *LVIS: A Dataset for Large Vocabulary Instance Segmentation*. <https://arxiv.org/abs/1908.03195>.

</div>

<div id="ref-HaftJavaherian2019" class="csl-entry">

Haft-Javaherian, Linjing AND Muse, Mohammad AND Fang. 2019. “Deep Convolutional Neural Networks for Segmenting 3D in Vivo Multiphoton Images of Vasculature in Alzheimer Disease Mouse Models.” *PLOS ONE* 14 (3): 1–21. <https://doi.org/10.1371/journal.pone.0213539>.

</div>

<div id="ref-HamarnehVascuSynth" class="csl-entry">

Hamarneh, Ghassan, and Preet Jassi. 2010. “VascuSynth: Simulating Vascular Trees for Generating Volumetric Image Data with Ground Truth Segmentation and Tree Analysis.” *Computerized Medical Imaging and Graphics* 34 (8): 605–16. <https://doi.org/10.1016/j.compmedimag.2010.06.002>.

</div>

<div id="ref-He2016" class="csl-entry">

He, Kaiming, Xiangyu Zhang, Shaoqing Ren, and Jian Sun. 2016. “Deep Residual Learning for Image Recognition.” *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*, 770–78.

</div>

<div id="ref-HolmDRHAGIS" class="csl-entry">

Holm, Sven, Greg Russell, Vincent Nourrit, and Niall McLoughlin. 2017. “<span class="nocase">DR HAGIS—a fundus image database for the automatic extraction of retinal surface vessels from diabetic patients</span>.” *Journal of Medical Imaging* 4 (1): 014503. <https://doi.org/10.1117/1.JMI.4.1.014503>.

</div>

<div id="ref-HooverSTARE" class="csl-entry">

Hoover, A. D., V. Kouznetsova, and M. Goldbaum. 2000. “Locating Blood Vessels in Retinal Images by Piecewise Threshold Probing of a Matched Filter Response.” *IEEE Transactions on Medical Imaging* 19 (3): 203–10. <https://doi.org/10.1109/42.845178>.

</div>

<div id="ref-Isensee2021" class="csl-entry">

Isensee, Fabian, Paul F Jaeger, Simon A A Kohl, Jens Petersen, and Klaus H Maier-Hein. 2021. “<span class="nocase">nnU-Net</span>: A Self-Configuring Method for Deep Learning-Based Biomedical Image Segmentation.” *Nature Methods* 18 (2): 203–11.

</div>

<div id="ref-KauppiDIARETDB1" class="csl-entry">

Kauppi, T., V. Kalesnykiene, J.-K. Kamarainen, et al. 2007. “The DIARETDB1 Diabetic Retinopathy Database and Evaluation Protocol.” *Proceedings of the British Machine Vision Conference*, 15.1–10.

</div>

<div id="ref-KauppiDIARETDB0" class="csl-entry">

Kauppi, Tomi, Valentina Kalesnykiene, Joni-Kristian Kämäräinen, et al. 2007. “DIARETDB 0 : Evaluation Database and Methodology for Diabetic Retinopathy Algorithms.” <https://api.semanticscholar.org/CorpusID:573081>.

</div>

<div id="ref-Kingma2014" class="csl-entry">

Kingma, Diederik P. 2014. “Adam: A Method for Stochastic Optimization.” *arXiv Preprint arXiv:1412.6980*.

</div>

<div id="ref-KirstTUBEMAP" class="csl-entry">

Kirst, Christoph, Sophie Skriabine, Alba Vieites-Prado, et al. 2020. “Mapping the Fine-Scale Organization and Plasticity of the Brain Vasculature.” *Cell* 180 (4): 780–795.e25. https://doi.org/<https://doi.org/10.1016/j.cell.2020.01.028>.

</div>

<div id="ref-Kovacs2022" class="csl-entry">

Kovács, György, and Attila Fazekas. 2022. “A New Baseline for Retinal Vessel Segmentation: Numerical Identification and Correction of Methodological Inconsistencies Affecting 100+ Papers.” *Medical Image Analysis* 75: 102300. https://doi.org/<https://doi.org/10.1016/j.media.2021.102300>.

</div>

<div id="ref-Krestanova2020" class="csl-entry">

Krestanova, Alice, Jan Kubicek, and Marek Penhaker. 2020. “Recent Techniques and Trends for Retinal Blood Vessel Extraction and Tortuosity Evaluation: A Comprehensive Review.” *Ieee Access* 8: 197787–816.

</div>

<div id="ref-Kuo2023" class="csl-entry">

Kuo, Willy, Diego Rossinelli, Georg Schulz, et al. 2023. “Terabyte-Scale Supervised 3D Training and Benchmarking Dataset of the Mouse Kidney.” *Scientific Data* 10 (1): 510. <https://doi.org/10.1038/s41597-023-02407-5>.

</div>

<div id="ref-Lacoste2014" class="csl-entry">

Lacoste, Baptiste, Cesar H. Comin, Ayal Ben-Zvi, et al. 2014. “Sensory-Related Neural Activity Regulates the Structure of Vascular Networks in the Cerebral Cortex.” *Neuron* 83 (5): 1117–30. https://doi.org/<https://doi.org/10.1016/j.neuron.2014.07.034>.

</div>

<div id="ref-Mosinska2018" class="csl-entry">

Mosinska, Agata, Pablo Marquez-Neila, Mateusz Koziński, and Pascal Fua. 2018. “Beyond the Pixel-Wise Loss for Topology-Aware Delineation.” *Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition*, 3136–45.

</div>

<div id="ref-NiemeijerROC" class="csl-entry">

Niemeijer, Meindert, Bram van Ginneken, Michael J. Cree, et al. 2010. “Retinopathy Online Challenge: Automatic Detection of Microaneurysms in Digital Color Fundus Photographs.” *IEEE Transactions on Medical Imaging* 29 (1): 185–95. <https://doi.org/10.1109/TMI.2009.2033909>.

</div>

<div id="ref-NiemeijerINSPIRE-AVR" class="csl-entry">

Niemeijer, Meindert, Xiayu Xu, Alina V. Dumitrescu, et al. 2011. “Automated Measurement of the Arteriolar-to-Venular Width Ratio in Digital Color Fundus Photographs.” *IEEE Transactions on Medical Imaging* 30 (11): 1941–50. <https://doi.org/10.1109/TMI.2011.2159619>.

</div>

<div id="ref-OdstrcilikHRF" class="csl-entry">

Odstrcilik, Jan, Radim Kolar, Attila Budai, et al. 2013. “Retinal Vessel Segmentation by Improved Matched Filtering: Evaluation on a New High-Resolution Fundus Image Database.” *IET Image Processing* 7 (4): 373–83. https://doi.org/<https://doi.org/10.1049/iet-ipr.2012.0455>.

</div>

<div id="ref-Ouellette2020" class="csl-entry">

Ouellette, Julie, Xavier Toussay, Cesar H. Comin, et al. 2020. “Vascular Contributions to 16p11.2 Deletion Autism Syndrome Modeled in Mice.” *Nature Neuroscience* 23 (9): 1090–101. <https://doi.org/10.1038/s41593-020-0663-1>.

</div>

<div id="ref-Paetzold2021" class="csl-entry">

<span class="nocase">Paetzold, Johannes C., Julian McGinnis, Suprosanna Shit, et al.</span> 2021. “Whole Brain Vessel Graphs: A Dataset and Benchmark for Graph Learning and Neuroscience.” *Thirty-Fifth Conference on Neural Information Processing Systems Datasets and Benchmarks Track (Round 2)*. <https://openreview.net/forum?id=jpwGODt2Av>.

</div>

<div id="ref-Palagyi1998" class="csl-entry">

Palàgyi, Kàlmàn, and Attila Kuba. 1998. “A 3D 6-Subiteration Thinning Algorithm for Extracting Medial Lines.” *Pattern Recognition Letters* 19 (7): 613–27. https://doi.org/<https://doi.org/10.1016/S0167-8655(98)00031-2>.

</div>

<div id="ref-Perez-Beteta2018" class="csl-entry">

Pérez-Beteta, Julián, David Molina-Garcı́a, José A. Ortiz-Alhambra, et al. 2018. “Tumor Surface Regularity at MR Imaging Predicts Survival and Response to Surgery in Patients with Glioblastoma.” *Radiology* 288 (1): 218–25. <https://doi.org/10.1148/radiol.2018171051>.

</div>

<div id="ref-Poon2023" class="csl-entry">

Poon, Charissa, Petteri Teikari, Muhammad Febrian Rachmadi, Henrik Skibbe, and Kullervo Hynynen. 2023. “A Dataset of Rodent Cerebrovasculature from in Vivo Multiphoton Fluorescence Microscopy Imaging.” *Scientific Data* 10 (1): 141. <https://doi.org/10.1038/s41597-023-02048-8>.

</div>

<div id="ref-Radford2022" class="csl-entry">

Radford, Alec, Jong Wook Kim, Tao Xu, Greg Brockman, Christine McLeavey, and Ilya Sutskever. 2022. *Robust Speech Recognition via Large-Scale Weak Supervision*. <https://arxiv.org/abs/2212.04356>.

</div>

<div id="ref-Raza2023" class="csl-entry">

Raza, Ali, Jamal Uddin, Abdullah Almuhaimeed, Shahid Akbar, Quan Zou, and Ashfaq Ahmad. 2023. “AIPs-SnTCN: Predicting Anti-Inflammatory Peptides Using fastText and Transformer Encoder-Based Hybrid Word Embedding with Self-Normalized Temporal Convolutional Networks.” *Journal of Chemical Information and Modeling* 63 (21): 6537–54. <https://doi.org/10.1021/acs.jcim.3c01563>.

</div>

<div id="ref-Reinke2021" class="csl-entry">

<span class="nocase">Reinke, Annika, Minu D. Tizabi, Carole H. Sudre, et al.</span> 2021. *Common Limitations of Image Processing Metrics: A Picture Story*. arXiv. <https://doi.org/10.48550/ARXIV.2104.05642>.

</div>

<div id="ref-Ronneberger2015" class="csl-entry">

Ronneberger, Olaf, Philipp Fischer, and Thomas Brox. 2015. “U-Net: Convolutional Networks for Biomedical Image Segmentation.” *Medical Image Computing and Computer-Assisted Intervention–MICCAI 2015: 18th International Conference, Munich, Germany, October 5-9, 2015, Proceedings, Part III 18*, 234–41.

</div>

<div id="ref-RudyantoVESSEL12" class="csl-entry">

<span class="nocase">Rudyanto, Rina D., Sjoerd Kerkstra, Eva M. van Rikxoort, et al.</span> 2014. “Comparing Algorithms for Automated Vessel Segmentation in Computed Tomography Scans of the Lung: The VESSEL12 Study.” *Medical Image Analysis* 18 (7): 1217–32. https://doi.org/<https://doi.org/10.1016/j.media.2014.07.003>.

</div>

<div id="ref-Shamshad2023" class="csl-entry">

Shamshad, Fahad, Salman Khan, Syed Waqas Zamir, et al. 2023. “Transformers in Medical Imaging: A Survey.” *Medical Image Analysis* 88: 102802. https://doi.org/<https://doi.org/10.1016/j.media.2023.102802>.

</div>

<div id="ref-Shit2020" class="csl-entry">

Shit, Suprosanna, Johannes C. Paetzold, Anjany Sekuboyina, et al. 2020. “clDice - a Topology-Preserving Loss Function for Tubular Structure Segmentation.” *CoRR* abs/2003.07311. <https://arxiv.org/abs/2003.07311>.

</div>

<div id="ref-Vessmap2023" class="csl-entry">

Silva, Matheus Viana da, Natália de Carvalho Santos, Baptiste Lacoste, and Cesar Henrique Comin. 2023. *VessMAP - Feature-Mapped Cortex Vasculature Dataset*. <a href="https://zenodo.org/records/10045265" class="uri">Https://zenodo.org/records/10045265</a>.

</div>

<div id="ref-SolerIRCADb" class="csl-entry">

Soler, Luc, Alexandre Hostettler, Vincent Agnus, et al. 2010. *3D Image Reconstruction for Comparison of Algorithm Database: A Patient Specific Anatomical and Medical Image Database*. <a href="https://www-sop.inria.fr/geometrica/events/wam/abstract-ircad.pdf" class="uri">Https://www-sop.inria.fr/geometrica/events/wam/abstract-ircad.pdf</a>.

</div>

<div id="ref-StaalDRIVE" class="csl-entry">

Staal, J., M. D. Abramoff, M. Niemeijer, M. A. Viergever, and B. van Ginneken. 2004. “Ridge-Based Vessel Segmentation in Color Images of the Retina.” *IEEE Transactions on Medical Imaging* 23 (4): 501–9. <https://doi.org/10.1109/TMI.2004.825627>.

</div>

<div id="ref-Tetteh2020" class="csl-entry">

Tetteh, Giles, Velizar Efremov, Nils D. Forkert, et al. 2020. “DeepVesselNet: Vessel Segmentation, Centerline Prediction, and Bifurcation Detection in 3-d Angiographic Volumes.” *Frontiers in Neuroscience* 14. <https://doi.org/10.3389/fnins.2020.592352>.

</div>

<div id="ref-TodorovVESSAP" class="csl-entry">

Todorov, Mihail Ivilinov, Johannes Christian Paetzold, Oliver Schoppe, et al. 2020. “Machine Learning Analysis of Whole Mouse Brain Vasculature.” *Nature Methods* 17 (4): 442–49.

</div>

<div id="ref-Vaswani2017" class="csl-entry">

Vaswani, Ashish, Noam Shazeer, Niki Parmar, et al. 2017. “Attention Is All You Need.” In *Advances in Neural Information Processing Systems*, edited by I. Guyon, U. Von Luxburg, S. Bengio, et al., vol. 30. Curran Associates, Inc. <https://proceedings.neurips.cc/paper_files/paper/2017/file/3f5ee243547dee91fbd053c1c4a845aa-Paper.pdf>.

</div>

<div id="ref-VICAVR" class="csl-entry">

*<span class="nocase">VICAVR dataset</span>*. n.d. <a href="http://www.varpa.es/research/ophtalmology.html#vicavr" class="uri">Http://www.varpa.es/research/ophtalmology.html#vicavr</a>.

</div>

<div id="ref-Wan2024" class="csl-entry">

Wan, Jia, Wanhua Li, Jason Ken Adhinarta, et al. 2024. *TriSAM: Tri-Plane SAM for Zero-Shot Cortical Blood Vessel Segmentation in VEM Images*. <https://arxiv.org/abs/2401.13961>.

</div>

<div id="ref-wang2023sam" class="csl-entry">

<span class="nocase">Wang, Haoyu, Sizheng Guo, Jin Ye, et al.</span> 2023. “Sam-Med3d: Towards General-Purpose Segmentation Models for Volumetric Medical Images.” *arXiv Preprint arXiv:2310.15161v3*.

</div>

<div id="ref-wasserthal2023totalsegmentator" class="csl-entry">

<span class="nocase">Wasserthal, Jakob, Hanns-Christian Breit, Manfred T Meyer, et al.</span> 2023. “TotalSegmentator: Robust Segmentation of 104 Anatomic Structures in CT Images.” *Radiology: Artificial Intelligence* 5 (5).

</div>

<div id="ref-Xu2024" class="csl-entry">

Xu, Yan, Rixiang Quan, Weiting Xu, Yi Huang, Xiaolong Chen, and Fengyuan Liu. 2024. “Advances in Medical Image Segmentation: A Comprehensive Review of Traditional, Deep Learning and Hybrid Approaches.” *Bioengineering* 11 (10). <https://doi.org/10.3390/bioengineering11101034>.

</div>

<div id="ref-Yagis2024" class="csl-entry">

Yagis, Ekin, Shahab Aslani, Yashvardhan Jain, et al. 2024. “Deep Learning for 3D Vascular Segmentation in Phase Contrast Tomography.” In *Res Sq*. July.

</div>

<div id="ref-yang2023medmnist" class="csl-entry">

Yang, Jiancheng, Rui Shi, Donglai Wei, et al. 2023. “Medmnist V2-a Large-Scale Lightweight Benchmark for 2d and 3d Biomedical Image Classification.” *Scientific Data* 10 (1): 41.

</div>

<div id="ref-Yang2024" class="csl-entry">

Yang, Kaiyuan, Fabio Musio, Yihui Ma, et al. 2024. “Benchmarking the CoW with the TopCoW Challenge: Topology-Aware Anatomical Segmentation of the Circle of Willis for CTA and MRA.” In *ArXiv*. April.

</div>

<div id="ref-Yu2024" class="csl-entry">

Yu, Ruonan, Songhua Liu, and Xinchao Wang. 2024. “Dataset Distillation: A Comprehensive Review.” *IEEE Transactions on Pattern Analysis and Machine Intelligence* 46 (1): 150–70. <https://doi.org/10.1109/TPAMI.2023.3323376>.

</div>

<div id="ref-Zhang2024" class="csl-entry">

Zhang, Duzhen, Yahan Yu, Jiahua Dong, et al. 2024. “Mm-Llms: Recent Advances in Multimodal Large Language Models.” *arXiv Preprint arXiv:2401.13601*.

</div>

<div id="ref-ZhangIOSTARRCSLO" class="csl-entry">

Zhang, Jiong, Behdad Dashtbozorg, Erik Bekkers, Josien P. W. Pluim, Remco Duits, and Bart M. ter Haar Romeny. 2016. “Robust Retinal Vessel Segmentation via Locally Adaptive Derivative Frames in Orientation Scores.” *IEEE Transactions on Medical Imaging* 35 (12): 2631–44. <https://doi.org/10.1109/TMI.2016.2587062>.

</div>

<div id="ref-Zheng2019" class="csl-entry">

Zheng, Hao, Lin Yang, Jianxu Chen, et al. 2019. “Biomedical Image Segmentation via Representative Annotation.” *Proceedings of the AAAI Conference on Artificial Intelligence* 33: 5901–8.

</div>

</div>
