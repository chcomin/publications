---
published: false
layout: article
title: Gland context networks: a novel approach for improving prostate cancer identification
authors:
- name: Rodrigo de P. Mendes
- name: Xin Yuan
- name: Elizabeth M. Genega
- name: Xiaoyin Xu
- name: Luciano da F. Costa
- name: Cesar H. Comin
  orcid: 0000-0003-1207-4982
date: 2021-11-06
year: 2021                         
journal: Computerized Medical Imaging and Graphics
volume: 94
issue: 1
pages: 101999
doi: 10.1016/j.compmedimag.2021.101999
abstract: |
  Prostate cancer (PCa) is a pervasive condition that is manifested in a wide range of histologic patterns in biopsy samples. Given the importance of identifying abnormal prostate tissue to improve prognosis, many computerized methodologies aimed at assisting pathologists in diagnosis have been developed. It is often argued that improved diagnosis of a tissue region can be obtained by considering measurements that can take into account several properties of its surroundings, therefore providing a more robust context for the analysis. Here we propose a novel methodology that can be used for systematically defining contextual features regarding prostate glands. This is done by defining a Gland Context Network (GCN), a representation of the prostate sample containing information about the spatial relationship between glands as well as the similarity between their appearance. We show that such a network can be used for establishing contextual features at any spatial scale, therefore providing information that is not easily obtained from traditional shape and textural features. Furthermore, it is shown that even basic features derived from a GCN can lead to state-of-the-art classification performance regarding PCa. All in all, GCNs can assist in defining more effective approaches for PCa grading.
keywords:
  - Prostate cancer
  - Gland segmentation
  - Graph representation
mathjax: true         
bibtex: |
  @article{mendes2021gland,
    title={Gland context networks: A novel approach for improving prostate cancer identification},
    author={Mendes, Rodrigo de P and Yuan, Xin and Genega, Elizabeth M and Xu, Xiaoyin and Costa, Luciano da F and Comin, Cesar H},
    journal={Computerized Medical Imaging and Graphics},
    volume={94},
    pages={101999},
    year={2021},
    publisher={Elsevier}
  }
---

# Introduction

Prostate cancer (PCa) is one of the most common types of cancer in men, representing one of the leading causes of cancer related deaths worldwide (Bray et al. 2018). Early detection of PCa can improve prognosis and significantly reduce the risk of death (Wong et al. 2016). Common approaches for PCa screening include rectal examination, prostate-specific antigen (PSA) testing and MRI (<span class="nocase">Schröder et al.</span> 2009; <span class="nocase">Andriole et al.</span> 2009; Sarkar and Das 2016). If PCa is suspected, a biopsy of the prostate can be performed to confirm the diagnosis (Patel and Jones 2009). Prostate samples extracted from the biopsy can be used for PCa grading and prognosis using the Gleason Grading System (GGS), a methodology developed by Donald Gleason (Gleason 1977) that has been refined by the International Society of Urological Pathology (ISUP) (<span class="nocase">Leenders et al.</span> 2020; Kryvenko and Epstein 2016; Epstein et al. 2016, 2005).

The GGS provides a set of patterns regarding the prostate tissue that a pathologist can use as reference for PCa prognosis. The patterns are usually related to how well-differentiated the prostate glands are. Non-differentiated glands are usually associated with PCa and thus with higher GGS scores. Currently, GGS scores are evaluated visually by pathologists, which may lead to varying scores between different pathologists, misdiagnosis and can also be time-consuming. Recently, there have been many efforts aimed at providing systematic and unbiased computerized segmentation and classification of prostate tissues for aiding in PCa analysis (Goldenberg et al. 2019; <span class="nocase">Nir et al.</span> 2018; Singh et al. 2017; Arvaniti et al. 2018; Doyle et al. 2010; Nguyen, Sabata, et al. 2012).

When identifying PCa in prostate tissue, context can be particularly important. A well-differentiated gland surrounded by non-differentiated glands is probably still associated with PCa. Thus, it is usual to define properties that are associated with the *context* of a given gland or tissue region. For instance Nguyen et al. (Nguyen, Sarkar, et al. 2012) considered not only the shapes of the glands for classification, but also the similarity between the shapes of nearby glands. Doyle et al. (Doyle et al. 2007) defined *architectural features* based on graphs constructed from cell nuclei as well as gland centroids. The features included average edge length, edge length variability, as well as Haralick’s texture features (Haralick et al. 1973). Despite their graph construction methodology being similar to our approach, the graphs were not used for providing context for individual gland features, but for defining additional features related only to connectivity. Monaco et al. (Monaco et al. 2008) used Markov random fields for propagating the labeling of cancerous glands in order to identify cancerous regions.

The definition of *ad hoc* contextual features may neglect important information for tissue classification. For instance, if elongated glands tended to appear together in PCa tissue, but gland elongation was measured only individually, their proximity could not be considered for classifying the tissue samples. Defining specific contextual features can be avoided by using, for instance, end-to-end Convolutional Neural Networks (CNN) (Arvaniti et al. 2018; Goldenberg et al. 2019). <span style="color: bluecolor">However, the results obtained by CNNs are usually more difficult to interpret than those obtained using handcrafted features.</span>

In the current work we describe a methodology for systematically assigning context to any feature used for prostate tissue characterization. The context is defined using what we call a *gland context network* (GCN), a weighted graph representing prostate glands and the relationship between their features. We show that many properties calculated based on the generated networks lead to tissue characterization that is not captured by traditional properties adopted in the literature. Thus, GCNs can be used not only for automated identification of PCa, but also for identifying novel features associated with PCa that may be useful for future revisions of the GGS. To show the potential of the methodology, we create and characterize GCNs for <span style="color: bluecolor">five</span> whole-mount prostate samples and use the generated structures to identify cancerous regions. For the considered case study, we identify that altered glands can be detected with high accuracy even by using only a simple network feature, the degree of the nodes.

The remainder of the article is organized as follows. In Section <a href="#s:basic_concepts" data-reference-type="ref" data-reference="s:basic_concepts">2</a> we present the main features commonly used for PCa classification as well as some basic concepts regarding network theory. The motivation and definition of GCNs is presented in Section <a href="#s:gcn" data-reference-type="ref" data-reference="s:gcn">3</a>. Section <a href="#s:experiments" data-reference-type="ref" data-reference="s:experiments">4</a> contains examples and a discussion about important parameters of GCNs as well as a case study of PCa classification for whole-mount prostate samples. In Section <a href="#s:conclusion" data-reference-type="ref" data-reference="s:conclusion">5</a> we present the conclusions.

# Basic Concepts

## Characterizing Gland Shape

The images considered in our analysis were obtained from staining models with hematoxylin and eosin (H&E) from prostate tissues. The main structures to be analyzed in the images are the several glands in the prostate tissue, which contain three main components: nuclei, cytoplasm and lumen. Figure <a href="#fig:GlandularStructure" data-reference-type="ref" data-reference="fig:GlandularStructure">[fig:GlandularStructure]</a> illustrates these structures. Figure <a href="#fig:GlandularStructure" data-reference-type="ref" data-reference="fig:GlandularStructure">[fig:GlandularStructure]</a>(a) shows a H&E stained prostate tissue image with one gland highlighted. The nuclei (figure <a href="#fig:GlandularStructure" data-reference-type="ref" data-reference="fig:GlandularStructure">[fig:GlandularStructure]</a>(b)) tends to be the darkest structure inside the gland, while the lumen (figure <a href="#fig:GlandularStructure" data-reference-type="ref" data-reference="fig:GlandularStructure">[fig:GlandularStructure]</a>(c)) is usually close to the white color. The cytoplasm (figure <a href="#fig:GlandularStructure" data-reference-type="ref" data-reference="fig:GlandularStructure">[fig:GlandularStructure]</a>(d)) tends to be stained in a light pink color. The gland is mainly the union of these three structures (figure <a href="#fig:GlandularStructure" data-reference-type="ref" data-reference="fig:GlandularStructure">[fig:GlandularStructure]</a>(e)).

The grading system described by Gleason (Gleason 1977) and modified and accepted by the ISUP considers, among other characteristics, the shape and size of the glands to identify the architectural growth patterns (<span class="nocase">Leenders et al.</span> 2020; Epstein et al. 2016, 2005). In the revisions of the GGS made by the ISUP in 2014 and 2019, the shape is mentioned as a predictor for several of the scores. For instance, benign patterns tend to have large circular and oval glands, while Gleason grade 3 patterns usually have smaller and more circular glands when compared to benign patterns. Furthermore, irregular glandular structures can be observed on Gleason grade 4 patterns since some glands are undifferentiated, fused with other glands, or contain many small holes (cribriform patterns) (Nguyen, Sabata, et al. 2012; Epstein et al. 2016).

The GGS provides a comprehensive reference for the many different patterns that can be observed in prostate tissues. Still, the scores are evaluated qualitatively. Thus, it is relevant to define quantitative values for the shape characteristics considered by the specialists in PCa prognosis. Many shape features were used in previous automated PCa prognosis studies (Nguyen, Sabata, et al. 2012; Nguyen, Sarkar, et al. 2012; Naik et al. 2008, 2007). In order to show the potential of the GCC methodology, a set of shape features identified as relevant for PCa prognosis in the aforementioned studies was selected. The considered features and the respective definitions used in this work are:

**Area**: quantifies the size of the gland. It is calculated as the number of pixels representing the gland. The area is considered an important feature in automated PCa prognosis since it tends to change as the cancer grows (Nguyen, Sarkar, et al. 2012; <span class="nocase">Epstein et al.</span> 2005, 2016).

**Perimeter**: it is associated with the size, and thus with the area, of a gland, but it provides complementary information since two shapes having similar areas can have very distinct perimeters. It is calculated as the arc length of the parametric contour of the gland (Cesar Jr and Fontura Costa 2009).

**Solidity**: quantifies how close the shape of the gland is to a convex polygon. It is calculated as the ratio between the area of the gland and the area of the convex hull of the gland (Cesar Jr and Fontura Costa 2009). Glands having irregular contours tend to have low solidity.

**Eccentricity**: represents the elongation of a gland. In this work it is calculated as the ratio between the eigenvalues of the covariance matrix calculated from the positions of the pixels representing the shape (Cesar Jr and Fontura Costa 2009). Interestingly, patterns with a score of 5 in the GGS tend to contain glands having cylindrical shapes (Epstein et al. 2016). The cross-section of such glands can appear as elongated shapes in tissue samples.

**Diameter**: quantifies the maximum distance between any two points in the contour of a gland.

## Complex Networks

In the past 20 years complex networks became an important tool for analyzing interconnected systems such as protein-protein interactions (<span class="nocase">Han et al.</span> 2004; Vella et al. 2018), social networks (Castellano et al. 2009) and cities (Domingues et al. 2018). A network or graph can be understood as a topological abstraction of the system in which the elements are represented by nodes (Costa 2004; Fontoura Costa 2006), with are interconnected by edges if they are related according to some criterion. Such an abstraction provides a common representation for many distinct systems, allowing the respective application of methodologies from network theory for characterizing and modeling their dynamics. For the purpose of defining context networks, two main aspects need to be considered. They are the methodology used for building the network and the properties used for characterizing their nodes. Those aspects are presented and discussed below.

### Defining Connectivity

A network can be mathematically represented as $`G = (V,E)`$, where $`V`$ is the set of nodes and $`E`$ the set of edges, reflecting the connectivity between pairs of nodes. Therefore, two nodes $`a,b`$ of $`V`$ are said to be adjacent, or connected, if $`\left(a,b\right) \in E`$. A network can also be weighted, in which case a value is associated to each of its edges. Usually, the weight is associated with the degree of relationship between the nodes, quantified by some similarity or dissimilarity metric. Networks can also be categorized as being spatial or non-spatial. In spatial networks, each node has a specific position in some metric space, usually the Euclidean space, and the distance between pairs of nodes influences their connectivity. In the case of non-spatial networks, other methodologies for connecting the nodes are considered. The most common approaches include connecting the nodes randomly (Erdős P 1959), with preference to large-degree nodes (Barabási and Albert 1999) or according to a fixed sequence of the number of connections made by each node (Newman 2018). In the following, we only consider spatial networks. Still, we will show that GCN can greatly benefit from defining connectivity from both the spatial distance and the shape and texture similarity of the glands they represent.

Many different approaches can be used for taking into account the distance between nodes when deciding if they should be connected or not. In the experiments we only consider the Geometric Graph criterion (Dall and Christensen 2002), explained below, but relevant alternatives are: i) connecting nodes according to a probability that decays exponentially with the distance between the nodes, known as the Waxman criterion (Waxman 1988); ii) connecting nodes having adjacent Voronoi cells (Barthélemy 2011); and iii) connecting pairs of nodes only if a disk that is centered at the midpoint of the two nodes and have a diameter equal to the distance between the nodes do not contain any other node (Gabriel and Sokal 1969). We will show that the methodology used for defining connectivity can be easily changed when generating context graphs.

For creating a geometric graph, nodes are connected by an edge if they are separated by a distance less than $`2r`$, where $`r`$ is a parameter of the model. Therefore, each node can be seen as having a region of influence, given by a disk of radius $`r`$ centered at the node. Two nodes are connected if their corresponding disks overlap. Parameter $`k`$ has a large impact on many properties of the generated network. Particularly, the expected number of connections of a node is proportional do $`r`$.

### Network Properties

Given a network, it is relevant to characterize the nodes according to the respective topology, that is, the pattern of connections made by the nodes. In order to do so, we first define two key concepts. The *shortest path length* between two nodes $`V_i`$ and $`V_j`$ is calculated as the number of edges in the smallest path between the two nodes, where a path is defined as a sequence of edges in the network. The *$`k`$-th neighborhood* of a given node $`V_i`$ is defined as the set of nodes that can be reached from node $`V_i`$ using paths with length smaller than or equal to $`k`$.

When characterizing the connectivity of nodes, not only the immediate connections of the node need to be considered. For instance, it might be interesting to measure the number of connections among the neighbors of a node, or among the neighbors of the neighbors of that node. If only nodes in the first neighborhood of a node are considered, the respective characterization will be very local, that is, it will concern only nodes directly associated with the reference node (see green region in Figure <a href="#fig:neighborhoods" data-reference-type="ref" data-reference="fig:neighborhoods">[fig:neighborhoods]</a>). Contrariwise, if a large neighborhood is considered, nodes indirectly associated with the reference node will be also taken into account (see yellow region in Figure <a href="#fig:neighborhoods" data-reference-type="ref" data-reference="fig:neighborhoods">[fig:neighborhoods]</a>). This will be a key concept in Context Networks, since the neighborhood used for characterizing the node will set the desired context.

Many topological features can be used for characterizing the connectivity of a network (Costa et al. 2007). In the following, we describe some basic features that will be considered for evaluating the potential of GCNs.

**Degree:** The number of connections made by a node. It is usually associated with the importance, or centrality, of a node since nodes with large degrees are associated with many other nodes in the network. The probability distribution of the node degrees, known as the degree distribution, can reveal many important properties about the network (Costa et al. 2007). An illustration of this property is shown in Figure <a href="#fig:NetworkProperties" data-reference-type="ref" data-reference="fig:NetworkProperties">[fig:NetworkProperties]</a>(a).

**Strength**: Defined for weighted networks, it is calculated as the sum of weights of the edges connected to a node. It can be considered as the degree counterpart for weighted networks. Thus, it can also be interpreted as a measure of importance of a node (Costa et al. 2007). Figure <a href="#fig:NetworkProperties" data-reference-type="ref" data-reference="fig:NetworkProperties">[fig:NetworkProperties]</a>(b) shows a weighted network and the corresponding edge weights and node strengths.

**Betweenness centrality**: The shortest path length between two nodes is of great importance since it underlies the concept of proximity for nodes in a network. The betweenness centrality of a node $`V_i`$ is associated with the number of shortest paths between any two nodes in the network that pass through node $`V_i`$ (Costa et al. 2007). Nodes with large betweenness can usually be seen as “bridges” between different groups of nodes in the network. Figure <a href="#fig:NetworkProperties" data-reference-type="ref" data-reference="fig:NetworkProperties">[fig:NetworkProperties]</a>(c) shows an example of nodes with large betweenness.

# Gland Context Networks

As presented in Section <a href="#s:shape_text" data-reference-type="ref" data-reference="s:shape_text">2.1</a>, glands can be characterized using shape and texture features. However, in the GGS it is stated that gland appearance should not be considered isolatedly (Kumar et al. 2014; Nguyen, Sarkar, et al. 2012; <span class="nocase">Epstein et al.</span> 2005, 2005), that is, the context of the gland is also important. For instance, a single gland having an appearance that might be associated with healthy glands, if surrounded by undifferentiated glands, will still be associated with a high Gleason score. For such reasons, some methodologies for defining context have been developed (Nguyen, Sarkar, et al. 2012; Doyle et al. 2007; Monaco et al. 2008). Yet, the context is usually not taken into account in a systematic fashion, or it is only considered for specific properties.

Here we define a systematic approach for setting the context of a gland using what we call a *Context Network*. Glands are represented as nodes, and connections indicate some relationship between glands. This relationship can be spatial, in the sense that the distances between the glands are used when defining the connectivity, may consider only the shape or texture of the glands, that is, similar glands tend to be connected, or it can be a mixture of both aspects. The obtained network can then be characterized using complex network properties (Costa et al. 2007), which are used as additional features for PCa prognosis.

Let $`S`$ represent the set of glands in a sample, where the $`i`$-th gland is referred as $`s_i`$. Suppose $`m`$ shape and texture features $`f_1^i,f_2^i,\dots,f_m^i`$ have been measured for each gland $`s_i`$. These features can be represented by a vector $`\vec{f}^i`$. Each gland also has a position $`\vec{p}^i=(p^i_1,p^i_2)`$. In the following, we consider that the position is given by the centroid of the gland.

The spatial distance between two glands $`i`$ and $`j`$ is calculated as

``` math
\begin{equation}
d^s_{ij} = \sqrt{\sum_{k=1}^{2}(\tilde{p}^i_k-\tilde{p}^j_k)^2}
\end{equation}
```
where <span style="color: bluecolor">$`\tilde{p}^i_k`$</span> is the standard score of the $`k`$-th coordinate of gland $`i`$, that is,

``` math
\begin{equation}
\tilde{p}^i_k = \frac{p^i_k - \mu_k}{\sigma_k}
\end{equation}
```
where $`\mu_k`$ and $`\sigma_k`$ are, respectively, the average and standard deviation of the $`k`$-th coordinate of all the glands. The purpose of the normalization is to allow the definition of a general distance involving both the spatial coordinates of the glands and their shape and texture features.

The difference between the features of two glands is calculated as

``` math
\begin{equation}
d^f_{ij} = \sqrt{\sum_{k=1}^{m}(\tilde{f}^i_k-\tilde{f}^j_k)^2}
\end{equation}
```
where $`\tilde{f}^i_k`$ represents the standard score of $`f^i_k`$, defined using the same approach describe above.

A combination of the distance between the glands and their shape and texture similarity is then calculated as

``` math
\begin{equation}
    h_{ij} = \sqrt{\alpha \left(d^s_{ij}\right)^2 + (1-\alpha) \left(d^f_{ij}\right)^2},
\end{equation}
```
where $`\alpha \in [0,1]`$.

The weight of the connection between glands $`i`$ and $`j`$ is then given by

``` math
\begin{equation}
    w_{ij} = e^{-h_{ij}}.
\end{equation}
```

Parameter $`\alpha`$ sets the relative importance between the spatial distance and the feature similarity between the glands. When $`\alpha=1`$, only the spatial distance is taken into account, while $`\alpha=0`$ creates a context network in which edge weights are defined using only the appearance of the glands. It is important to note that the context network can be created for all features or using only a subset of them. For instance, it is possible to create an area context network, considering only this feature of the glands.

A GCN can be created using different combinations of distances $`d^s_{ij}`$ and $`d^f_{ij}`$. For instance, it is possible to use one value of $`\alpha`$ for defining the connectivity between nodes and another value for defining edge weights. Here we consider an approach for generating a network that is easy to visualize and interpret. The connections between nodes are defined using the geometric graph approach, presented in Section <a href="#s:def_con" data-reference-type="ref" data-reference="s:def_con">2.2.1</a>. Thus, the connectivity is defined using $`\alpha=1`$ and a given radius $`r`$ that is a parameter of the method. Next, edge weights are defined using $`\alpha=0`$, that is, only considering shape and texture features of the glands. This generates a network where only the distances between the glands define the existence or absence of edges and the weights of the edges depends only on the similarity between the appearance of the glands. Still, it is important to keep in mind that GCNs allow alternative approaches for building the network.

Having obtained the network, the nodes can be characterized using the properties discussed in Section <a href="#s:graph_prop" data-reference-type="ref" data-reference="s:graph_prop">2.2.2</a>. Therefore, the original feature vectors used for classifying glands or samples into healthy or unhealthy can be systematically enhanced by considering the context of the glands. Figure <a href="#fig:FluxDiagram" data-reference-type="ref" data-reference="fig:FluxDiagram">[fig:FluxDiagram]</a> illustrates the advantage of using GCNs for adding context for PCa analysis. The usual steps applied for PCa classification or GGS assessment are represented in green. GCNs can be added to the analysis for augmenting the original shape and texture features.

It is important to note that GCNs cannot be used for characterizing latter stages of PCa since the glands become undifferentiated. Thus, GCNs are aimed at providing early prognosis and also in revealing novel properties for tissues containing differentiated or partially differentiated glands.

# Experiments

In the following we provide examples of GCNs created using prostate tissue samples. We describe important properties that can be obtained from the generated networks and also provide an example of identifying PCa.

## Dataset

The dataset used in the examples consists of sections from <span style="color: bluecolor">five</span> whole-mount prostate samples. The slides were imaged by a brightfield automatic microscope, which was configured to scan over the whole specimen and form a large mosaic picture composed of 40 to 50 individual images. Figure <a href="#fig:DataSetAndMaskGlands" data-reference-type="ref" data-reference="fig:DataSetAndMaskGlands">[fig:DataSetAndMaskGlands]</a> shows one of the images in the dataset. The original sample is shown in Figure <a href="#fig:DataSetAndMaskGlands" data-reference-type="ref" data-reference="fig:DataSetAndMaskGlands">[fig:DataSetAndMaskGlands]</a>(a). A specialist analyzed the sample and identified two regions evaluated at a score of 3+3 in the GGS for PCa, which are indicated in red in the image. Figure <a href="#fig:DataSetAndMaskGlands" data-reference-type="ref" data-reference="fig:DataSetAndMaskGlands">[fig:DataSetAndMaskGlands]</a>(b) shows a binary image containing manually segmented glands. We note that the focus of the methodology is on classifying individual glands. The <span style="color: bluecolor">five</span> samples together contain <span style="color: bluecolor">9491</span> gland units, which was deemed enough for assessing the potential of GCNs in representing and characterizing prostate glands.

## Gland Context Network Creation

Following the procedure described in Section <a href="#s:gcn" data-reference-type="ref" data-reference="s:gcn">3</a>, GCNs were created based on manual segmentations of the glands in whole-mount prostate samples. Figure <a href="#fig:GeometricGraph" data-reference-type="ref" data-reference="fig:GeometricGraph">[fig:GeometricGraph]</a> shows an example of GCN visualized without considering edges weights, only the connectivity between glands. The connectivity alone can already reveal many interesting facts about the glands and their context. First, it is clear that glands tend to organize into clusters, defining subgraphs containing nodes that are more connected between themselves than with nodes that are not in the subgraph. Such clusters are commonly called *communities* in network theory (Fortunato 2010). Communities tend to be associated with components of a system that carry out similar functions. Many community detection methods have been developed in the literature (Fortunato 2010) and could be used for identifying clusters of glands. This procedure could bring additional insights regarding gland organization when compared to the identification of connected components. The latter has been used for defining context in prostate tissues (Nguyen, Sarkar, et al. 2012).

The generated GCNs are also very heterogeneous. Some regions contain a large number of connections while others contain many nodes having small degree. Examples of two highly distinct regions can be seen in Figures <a href="#fig:GeometricGraph" data-reference-type="ref" data-reference="fig:GeometricGraph">[fig:GeometricGraph]</a>(a) and (b). It is important to note that variations on edge density tend to be the most noticeable characteristic when visualizing GCNs, but these networks may contain many other important information that is not immediately identified by visual inspection. For instance, the clustering coefficient (Costa et al. 2007), which, roughly speaking, is associated to the number of triangles connected to a node, varies throughout the network.

Besides connectivity, GCNs can also represent similarity between glands. Figure <a href="#fig:PonderedGraph" data-reference-type="ref" data-reference="fig:PonderedGraph">[fig:PonderedGraph]</a> shows two regions of a weighted GCN created using the method presented in Section <a href="#s:gcn" data-reference-type="ref" data-reference="s:gcn">3</a>. Thicker edges are associated to edges having larger weight, and thus to glands having similar shapes. In this case, all shape properties described in Section <a href="#s:shape_text" data-reference-type="ref" data-reference="s:shape_text">2.1</a> were used for defining the weights according to Equation <a href="#eq:weights" data-reference-type="ref" data-reference="eq:weights">[eq:weights]</a>. Among other properties, the GCN reveals clusters of glands that are close to each other and have similar shapes.

As mentioned before, GCNs can be created using any set of gland properties. The main purpose being to allow a systematic incorporation of context to the considered properties. The context is created by calculating complex networks features based on the generated GCN. For instance, the degree of a node incorporates a local context regarding neighboring glands. Figure <a href="#fig:ColorGraph" data-reference-type="ref" data-reference="fig:ColorGraph">[fig:ColorGraph]</a> shows the degrees of the nodes in the GCN created from the sample in Figure <a href="#fig:DataSetAndMaskGlands" data-reference-type="ref" data-reference="fig:DataSetAndMaskGlands">[fig:DataSetAndMaskGlands]</a>. Interestingly, larger degree nodes tend to be located inside or near the region demarcated by the specialist (shown in Figure <a href="#fig:DataSetAndMaskGlands" data-reference-type="ref" data-reference="fig:DataSetAndMaskGlands">[fig:DataSetAndMaskGlands]</a>), suggesting a relationship between node degree and PCa.

The degree can reveal local context, but other network measurements could be used for augmenting glands properties at medium, global, or even specific scales (Fontoura Costa and Silva 2006). Such measurements may reveal additional information about gland properties that are not easily quantifiable using traditional methods. As a consequence, GCN derived properties may improve the accuracy of PCa identification and also of automated Gleason score assignment.

## A case study: Gland Classification Using Context Networks

As an example of the potential of using GCNs for identifying PCa, we employed GCN properties for classifying glands into healthy and unhealthy (GGS score of 3+3, as indicated by the specialist). The classification, done at the individual gland level, can then be used for identifying abnormal tissue regions. Different sets of properties were obtained and used for classification: i) shape features (SF), ii) network features (NF), iii) all features (AF), and iv) only the degree feature (DF).

In order to quantify the performance of the method, we measured the precision, recall, specificity and accuracy of the classification (Fawcett 2006). Glands belonging to the region demarcated by the specialist and that were classified as unhealthy defined the true positive (TP) samples. The FN, TN and FP samples were defined accordingly. <span style="color: bluecolor">For classifying the glands, we considered the K-nearest neighbors, Support Vector Machine (SVM) and Multilayer Perceptron classifiers. Each method was applied using 5-fold cross-validation. Each fold of the cross-validation contained the same number of healthy and unhealthy glands. A grid search of the methods parameters was applied for searching their respective optimal configurations. The three methods provided similar classification performance. Thus, in the following we present only the results obtained for the K-nearest neighbors classifier with $`K=5`$.</span>

Figure <a href="#fig:GlandsColorizedByClassification" data-reference-type="ref" data-reference="fig:GlandsColorizedByClassification">[fig:GlandsColorizedByClassification]</a> shows a typical result obtained from the cross-validation using the whole set of features. The colors differentiate between TP, FN, TN and FP glands. It is clear that the vast majority of glands are correctly classified. Interestingly, most of the incorrectly classified glands are near the border of the region demarcated by the specialist, indicating that these glands might already have been affected by PCa.

The performance metrics of gland classification were measured for <span style="color: bluecolor">five</span> whole-mount samples and the four sets of properties described above. The average value obtained for each metric is shown in Figure <a href="#fig:ConfusionMatrix" data-reference-type="ref" data-reference="fig:ConfusionMatrix">[fig:ConfusionMatrix]</a>. It is clear that features derived from the GCNs improved the classification when compared to using the gland shape features alone. The average accuracy obtained when using the network feature set was around <span style="color: bluecolor">$`93\% \pm 2.5\%`$</span>, which is better or equivalent to state-of-the-art results obtained by other approaches such as 91.9% by Nir et al. (<span class="nocase">Nir et al.</span> 2018), 85.4% by Doyle et al. (Doyle et al. 2007), 95.14% by Naik et al. (Naik et al. 2008), and 79% by Nguyen et al. (Nguyen, Sarkar, et al. 2012). However, it is important to note that, with the exception of Naik et al., the other approaches used automatically segmented glands in the classification.

Figure <a href="#fig:ConfusionMatrix" data-reference-type="ref" data-reference="fig:ConfusionMatrix">[fig:ConfusionMatrix]</a> also shows that the degree alone led to a good classification performance. The degree tends to be higher in regions containing small glands in large numbers. Thus, the GCN methodology revealed, in a straightforward fashion, that these two properties were important for identifying tissue regions affected by PCa in the analyzed samples.

<span style="color: bluecolor">Segmenting glands is a hard task. Changes in the segmentation might influence the generation of the GCN, which in turn will influence the performance of classification tasks. In order to verify the impact that changes in the segmentation can have in the gland classification, four types of perturbations were applied to the manual annotations of the glands: i) binary erosion; ii) binary dilation; iii) binary dilation while avoiding mergings of the glands and iv) random removal of glands. The erosion and dilation procedures represent situations where the segmented glands are smaller or larger than they should be. Procedure ii) tends to generate very large connected components due to the dilation procedure, which is unrealistic in real situations. On the other hand, procedure iii) leads to dilated glands but the number of glands do not change. The random removal of glands represents situations where glands are missed either by an expert annotator or by automated segmentation procedures.</span>

<span style="color: bluecolor">Figure <a href="#fig:ResPertub" data-reference-type="ref" data-reference="fig:ResPertub">[fig:ResPertub]</a> shows the performance metrics obtained for different amounts of erosion, dilation and gland removal iterations applied to the segmented glands. For the erosion and dilation, the performance is measured as a function of the radius of the structuring element used in the respective morphological operation. Regarding the erosion (Figure <a href="#fig:ResPertub" data-reference-type="ref" data-reference="fig:ResPertub">[fig:ResPertub]</a>(a)), the method is robust to erosions of up to a radius of 13 pixels. We observed that, for this radius, the segmented glands are much smaller than the actual glands in the image. This can be verified by the disappearance of many glands for radii larger than 13 pixels, as seen in the plot. For instance, for a radius of 15 pixels roughly 15% of the glands disappear. Given that the method shows good performance even when the segmentation of the glands has low quality (e.g., for a a radius of 13 pixels), we consider that the method is robust to the erosion perturbation.</span>

<span style="color: bluecolor">Regarding the robustness of the method with respect to the dilation operation (Figure <a href="#fig:ResPertub" data-reference-type="ref" data-reference="fig:ResPertub">[fig:ResPertub]</a>(b)), it is clear that small amounts of dilation can significantly decrease the performance of the classification. In particular, the precision becomes much lower after only 5 dilations. On the other hand, as explained above, since the glands are very close to each other, the dilation procedure leads to large connected components covering almost the whole sample, which do not realistically represent glands or possible errors in manual annotations. This can be seen by the rapid decrease in gland number as the dilation becomes larger. Figure <a href="#fig:ResPertub" data-reference-type="ref" data-reference="fig:ResPertub">[fig:ResPertub]</a>(c) shows the result for the dilation procedure while avoiding gland merges. In this case, there is almost no change in the classification performance. This happens because, as mentioned before, features related to the spatial distribution of the glands tended to be more relevant for classification than those associated with their shapes. Since the dilation without gland merges does not change the spatial distribution of the glands, the performance remains high. </span>

<span style="color: bluecolor">The results obtained for the random removal of glands are shown in Figure <a href="#fig:ResPertub" data-reference-type="ref" data-reference="fig:ResPertub">[fig:ResPertub]</a>(d). Surprisingly, a good performance is obtained even when around 60% of the glands are removed. This is so because when glands are randomly removed with uniform probability, their overall spatial distribution does not change. For instance, after the removal, regions having a high density of glands will still have relatively large density when compared to other regions in the sample. The information regarding the spatial distribution of the remaining glands is enough to correctly classify them.</span>

<span style="color: bluecolor">The results of the perturbation analysis indicate that, at least for the considered data, GCNs are robust with respect to the considered changes in the shape of the glands as well as to situations where some glands are not correctly segmented.</span>

<span style="color: bluecolor">A challenging aspect for quantifying the performance of tumor detection methodologies is that it is not straightforward to define precise boundaries of a tumor. This can be a source of variation on the ground truth gland classes used for measuring the performance of the method. In order to verify to which extent the manual annotation of the tumor regions influenced our results, an intra-expert evaluation procedure was employed. The results of the methodology were shown to the specialist that originally identified the tumor regions in our samples. The annotations were re-evaluated by the specialist considering the TP, FN, TN and FP glands identified by our analysis. The specialist’s assessment was that TP and TN glands were indeed correctly classified. Interestingly, the specialist found that the majority of FN glands, that is, glands inside the delimited tumor region but that were classified by our method as being normal (green glands in Figure <a href="#fig:GlandsColorizedByClassification" data-reference-type="ref" data-reference="fig:GlandsColorizedByClassification">[fig:GlandsColorizedByClassification]</a>), were indeed normal glands. Those glands should be considered normal because, despite being inside or close to the tumor region, they had a more disperse spatial distribution and still had basal cells. Furthermore, many glands that were close to, but outside, the cancerous region and were classified as being abnormal by the method (yellow glands in Figure <a href="#fig:GlandsColorizedByClassification" data-reference-type="ref" data-reference="fig:GlandsColorizedByClassification">[fig:GlandsColorizedByClassification]</a>) were suspicious of being tumor acini (atypical acini), given their small sizes and small distances to each other. An absence of basal cells on these glands was also observed. Thus, the method successfully identified abnormal glands that were previously not included in the tumor region.</span>

# Conclusions

The automated or semi-automated characterization of prostate tissue can provide important support for pathologists to identify abnormal tissues. Furthermore, analyzing prostate images at a large scale, using whole-mount samples, may reveal important factors for grading PCa that have not been accounted for in the most up-to-date revision of the ISUP guidelines (<span class="nocase">Leenders et al.</span> 2020). As a consequence, some automated methods have been developed for segmenting, characterizing and classifying prostate tissue (Goldenberg et al. 2019; <span class="nocase">Nir et al.</span> 2018; Singh et al. 2017; Arvaniti et al. 2018; Nguyen, Sabata, et al. 2012). Since the developed methods tend to have an automated segmentation as a first step, the remaining analyses becomes highly dependent on the quality of the segmentation. For instance, it may be difficult to identify if a poor performance on identifying abnormal tissue was due to inadequate features or if it was caused by tissue not being correctly segmented.

In this work we assumed that the glands have already been correctly segmented, either manually or automatically, and focused on a methodology for characterizing the identified glands. The concept of a Gland Context Network (GCN), a weighted graph describing the spatial and visual relationship between the glands, was presented. We argued that one of the main benefits of using GCNs is that the vast literature regarding complex networks characterization becomes immediately available for augmenting individual gland measurements.

The proposed methodology can be used for a more comprehensive characterization of tissues by integrating information from varying spatial scales. As an illustration, we created GCNs for whole-mount samples and showed that heterogeneous networks with many interesting properties are created and can be further investigated. In particular, clusters of glands are revealed and might be identified using community detection algorithms. Furthermore, we showed that even in a straightforward gland classification procedure, using an off-the-shelf classifier, GCN properties led to the identification of the vast majority of abnormal glands. The classification performance was verified to be high even when the glands are not correctly segmented.

For future analyses, GCNs could be defined and applied for a large prostate tissue dataset for the identification of novel contextual properties associated to different Gleason scores. Furthermore, GCNs could be used for further improving the classification of the glands using a two step classification scheme, applied as follows. As can be seen in Figure <a href="#fig:GlandsColorizedByClassification" data-reference-type="ref" data-reference="fig:GlandsColorizedByClassification">[fig:GlandsColorizedByClassification]</a>, glands that were incorrectly classified are usually surrounded by correctly classified glands. Thus, one could apply a consensus formation dynamics, such as opinion spreading (Sznajd-Weron and Sznajd 2000; Castellano et al. 2009), to propagate gland categories throughout the GCN, which would correct most of the misclassifications. <span style="color: bluecolor">The methodology could also be applied to other types of tissues, provided that appropriate features are used for building the GCN.</span>

# Acknowledgments

Cesar H. Comin thanks FAPESP (grant no. 18/09125-4) for financial support. Luciano da F. Costa thanks CNPq (grant no. 307085/2018-0) for sponsorship. This work has also been supported by the FAPESP grant 15/22308-2.

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-andriole2009mortality" class="csl-entry">

<span class="nocase">Andriole, Gerald L, E David Crawford, Robert L Grubb III, et al.</span> 2009. “Mortality Results from a Randomized Prostate-Cancer Screening Trial.” *New England Journal of Medicine* 360 (13): 1310–19.

</div>

<div id="ref-arvaniti2018automated" class="csl-entry">

Arvaniti, Eirini, Kim S Fricker, Michael Moret, et al. 2018. “Automated Gleason Grading of Prostate Cancer Tissue Microarrays via Deep Learning.” *Scientific Reports* 8 (1): 1–11.

</div>

<div id="ref-barabasi1999emergence" class="csl-entry">

Barabási, Albert-László, and Réka Albert. 1999. “Emergence of Scaling in Random Networks.” *Science* 286 (5439): 509–12.

</div>

<div id="ref-barthelemy2011spatial" class="csl-entry">

Barthélemy, Marc. 2011. “Spatial Networks.” *Physics Reports* 499 (1-3): 1–101.

</div>

<div id="ref-bray2018global" class="csl-entry">

Bray, Freddie, Jacques Ferlay, Isabelle Soerjomataram, Rebecca L Siegel, Lindsey A Torre, and Ahmedin Jemal. 2018. “Global Cancer Statistics 2018: GLOBOCAN Estimates of Incidence and Mortality Worldwide for 36 Cancers in 185 Countries.” *CA: A Cancer Journal for Clinicians* 68 (6): 394–424.

</div>

<div id="ref-castellano2009statistical" class="csl-entry">

Castellano, Claudio, Santo Fortunato, and Vittorio Loreto. 2009. “Statistical Physics of Social Dynamics.” *Reviews of Modern Physics* 81 (2): 591.

</div>

<div id="ref-cesar2009shape" class="csl-entry">

Cesar Jr, Roberto Marcondes, and Luciano da Fontura Costa. 2009. *Shape Classification and Analysis: Theory and Practice*. CRC Press.

</div>

<div id="ref-costa2007characterization" class="csl-entry">

Costa, L da F, Francisco A Rodrigues, Gonzalo Travieso, and Paulino Ribeiro Villas Boas. 2007. “Characterization of Complex Networks: A Survey of Measurements.” *Advances in Physics* 56 (1): 167–242.

</div>

<div id="ref-costa2004complex" class="csl-entry">

Costa, Luciano da Fontoura. 2004. “Complex Networks, Simple Vision.” *arXiv Preprint Cond-Mat/0403346*.

</div>

<div id="ref-dall2002random" class="csl-entry">

Dall, Jesper, and Michael Christensen. 2002. “Random Geometric Graphs.” *Physical Review E* 66 (1): 016121.

</div>

<div id="ref-domingues2018topological" class="csl-entry">

Domingues, Guilherme S, Filipi N Silva, Cesar H Comin, and L da F Costa. 2018. “Topological Characterization of World Cities.” *Journal of Statistical Mechanics: Theory and Experiment* 2018 (8): 083212.

</div>

<div id="ref-doyle2010boosted" class="csl-entry">

Doyle, Scott, Michael Feldman, John Tomaszewski, and Anant Madabhushi. 2010. “A Boosted Bayesian Multiresolution Classifier for Prostate Cancer Detection from Digitized Needle Biopsies.” *IEEE Transactions on Biomedical Engineering* 59 (5): 1205–18.

</div>

<div id="ref-doyle2007automated" class="csl-entry">

Doyle, Scott, Mark Hwang, Kinsuk Shah, Anant Madabhushi, Michael Feldman, and John Tomaszeweski. 2007. “Automated Grading of Prostate Cancer Using Architectural and Textural Image Features.” *Biomedical Imaging: From Nano to Macro, 2007. ISBI 2007. 4th IEEE International Symposium on*, 1284–87.

</div>

<div id="ref-epstein20052005" class="csl-entry">

<span class="nocase">Epstein, Jonathan I, William C Allsbrook Jr, Mahul B Amin, Lars L Egevad, ISUP Grading Committee, et al.</span> 2005. “The 2005 International Society of Urological Pathology (ISUP) Consensus Conference on Gleason Grading of Prostatic Carcinoma.” *The American Journal of Surgical Pathology* 29 (9): 1228–42.

</div>

<div id="ref-epstein20162014" class="csl-entry">

Epstein, Jonathan I, Lars Egevad, Mahul B Amin, Brett Delahunt, John R Srigley, and Peter A Humphrey. 2016. “The 2014 International Society of Urological Pathology (ISUP) Consensus Conference on Gleason Grading of Prostatic Carcinoma.” *The American Journal of Surgical Pathology* 40 (2): 244–52.

</div>

<div id="ref-erdHos1959renyi" class="csl-entry">

Erdős P, Rényi A. 1959. “On Random Graphs.” *Publicationes Mathematicae Debrecen* 6 (290-297).

</div>

<div id="ref-fawcett2006introduction" class="csl-entry">

Fawcett, Tom. 2006. “<span class="nocase">An introduction to ROC analysis</span>.” *Pattern Recognition Letters* 27 (8): 861–74.

</div>

<div id="ref-da2006complex" class="csl-entry">

Fontoura Costa, Luciano da. 2006. “Complex Networks: New Concepts and Tools for Real-Time Imaging and Vision.” *arXiv Preprint Cs.CV/0606060*.

</div>

<div id="ref-da2006hierarchical" class="csl-entry">

Fontoura Costa, Luciano da, and Filipi Nascimento Silva. 2006. “Hierarchical Characterization of Complex Networks.” *Journal of Statistical Physics* 125 (4): 841–72.

</div>

<div id="ref-fortunato2010community" class="csl-entry">

Fortunato, Santo. 2010. “Community Detection in Graphs.” *Physics Reports* 486 (3-5): 75–174.

</div>

<div id="ref-gabriel1969new" class="csl-entry">

Gabriel, K Ruben, and Robert R Sokal. 1969. “A New Statistical Approach to Geographic Variation Analysis.” *Systematic Zoology* 18 (3): 259–78.

</div>

<div id="ref-gleason1977veteran" class="csl-entry">

Gleason, DF. 1977. “<span class="nocase">The Veteran’s Administration Cooperative Urologic Research Group: histologic grading and clinical staging of prostatic carcinoma</span>.” In *Urologic Pathology: The Prostate*, edited by M Tannenbaum. Philadelphia: Lea; Febiger.

</div>

<div id="ref-goldenberg2019new" class="csl-entry">

Goldenberg, S Larry, Guy Nir, and Septimiu E Salcudean. 2019. “A New Era: Artificial Intelligence and Machine Learning in Prostate Cancer.” *Nature Reviews Urology* 16 (7): 391–403.

</div>

<div id="ref-han2004evidence" class="csl-entry">

<span class="nocase">Han, Jing-Dong J, Nicolas Bertin, Tong Hao, et al.</span> 2004. “Evidence for Dynamically Organized Modularity in the Yeast Protein–Protein Interaction Network.” *Nature* 430 (6995): 88–93.

</div>

<div id="ref-haralick1973textural" class="csl-entry">

Haralick, Robert M, Karthikeyan Shanmugam, and Its’ Hak Dinstein. 1973. “Textural Features for Image Classification.” *IEEE Transactions on Systems, Man, and Cybernetics* SMC-3 (6): 610–21.

</div>

<div id="ref-kryvenko2016prostate" class="csl-entry">

Kryvenko, Oleksandr N, and Jonathan I Epstein. 2016. “<span class="nocase">Prostate cancer grading: a decade after the 2005 modified Gleason grading system</span>.” *Archives of Pathology & Laboratory Medicine* 140 (10): 1140–52.

</div>

<div id="ref-kumar2014robbins" class="csl-entry">

Kumar, Vinay, Abul K Abbas, Nelson Fausto, and Jon C Aster. 2014. *Robbins and Cotran Pathologic Basis of Disease, Professional Edition e-Book*. Elsevier Health Sciences.

</div>

<div id="ref-van20202019" class="csl-entry">

<span class="nocase">Leenders, Geert JLH van, Theodorus H van der Kwast, David J Grignon, et al.</span> 2020. “The 2019 International Society of Urological Pathology (ISUP) Consensus Conference on Grading of Prostatic Carcinoma.” *The American Journal of Surgical Pathology*.

</div>

<div id="ref-monaco2008detection" class="csl-entry">

Monaco, J, J Tomaszewski, M Feldman, et al. 2008. “Detection of Prostate Cancer from Whole-Mount Histology Images Using Markov Random Fields.” *Workshop on Microscopic Image Analysis with Applications in Biology (in Conjunction with MICCAI)*.

</div>

<div id="ref-naik2008automated" class="csl-entry">

Naik, Shivang, Scott Doyle, Shannon Agner, Anant Madabhushi, Michael Feldman, and John Tomaszewski. 2008. “Automated Gland and Nuclei Segmentation for Grading of Prostate and Breast Cancer Histopathology.” *2008 5th IEEE International Symposium on Biomedical Imaging: From Nano to Macro*, 284–87.

</div>

<div id="ref-naik2007gland" class="csl-entry">

Naik, Shivang, Scott Doyle, Michael Feldman, John Tomaszewski, and Anant Madabhushi. 2007. “<span class="nocase">Gland segmentation and computerized Gleason grading of prostate histology by integrating low-, high-level and domain specific information</span>.” *MIAAB Workshop*, 1–8.

</div>

<div id="ref-newman2018networks" class="csl-entry">

Newman, Mark. 2018. *Networks*. Oxford University Press.

</div>

<div id="ref-nguyen2012prostate" class="csl-entry">

Nguyen, Kien, Bikash Sabata, and Anil K Jain. 2012. “<span class="nocase">Prostate cancer grading: Gland segmentation and structural features</span>.” *Pattern Recognition Letters* 33 (7): 951–61.

</div>

<div id="ref-Nguyen2012Structure" class="csl-entry">

Nguyen, Kien, Anindya Sarkar, and Anil K Jain. 2012. “Structure and Context in Prostatic Gland Segmentation and Classification.” *International Conference on Medical Image Computing and Computer-Assisted Intervention*, 115–23.

</div>

<div id="ref-nir2018automatic" class="csl-entry">

<span class="nocase">Nir, Guy, Soheil Hor, Davood Karimi, et al.</span> 2018. “Automatic Grading of Prostate Cancer in Digitized Histopathology Images: Learning from Multiple Experts.” *Medical Image Analysis* 50: 167–80.

</div>

<div id="ref-patel2009optimal" class="csl-entry">

Patel, Amit R, and J Stephen Jones. 2009. “Optimal Biopsy Strategies for the Diagnosis and Staging of Prostate Cancer.” *Current Opinion in Urology* 19 (3): 232–37.

</div>

<div id="ref-sarkar2016review" class="csl-entry">

Sarkar, Saradwata, and Sudipta Das. 2016. “A Review of Imaging Methods for Prostate Cancer Detection: Supplementary Issue: Image and Video Acquisition and Processing for Clinical Applications.” *Biomedical Engineering and Computational Biology* 7: BECB–S34255.

</div>

<div id="ref-schroder2009screening" class="csl-entry">

<span class="nocase">Schröder, Fritz H, Jonas Hugosson, Monique J Roobol, et al.</span> 2009. “Screening and Prostate-Cancer Mortality in a Randomized European Study.” *New England Journal of Medicine* 360 (13): 1320–28.

</div>

<div id="ref-singh2017gland" class="csl-entry">

Singh, Malay, Emarene Mationg Kalaw, Danilo Medina Giron, Kian-Tai Chong, Chew Lim Tan, and Hwee Kuan Lee. 2017. “Gland Segmentation in Prostate Histopathological Images.” *Journal of Medical Imaging* 4 (2): 027501.

</div>

<div id="ref-sznajd2000opinion" class="csl-entry">

Sznajd-Weron, Katarzyna, and Jozef Sznajd. 2000. “Opinion Evolution in Closed Community.” *International Journal of Modern Physics C* 11 (06): 1157–65.

</div>

<div id="ref-vella2018mtgo" class="csl-entry">

Vella, Danila, Simone Marini, Francesca Vitali, Dario Di Silvestre, Giancarlo Mauri, and Riccardo Bellazzi. 2018. “MTGO: PPI Network Analysis via Topological and Functional Module Identification.” *Scientific Reports* 8 (1): 1–13.

</div>

<div id="ref-waxman1988routing" class="csl-entry">

Waxman, Bernard M. 1988. “Routing of Multipoint Connections.” *IEEE Journal on Selected Areas in Communications* 6 (9): 1617–22.

</div>

<div id="ref-wong2016global" class="csl-entry">

Wong, Martin CS, William B Goggins, Harry HX Wang, et al. 2016. “Global Incidence and Mortality for Prostate Cancer: Analysis of Temporal Patterns and Trends in 36 Countries.” *European Urology* 70 (5): 862–74.

</div>

</div>
