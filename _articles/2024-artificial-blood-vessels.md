---
published: true
layout: article
title: Artificial vascular image generation using blood vessel texture maps
authors:
  - name: Adriano dos Reis Carvalho
  - name: Matheus Viana da Silva
  - name: Cesar H. Comin
    orcid: 0000-0003-1207-4982
date: 2024-12-01                   
year: 2024                   
journal: Computers in Biology and Medicine
volume: 183
issue: 1
pages: 109226
doi: 10.1016/j.compbiomed.2024.109226
pdf_url: /assets/pdfs/2024-artificial-blood-vessels.pdf
abstract: |
  Background: Current methods for identifying blood vessels in digital images typically involve training neural networks on pixel-wise annotated data. However, manually outlining whole vessel trees in images tends to be very costly. One approach for reducing the amount of manual annotation is to pre-train networks on artificially generated vessel images. Recent pre-training approaches focus on generating proper artificial geometries for the vessels, while the appearance of the vessels is defined using general statistics of the real samples or generative networks requiring an additional training procedure to be defined. In contrast, we propose a methodology for generating blood vessels with realistic textures extracted directly from manually annotated vessel segments from real samples. The method allows the generation of artificial images having blood vessels with similar geometry and texture to the real samples using only a handful of manually annotated vessels. Methods: The first step of the method is the manual annotation of the borders of a small vessel segment, which takes only a few seconds. The annotation is then used for creating a reference image containing the texture of the vessel, called a texture map. A procedure is then defined to allow texture maps to be placed on top of any smooth curve using a piecewise linear transformation. Artificial images are then created by generating a set of vessel geometries using Bézier curves and assigning vessel texture maps to the curves. Results: The method is validated on a fluorescence microscopy (CORTEX) and a fundus photography (DRIVE) dataset. We show that manually annotating only 0.03% of the vessels in the CORTEX dataset allows pre-training a network to reach, on average, a Dice score of 0.87$`\pm`$<!-- -->0.02, which is close to the baseline score of 0.92 obtained when all vessels of the training split of the dataset are annotated. For the DRIVE dataset, on average, a Dice score of 0.74$`\pm`$<!-- -->0.02 is obtained by annotating only 0.29% of the vessels, which is also close to the baseline Dice score of 0.81 obtained when all vessels are annotated. Conclusion: The proposed method can be used for disentangling the geometry and texture of blood vessels, which allows a significant improvement of network pre-training performance when compared to other pre-training methods commonly used in the literature.
keywords:
  - Artificial blood vessels
  - Blood vessel segmentation
  - Texture mapping
mathjax: true         
bibtex: |
  @article{dos2024artificial,
    title={Artificial vascular image generation using blood vessel texture maps},
    author={dos Reis Carvalho, Adriano and da Silva, Matheus Viana and Comin, Cesar H},
    journal={Computers in Biology and Medicine},
    volume={183},
    pages={109226},
    year={2024},
    publisher={Elsevier}
  }
---

# Introduction

Generating accurate segmentations of blood vessels from digital images is an important task for diagnosis (<a href="#ref-mookiah2021review">Mookiah et al. 2021</a>; <a href="#ref-eladawi2018early">Eladawi et al. 2018</a>; <a href="#ref-sangeethaa2018intelligent">Sangeethaa and Uma Maheswari 2018</a>; <a href="#ref-li2021blood">Li et al. 2021</a>; <a href="#ref-almotiri2018multi">Almotiri et al. 2018</a>; <a href="#ref-nair2020blood">Nair and Muthuvel 2020</a>) as well as for developing precise measurements to support novel studies regarding the role of blood vessels on disease evolution (<a href="#ref-roda2021blood">Roda et al. 2021</a>; <a href="#ref-ouellette2020vascular"><span class="nocase">Ouellette et al.</span> 2020</a>; <a href="#ref-wong2019blood">Wong et al. 2019</a>; <a href="#ref-dolati2015pre">Dolati et al. 2015</a>). Current state-of-the-art approaches for segmentation involve training neural networks on manually annotated samples (<a href="#ref-mookiah2021review">Mookiah et al. 2021</a>). The samples used for training must represent well the distribution of the whole dataset so that the neural network can generalize to unseen samples. An important difficulty with such an approach is that the pixel-wise annotation of blood vessels is very costly.

One common approach to reduce the amount of samples that need to be manually annotated is to pre-train the network on artificially generated blood vessels (<a href="#ref-mou2021cs2"><span class="nocase">Mou et al.</span> 2021</a>; <a href="#ref-todorov2020machine"><span class="nocase">Todorov et al.</span> 2020</a>; <a href="#ref-chen2023all">Chen et al. 2023</a>; <a href="#ref-wijethilake2023deep">Wijethilake et al. 2023</a>). Given that blood vessels tend to have a tubular structure, a simple methodology for creating artificial vasculature is to randomly generate tubes with varying thickness and to blur the tubes with a smoothing filter (<a href="#ref-tetteh2020deepvesselnet">Tetteh et al. 2020</a>; <a href="#ref-sule2020effects">Sule et al. 2020</a>). More advanced techniques involve simulating the growth of vascular trees (<a href="#ref-schneider2012tissue">Schneider et al. 2012</a>) to generate images with accurate distribution of vessel geometries. Common to many approaches that have been developed is the focus on the *geometry* of the vessels, and not on the *appearance* of the vessels. One line of work that does focuses on appearance consists of different approaches for style transfer and image generation (<a href="#ref-tmenova2019cyclegan">Tmenova et al. 2019</a>; <a href="#ref-ma2019neural">Ma et al. 2019</a>; <a href="#ref-wu2022vessel">Wu et al. 2022</a>), but these techniques do not allow precise control of the generated vessels regarding properties such as vessel density, caliber, tortuosity, and local intensity changes. Furthermore, they require the development of an additional neural network training procedure for the generative model, which in practice requires hyperparameter tuning and can have poor performance for samples at the tail of the data distribution.

In this work, we develop a methodology for extracting the texture of manually annotated blood vessels from real samples and generating artificial images using the extracted textures. The method only requires the annotation of a few vessel segments of a dataset. Given a dataset containing blood vessel images with different appearances (example images are shown in Figure <a href="#f:motivation">1</a>), a common practice would be to annotate all blood vessels in some of the samples to train a segmentation algorithm. The developed method involves the annotation of one or a few vessel segments from some of the images, shown in green in Figure <a href="#f:motivation">1</a>. The annotations are used for generating artificial images containing blood vessels with the same texture as the annotated segments, which can then be used for pre-training neural networks. This allows the network to be adjusted to as many artificial images as desired, each image containing a rich set of blood vessel geometries and appearances. Notably, the annotations can be done in seconds, or a few minutes if a more diverse dataset is desired. The neural network may then be refined on a few fully annotated real samples.

<figure id="f:motivation">
[Image omitted for text-only version]
<figcaption>Three fluorescence microscopy images containing blood vessels with different appearances. Green lines show example annotations of blood vessel segments.</figcaption>
</figure>

The methodology involves creating standardized representations of the appearance of the vessels by mapping the annotated segments into a rectilinear grid. The result is called a *texture map* since it shares some similarities with texture maps in computer graphics (<a href="#ref-hughes2013computer">Hughes et al. 2013</a>). A procedure is then developed for associating a given texture map to a randomly generated Bézier curve. This is done by defining two corresponding sets of points, one for the texture map and one for the Bézier curve. A Delaunay triangulation is generated for each set of points, and linear transformations are estimated between pairs of corresponding triangles of the two sets of points. This procedure allows the generation of artificial vessels with varying geometries and realistic textures that can be systematically controlled.

We show that artificial images generated using the methodology are effective for pre-training neural networks. On a fluorescence microscopy dataset, images generated from only 5 annotated vessel segments, which corresponds to annotating 0.03% of the vessels in the dataset, lead to an average Dice score of 0.87$`\pm`$<!-- -->0.02, which is close to the baseline value of 0.92 obtained when training using the fully annotated dataset. For a fundus photography dataset, an average Dice score of 0.74$`\pm`$<!-- -->0.02 is achieved by annotating 10 maps (0.29% of the vessels), which is also close to the baseline value of 0.81 obtained when all vessels are annotated. While the manual annotation of the entire datasets can take weeks or months, 10 vessel segments can be annotated in under 5 minutes.

The code of the methodology and the obtained results are publicly available online[^1].

# Related works

In (<a href="#ref-galarreta2013three">Galarreta-Valverde et al. 2013</a>) the authors proposed a methodology that extended the traditional L-systems grammar (<a href="#ref-lindenmayer1968mathematical">Lindenmayer 1968</a>), which was used for synthesizing angiographic images simulating computed tomography and magnetic resonance images. The generation of synthetic vessels was aimed at assisting in validating segmentation algorithms and to be used in virtual surgeries.

Vessel tree simulators were used to compare artificially created data with real vascular trees (<a href="#ref-kocinski20123d">Kociński et al. 2012</a>). The authors approached the comparison using the texture properties of the trees’ geometry (not the vessels’ surface). They showed that the models had important correlations with different types of confocal brain tissue images. Vascular tree parameter changes were also observed between healthy and tumorous tissue.

A few recent works (<a href="#ref-mou2021cs2"><span class="nocase">Mou et al.</span> 2021</a>; <a href="#ref-todorov2020machine"><span class="nocase">Todorov et al.</span> 2020</a>; <a href="#ref-wijethilake2023deep">Wijethilake et al. 2023</a>) used biologically simulated vascular trees for training neural networks using the method of Schneider et al. (<a href="#ref-schneider2012tissue">Schneider et al. 2012</a>). The method consists of an iterative generation of arterial trees through two main processes: constructive (growth) and destructive (degeneration). Each vessel segment is modeled as a cylindrical tube and tissue metabolism demands drive the angiogenic process. For instance, in (<a href="#ref-tetteh2020deepvesselnet">Tetteh et al. 2020</a>) the model was used to simulate magnetic resonance angiography images, which were then used to train different neural network architectures. It was shown that the synthetic data greatly contributed to network pre-training.

The aforementioned works focused mostly on generating plausible geometries for the vessels. Usually, the texture is added using simple procedures such as blurring and Gaussian or Poisson noise. Another line of work considers the generation of vasculature with plausible geometry and appearance using generative adversarial networks (<a href="#ref-tmenova2019cyclegan">Tmenova et al. 2019</a>; <a href="#ref-ma2019neural">Ma et al. 2019</a>; <a href="#ref-wu2022vessel">Wu et al. 2022</a>; <a href="#ref-popescu2021retinal">Popescu et al. 2021</a>; <a href="#ref-zhao2018synthesizing">Zhao et al. 2018</a>). But the generated vessels cannot be tightly controlled, for instance, one cannot easily generate samples containing blood vessels with large tortuosity or with low-contrast segments (i.e., discontinuities). In addition, a new training procedure needs to be applied when there is a distribution shift in the data, such as when changing microscope parameters. Our model allows the precise control of the geometry of the vessels as well as the selection of specific texture patterns for the generated images.

# Methodology

The method can be divided into five main steps. They are: (a) manual vessel delineation; (b) creation of vessel models; (c) generation of vessel texture maps, (d) transformation of the maps to follow randomly generated curves, and (e) insertion of the transformed maps into an artificially generated background. In the following sections, we describe each of the steps[^2].

## Manual vessel delineation

The first step consists of manually annotating a few blood vessel segments. Segments are annotated by delineating their borders, as shown in Figure <a href="#f:motivation">1</a>. There is no need to annotate the whole segment, only a portion of the segment is enough to obtain its texture pattern. A simple graphical user interface was created for this task, which stores the pixel coordinates of the two delineated borders as a .json file, but the segments can be annotated on any software, provided a file containing the coordinates is generated. The sets of points from the annotated borders are represented as $`\tilde{P}_{b1}`$ and $`\tilde{P}_{b2}`$.

Vessels annotated from different images generate a dataset of vessel segments. The method can work with a single annotated segment, but additional segments can increase the diversity of the textures of the vessels on the artificial images. Thus, it is useful to annotate vessels from different images and having distinct appearances. In our experiments, five segments usually led to a good segmentation performance.

## Generation of vessel models

The manual delineations $`\tilde{P}_{b1}`$ and $`\tilde{P}_{b2}`$ are used for obtaining additional information about the vessels such as their medial lines and normal vectors at each border position. The set of all such information for a given vessel segment is henceforth called the *vessel model* of the segment. An example of a vessel model is shown in Figure <a href="#f:vessel_model">2</a>. The procedure used to generate a vessel model is explained next.

<figure id="f:vessel_model" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Illustration of a vessel model. A precise delineation of the vessel border is represented in green, the vessel’s medial axis is represented in red, and normal vectors to each curve are represented as orange lines.</figcaption>
</figure>

Depending on the annotation tool used, the points in $`\tilde{P}_{b1}`$ and $`\tilde{P}_{b2}`$ might not be equally spaced, and the annotations might be noisy due to the precision required to trace vessel borders. Thus, for creating the vessel model, the first step is to interpolate the manual delineation of the vessel borders, which is done using a two-stage procedure. A linear interpolation of the points is first performed to uniformly sample the manual annotations. Then, a cubic spline interpolation is performed on the result of the first interpolation. This two-stage interpolation is useful to make sure that the interpolated curves pass close to all manually marked points. In our implementation, the functions *splprep* and *splev* of the SciPy Python package were used.

Both interpolations have a parameter $`\delta`$ setting the distance between interpolated points, which equivalently sets the total number of points used in the interpolated curves. A value of $`\delta=1`$ pixel was used in all experiments since smaller values lead to redundant information due to the finite size of the image grid. The result of the interpolation process are two sets of points $`P_{b1}`$ and $`P_{b2}`$ representing high-resolution smooth curves describing the borders of the vessel segment.

For each point in $`P_{b1}`$ and $`P_{b2}`$, a normal vector to the curve at the position of the point is calculated using the spline representation of the curve. The two respective sets of normal vectors are represented as $`N_{b1}`$ and $`N_{b2}`$. Figure <a href="#f:vessel_model">2</a> shows in green the interpolated curves and in orange the normal vectors at each point.

Next, $`P_{b1}`$ and $`P_{b2}`$ are used to calculate the medial axis of the blood vessel, which is obtained using a Voronoi tessellation of the border points (<a href="#ref-tagliasacchi20163d">Tagliasacchi et al. 2016</a>). In our implementation, the Voronoi tessellation is calculated using the *Voronoi* class from the SciPy Python package. Figure <a href="#f:voronoi">3</a> shows an example of Voronoi tessellation of a vessel segment. The tessellation provides the medial axis with sub-pixel accuracy, which makes the following procedures more accurate. However, the points defining the medial axis are not equally spaced. Thus, the same two-stage interpolation applied to the manual delineations is applied to the medial axis using the same value of $`\delta`$. Respective normal vectors are also similarly calculated. The resulting sets of medial axis points and respective normal vectors are represented, respectively, as $`P_m`$ and $`N_m`$. They are illustrated in Figure <a href="#f:vessel_model">2</a>.

<figure id="f:voronoi" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Example of Voronoi tessellation of vessel borders. The points used for the calculation are shown in black, they were obtained using <span class="math inline"><em>δ</em> = 2</span> in the interpolation for a better visualization of the tessellation. Yellow lines represent Voronoi cells. Red lines and points represent the calculated medial axis.</figcaption>
</figure>

The three generated curves (two borders and one medial axis) represented by $`P_{b1}`$, $`P_{b2}`$ and $`P_m`$ as well as the normal vectors $`N_{b1}`$, $`N_{b2}`$ and $`N_m`$ are stored and define the vessel model of the segment.

## Vessel texture map creation

Vessel models are used for generating vessel texture maps, which are images containing only the texture of the vessels. To do so, a coordinate system following the model geometry is defined. The medial axis defines one coordinate, while perpendicular lines to the medial axis define the second coordinate. The intensities of the vessel are then mapped into a new rectilinear grid. Figure <a href="#f:extraction_formulation_model">4</a> shows an illustration of the procedure. However, some important details need to be taken into account when generating the coordinates.

<figure id="f:extraction_formulation_model" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Creation of a texture map using the coordinate system of the vessel. (a) The medial axis (red) defines one coordinate, and perpendicular lines to the medial axis define a second coordinate (yellow and blue). (b) The intensities of the vessel are mapped to a rectilinear grid. The central row of the resulting image always corresponds to the medial axis. Image columns correspond to the perpendicular lines in (a). In this illustration, a coarse grid of points was used for visual clarity. The distances between the points used in the actual implementation are all equal to one pixel.</figcaption>
</figure>

First, a set of sampling lines perpendicular to the medial axis are created. Figure <a href="#f:extraction_formulation_model">4</a>(a) shows in yellow and blue examples of perpendicular lines. The normal vectors of the medial axis are used for defining a first guess for the direction of the lines. However, the normal of an axis point and the normals of corresponding border points might not be aligned. For instance, on high-curvature segments or when the border at one side significantly changes, the normal vector at the medial axis might lead to very distinct points at the opposing borders of the vessel.

To obtain a precise cross-section of the vessel at a medial axis point $`\mathbf{p}_m`$, we find the line that best aligns with the normal vector $`\mathbf{n}_m`$ at the point as well as the normals of the corresponding points at the border of the segment. The procedure used for finding the best-fitting line is described in Algorithm <a href="#a:cross_paths">5</a> and is illustrated in Figure <a href="#f:perpendicular_lines_training">6</a>. The algorithm receives as input the medial axis point $`\mathbf{p}_m`$ with respective normal $`\mathbf{n}_m`$ and the complete sets of border points $`P_{b1}`$ and $`P_{b2}`$ with their respective normals $`N_{b1}`$ and $`N_{b2}`$. A set of angles $`\Theta`$ is defined in the range $`[-45^\circ,45^\circ]`$ with steps of $`\Delta\theta=3^\circ`$ (line <a href="#acp_angles">\[acp_angles\]</a> of Algorithm <a href="#a:cross_paths">5</a>).

<figure id="a:cross_paths" data-latex-placement="!h">
<div class="algorithm">
<div class="algorithmic">

</div>
</div>
<figcaption>Calculation of the optimal cross-section angle</figcaption>
</figure>

<figure id="f:perpendicular_lines_training" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Example of cross-section angle identification for the medial axis point indicated in red. Candidate directions are shown as dashed lines. The blue line leads to the best alignment between the medial axis normal (black line) and the normals at border points (purple arrows), and thus is chosen as the cross-section direction.</figcaption>
</figure>

The medial point $`\mathbf{p}_m`$ and respective vector $`\mathbf{n}_m`$ define an initial candidate line for a cross-section of the vessel at point $`\mathbf{p}_m`$ (black line in Figure <a href="#f:perpendicular_lines_training">6</a>). For each angle $`\theta`$ in $`\Theta`$, the vector $`\mathbf{n}_m`$ is rotated by $`\theta`$ (line <a href="#acp_rot">\[acp_rot\]</a> of Algorithm <a href="#a:cross_paths">5</a>), thus defining a new candidate line for the cross-section of the vessel. Candidate lines are shown as dashed lines in Figure <a href="#f:perpendicular_lines_training">6</a>. The function *get_closest*($`\mathbf{p}_m`$, $`\mathbf{\tilde{n}}_m`$, $`P_{b1}`$) (line <a href="#acp_closest">\[acp_closest\]</a> of Algorithm <a href="#a:cross_paths">5</a>) returns the point $`\mathbf{p}_{b1}`$ in $`P_{b1}`$ with the smallest point-to-line distance to the line defined by $`\mathbf{p}_m`$ and $`\mathbf{\tilde{n}}_m`$. The same calculation is done for the other border, defining point $`\mathbf{p}_{b2}`$. The respective normals are also obtained (lines <a href="#acp_normal1">\[acp_normal1\]</a> and <a href="#acp_normal2">\[acp_normal2\]</a> of Algorithm <a href="#a:cross_paths">5</a>). Points $`\mathbf{p}_{b1}`$ and $`\mathbf{p}_{b2}`$ of each candidate line are shown as small brown dots in Figure <a href="#f:perpendicular_lines_training">6</a> and respective normals are shown as purple arrows.

The alignment between vectors $`\mathbf{\tilde{n}}_m`$, $`\mathbf{n}_m`$, $`\mathbf{n}_{b1}`$ and $`\mathbf{n}_{b2}`$ (line <a href="#acp_align">\[acp_align\]</a> of Algorithm <a href="#a:cross_paths">5</a>) is then calculated as

``` math
\begin{equation}
    e = 2\mathbf{n}_m\cdot\mathbf{\tilde{n}}_m + \mathbf{n}_{b1}\cdot\mathbf{\tilde{n}}_m + \mathbf{n}_{b2}\cdot\mathbf{\tilde{n}}_m,
\end{equation}
```
where $`\mathbf{x}\cdot \mathbf{y}`$ represents the dot product between $`\mathbf{x}`$ and $`\mathbf{y}`$. The equation represents the degree of alignment between the candidate line with direction represented by $`\mathbf{\tilde{n}}_m`$ and the normals of the medial and border points. A weight of 2 is used on $`\mathbf{n}_m\cdot\mathbf{\tilde{n}}_m`$ to slightly favor the alignment with the medial axis. The angle $`\theta_o`$ with the largest value of $`e`$ (line <a href="#acp_max">\[acp_max\]</a> of Algorithm <a href="#a:cross_paths">5</a>) defines the direction of the cross-section at point $`\mathbf{p}_m`$. This direction is illustrated as a blue line in Figure <a href="#f:perpendicular_lines_training">6</a>.

The identified cross-sectional lines together with the medial axis define a 2D coordinate system based on the vessel’s geometry. The medial axis represents a coordinate along the blood vessel, while the perpendicular lines represent a coordinate along cross-sections of the vessel. Sampling points are then created along each perpendicular line. Given a parameter $`r`$ setting the range of the perpendicular lines, that is, the distance between each endpoint of a line and the medial axis, the sampling points $`S=[-r, -r+\delta, \dots,r-\delta,r]`$ are created. The parameter $`\delta`$ sets the distance between points and is the same used when interpolating the border and medial axis points. The sampling points are then oriented according to the perpendicular lines found using Algorithm <a href="#a:cross_paths">5</a>. Examples of sampling points are shown as yellow dots in Figure <a href="#f:extraction_formulation_model">4</a>(a).

Each cross-sectional line can be mapped to a column of an output image. The number of sampling points in $`S`$ defines the number of rows of the image. An illustration is shown in Figure <a href="#f:extraction_formulation_model">4</a>(b). Thus, for each pixel of the output image, the corresponding intensity of the original annotated segment can be calculated using bilinear interpolation. In our implementation, this is done using the *map_coordinates* function of the SciPy Python package. The output image defines the vessel texture map $`M`$.

The main parameter of the map creation procedure is the range $`r`$ used for generating the cross-section sampling points. If $`r`$ is too small, the cross-section lines might not capture the whole vessel. The transformation also needs to capture some portion of the background of the image for the artificial image generation, but if $`r`$ is too large, other vessels might be included in the texture map, which will lead to artifacts. A good compromise is to use the largest diameter of the vessel segment together with an additional range of around 10 pixels to capture the background.

The sets of border points $`P_{b1}`$ and $`P_{b2}`$ are also projected onto the map. They are used for generating a segmentation mask for identifying pixels belonging to the vessel and to the background. Notice that it is not necessary to project the medial axis coordinates since it will always correspond to the middle row of the map.

## Association of vessel textures to arbitrary curves

The texture maps can be used for generating textures for arbitrary curves. In our methodology we focused on Bézier curves, which are simple to implement and can faithfully represent the curvilinear structure of a vessel. A Bézier curve is defined by the initial and final points, $`\mathbf{p}_i`$ and $`\mathbf{p}_f`$, as well as a set $`P_c`$ of $`q`$ intermediate control points that control the geometry and the smoothness of the curve. The curve is created using Algorithm <a href="#a:bezier">7</a>. $`q`$ linearly spaced values are created in the range $`[0.2, 0.8]`$ (line <a href="#ab_ts">\[ab_ts\]</a> of Algorithm <a href="#a:bezier">7</a>). These values are used for defining $`q`$ respective points along a straight line between $`\mathbf{p}_i`$ and $`\mathbf{p}_f`$ (line <a href="#ab_pl">\[ab_pl\]</a> of Algorithm <a href="#a:bezier">7</a>). Each point is then displaced along a normal vector $`\mathbf{n}_l`$ to the straight line (line <a href="#ab_disp">\[ab_disp\]</a> of Algorithm <a href="#a:bezier">7</a>). The normal vector is a unit vector having a dot product with vector $`\mathbf{p}_f-\mathbf{p}_i`$ equal to zero. The translation amount is randomly drawn with uniform probability in the range $`[-d_{m},d_{m}]`$, where $`d_{m}`$ is a parameter of the method. Lower values of $`d_{m}`$ lead to more straight curves.

<figure id="a:bezier" data-latex-placement="!h">
<div class="algorithm">

</div>
<figcaption>Bézier curve generation</figcaption>
</figure>

A Bézier curve with order $`q+1`$ can then be defined using control points $`\mathbf{p}_i`$, $`\mathbf{p}_f`$ and $`P_c`$. The application of the curve for transforming the maps requires a set of sampling points along the curve. Thus, for any given curve, $`n_p=100`$ uniformly spaced points between the initial and final points are created along the curve. The generated points of the curve are represented as $`P_r`$.

Given a vessel map $`M`$ and the points $`P_r`$, a procedure is defined to transform the map to follow the geometry of the curve. To do so, we first create two new curves by shifting points $`P_r`$ to the left-hand and right-hand sides of the original curve. Figure <a href="#f:trace_13">8</a> shows an example of the resulting geometry. The curves are created by finding all points with a point-to-line distance of $`d_b`$ to the original Bézier curve, where $`d_b`$ is given by half the number of rows of the vessel map. Then, all points having the endpoints of the Bézier curve as the closest points to them are removed. This defines two curves, which are then linearly interpolated to have the same number of points $`n_p`$ as $`P_r`$. The set of the three generated curves is called the *vessel geometry*. The points defining the three curves, and respective vessel geometry, are all represented by the set $`P_g`$.

<figure id="f:trace_13" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Example of a generated vessel geometry. The central curve is generated using Algorithm <a href="#a:bezier">7</a>. The green curves are translated versions of the central curve.</figcaption>
</figure>

Having obtained points $`P_g`$, a respective set of points $`P_v`$ needs to be defined for the vessel map with a one-to-one correspondence with points in $`P_g`$ so that the intensities can be transformed from the map to the curve. Before defining $`P_v`$, it is important to observe that the length of the Bézier curve, measured as the path length along the curve, and the length of the vessel map, measured as the number of columns of the map, are usually distinct. Thus, the vessel map is expanded to have the same length as the Bézier curve. This is done by repeating the map column-wise the number of times required for the lengths to be compatible.

Representing as $`c`$ the number of columns of the vessel map, $`n_p`$ sampling points are defined at row 0 of the map and at the column positions $`[0, \Delta_s, 2\Delta_s,\dots,(n_p-1)\Delta_s]`$, where $`\Delta_s=(c-1)/(n_p-1)`$. That is, $`n_p`$ points are created along the first row of the map with equal spacing between them from column positions 0 to $`c-1`$ (the last column of the map). The same procedure is repeated for the middle and last row of the map. Set $`P_v`$ is composed of the created points. Figure <a href="#f:delaunay_with_artefacts_zoom">9</a>(a) shows in red, yellow, and blue the points in $`P_v`$.

Having points $`P_g`$ and $`P_v`$, with a one-to-one correspondence between them, a Delaunay triangulation of each set of points is calculated. This defines two sets of triangles, also with a one-to-one correspondence. For each pair of triangles (one from the map and the other from the curve), an affine transform is estimated and the intensities are mapped using bilinear interpolation. In our implementation, this procedure is applied using the class *PiecewiseAffineTransform* and the *warp* function of the scikit-image Python package. Figure <a href="#f:delaunay_with_artefacts_zoom">9</a> shows an example triangulation of the original map and the Bézier curve together with the transformed image.

<figure id="f:delaunay_with_artefacts_zoom">
[Image omitted for text-only version]
<figcaption>Example of map transformation. (a) Original map, with points <span class="math inline"><em>P</em><sub><em>v</em></sub></span> represented by circles. (b) Transformed map, with points <span class="math inline"><em>P</em><sub><em>g</em></sub></span> also represented by circles. Orange lines represent the Delaunay triangulation of the points. The triangles highlighted in green in both images are a transformed version of each other.</figcaption>
</figure>

The Delaunay triangulation might present some artifacts because points that are on different parts of the border might generate triangles that are outside the curve. These triangles can be seen on the right side of Figure <a href="#f:delaunay_with_artefacts_zoom">9</a>(b). These artifacts were removed by zeroing all pixels outside the vessel geometry.

Besides the vessel texture, it is also necessary to map the location of vessel pixels so that a corresponding binary image can be created and used as a target for segmentation algorithms. This is done by creating a binary vessel image from the map image. This is simple since we have the position of the vessel borders on the map. The binary image is then transformed using the same transformation applied to the grayscale image, but using a nearest neighbor interpolation. Figure <a href="#f:transformation">10</a> shows an example of a transformed map and transformed binary vessel.

<figure id="f:transformation" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Example of a transformed map and binary vessel mask. Notice that the transformed map includes the vessel and, by design, a portion of the background of the original image which the map was extracted from.</figcaption>
</figure>

## Generation of artificial images using the transformed vessel maps

Having a collection of transformed maps, it is possible to insert them into an image to generate a realistic vasculature tissue image. One possibility is to generate a vascular tree and place the maps on each vessel segment of the tree. However, blood vessel identification is usually implemented as a semantic segmentation task. This is because digital samples usually contain only a portion of the vasculature of a tissue. In addition, for most biological tissues, the vasculature does not have a standard geometry at the typical scales captured by most imaging modalities. Therefore, we consider that the global geometry of the generated artificial images is not particularly relevant for vessel segmentation methods. Local properties such as the curvature, caliber, border definition, and texture are likely more relevant. This is particularly true for Convolutional Neural Networks, which are known to be more biased towards local features (<a href="#ref-islam2021shape">Islam et al. 2021</a>; <a href="#ref-geirhos2018imagenet">Geirhos et al. 2018</a>). Thus, we generate artificial images by randomly placing texture maps into an image, without following a global vascular structure.

The only remaining step is to generate an artificial background image $`B`$ to insert the maps into. This can be done in several ways. For instance, it is possible to generate a random image with values drawn from a normal distribution. One can also annotate a few background regions from a sample and replicate these regions multiple times to generate a full image.

Note that since the transformed maps include some portion of the background of the samples they were extracted from, the artificial background image does not need to be realistic. The main challenge of the segmentation algorithm will be to segment the vessel from the texture map background, and not from the artificial background. Thus, we do not consider the artificial background generation as part of our methodology. In the experiments, we used a simple methodology to create $`B`$, consisting of dividing a real sample into small windows and replacing windows containing blood vessels with windows that did not contain blood vessels. It is important to notice that the size of $`B`$ defines the size of the final artificial blood vessel image.

To generate images without a sharp change in intensity between the artificial background and the map background, before insertion onto the background the maps are normalized as

``` math
\begin{equation}
    \tilde{M} = M - \mu_m + \mu_b,
\end{equation}
```
where $`\mu_m`$ and $`\mu_b`$ are the average intensity of, respectively, the map’s background and the artificial background image. A normalization using the standard deviation of the intensities may also be used in case the intensities of the vessel maps have a larger standard deviation than the artificial background.

The procedure used for generating an artificial image is summarized in Algorithm <a href="#a:generation">11</a>. The algorithm receives as input a database of texture maps $`\mathcal{M}`$, the number of maps $`m`$ to use, an initial background image $`B`$, the number of vessels $`n_v`$ to insert, the minimum length $`s_l`$ of the artificial vessels and the $`q`$ and $`d_m`$ parameters for the Bézier curve generation. First, a subset $`\tilde{\mathcal{M}}`$ of $`m`$ texture maps is randomly drawn from $`\mathcal{M}`$ (line <a href="#gen_sub">\[gen_sub\]</a> of Algorithm <a href="#a:generation">11</a>). A value $`s_m`$ setting the maximum length of the vessel geometries is then calculated (line <a href="#gen_s">\[gen_s\]</a> of Algorithm <a href="#a:generation">11</a>). It is given by the number of rows or columns of the image, whichever is larger.

<figure id="a:generation" data-latex-placement="!h">
<div class="algorithm">

</div>
<figcaption>Artificial image generation</figcaption>
</figure>

For each vessel to be inserted into the image, the length $`d_e`$ of the vessel is drawn with uniform probability in the range $`[s_l,s_m]`$. The function *get_endpoints* (line <a href="#gen_ends">\[gen_ends\]</a> of Algorithm <a href="#a:generation">11</a>) randomly generates two points $`\mathbf{p}_i`$ and $`\mathbf{p}_f`$ with distance $`d_e`$ between them. Next, the geometry of the vessel $`P_g`$ is generated by the function *generate_geometry* using the methodology presented in Section <a href="#s:map_transf">3.4</a>. One texture map is then randomly drawn from the set of available maps. The map is transformed into the artificial geometry by the *transform* function using the method presented in Section <a href="#s:map_transf">3.4</a> and inserted into the image $`I`$.

The result is an artificial image containing the inserted blood vessel textures. Parameters $`n_v`$, $`s_l`$, $`q`$, and $`d_m`$ can be set according to the density and typical curvatures of the vessels from the real dataset. Parameter $`m`$ should be as large as possible since the diversity of the textures is increased. However, larger $`m`$ requires more annotated vessel segments.

# Results and Discussion

In this section, we present the results of the vessel model and texture map creation methods. We also show that the generated artificial images can be used to efficiently train neural networks. The experiments were conducted on an AMD Ryzen 5 5600G 3.90 GHz CPU with 16.0GiB RAM, except for the training and validation of neural networks, which were executed on an Intel I9 12900K 3.20 GHz CPU with an RTX 3090 GPU. Versions 1.10 and 0.21 of, respectively the SciPy and scikit-image Python packages were used.

## Datasets

Three datasets are used to show the potential of the methodology. The first dataset is composed of 50 confocal microscopy images of the mouse cortex. The images have a size of 1376 $`\times`$ 1104 pixels, with each pixel representing 0.908 $`\times`$ 0.908 $`\mu`$m. Details about the image acquisition procedure can be found in (<a href="#ref-freitas2022unbiased">Freitas-Andrade et al. 2022</a>). This dataset is challenging because the blood vessels do not have well-defined borders and some samples have low contrast. Figure <a href="#f:database_image_2">12</a> contains examples of images from the dataset. This dataset is henceforth called *CORTEX*.

<figure id="f:database_image_2" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Examples of images from the CORTEX dataset.</figcaption>
</figure>

We also use the DRIVE dataset (<a href="#ref-StaalDRIVE">Staal et al. 2004</a>), one of the most popular datasets for evaluating blood vessel segmentation algorithms. The dataset contains 20 training and 20 test color fundus photographs, each image having a size of 584 $`\times`$ 565 pixels. Following common practice in the literature, only the green channel of the images was used. The fundus images from the DRIVE dataset are markedly distinct from the microscopy images from the CORTEX dataset. Thick vessels tend to have better contrast with the background when compared to the CORTEX dataset, but the DRIVE images contain a larger number of very thin vessels, which tend to be difficult to segment.

As discussed above, one of the motivations for our method is to improve upon the common pre-training procedure of generating artificial vessels with trivial textures. Thus, we also compare our method with a technique used by recent works to simulate artificial vessels (<a href="#ref-mou2021cs2"><span class="nocase">Mou et al.</span> 2021</a>; <a href="#ref-todorov2020machine"><span class="nocase">Todorov et al.</span> 2020</a>; <a href="#ref-wijethilake2023deep">Wijethilake et al. 2023</a>), which we refer to as *TREE*. The technique consists of using the method of Schneider et al. (<a href="#ref-schneider2012tissue">Schneider et al. 2012</a>) to generate realistic artificial vascular trees. A Gaussian blur followed by Gaussian noise is then applied to the images. In (<a href="#ref-tetteh2020deepvesselnet">Tetteh et al. 2020</a>) a dataset containing 136 artificial samples was generated using the artificial tree method and made available online. We use this dataset in the experiments.

The CORTEX dataset is used in the following sections to provide examples of artificial images generated using the proposed methodology. All three datasets are used in Section <a href="#s:res_net">4.6</a> to quantify the potential of the method for pre-training neural networks.

## Vessel models

To measure the typical time spent on manual annotation of vessel segments, six annotators were asked to delineate 3 vessel segments in a given image from the CORTEX dataset, and we recorded the time taken for the annotation of the vessels. The results are shown in Table <a href="#a:mensuration">1</a>. Overall, the annotation of a vessel segment takes an average of 27 seconds, with the time varying according to the characteristics of the vessel. Vessels with larger dimensions or that are very tortuous tend to take more time to be annotated.

<table id="a:mensuration">
<caption>Vessel segment annotation time for six annotators and three vessel segments.</caption>
<thead>
<tr>
<th colspan="5" style="text-align: center;">Annotation time in seconds</th>
</tr>
</thead>
<tbody>
<tr>
<td style="text-align: left;"></td>
<td style="text-align: center;">Vessel one</td>
<td style="text-align: center;">Vessel two</td>
<td style="text-align: center;">Vessel three</td>
<td style="text-align: center;">Average</td>
</tr>
<tr>
<td style="text-align: left;">Annotator one</td>
<td style="text-align: center;">11.86</td>
<td style="text-align: center;">11.22</td>
<td style="text-align: center;">10.8</td>
<td style="text-align: center;">11.29</td>
</tr>
<tr>
<td style="text-align: left;">Annotator two</td>
<td style="text-align: center;">30</td>
<td style="text-align: center;">34</td>
<td style="text-align: center;">33</td>
<td style="text-align: center;">32.33</td>
</tr>
<tr>
<td style="text-align: left;">Annotator three</td>
<td style="text-align: center;">51.16</td>
<td style="text-align: center;">28.55</td>
<td style="text-align: center;">31</td>
<td style="text-align: center;">36.90</td>
</tr>
<tr>
<td style="text-align: left;">Annotator four</td>
<td style="text-align: center;">27.69</td>
<td style="text-align: center;">27.75</td>
<td style="text-align: center;">27.65</td>
<td style="text-align: center;">27.70</td>
</tr>
<tr>
<td style="text-align: left;">Annotator five</td>
<td style="text-align: center;">21.48</td>
<td style="text-align: center;">22.49</td>
<td style="text-align: center;">32.28</td>
<td style="text-align: center;">25.42</td>
</tr>
<tr>
<td style="text-align: left;">Annotator six</td>
<td style="text-align: center;">24.42</td>
<td style="text-align: center;">17.98</td>
<td style="text-align: center;">38.47</td>
<td style="text-align: center;">26.96</td>
</tr>
</tbody>
</table>

For the following experiments, a total of 343 vessel segments were annotated from the 50 images in the CORTEX dataset by a single annotator. On average, between 5 and 8 vessels were annotated for each image. The minimum and maximum caliber of the vessels were, respectively 4.1 and 25.59 pixels, with an overall average caliber of 8.65 pixels.

A vessel model was created for each annotated segment. Figure <a href="#f:models">13</a> shows examples of created models. It is important to have a rich set of vessel models. Thus, we annotated vessels with large variations of intensity, contrast, tortuosity, caliber, and noise amount.

<figure id="f:models" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Vessel models generated from manual annotations.</figcaption>
</figure>

## Vessel texture maps

The vessel models were used for generating respective vessel texture maps. The database of 343 vessel texture maps obtained is represented as $`\mathcal{M}`$. Figure <a href="#f:vessel_models">14</a> shows examples of vessel maps from $`\mathcal{M}`$. Each vessel map contains the intensities of the vessel segment and, by design, the background of the sample which the vessel was extracted from. The manual annotation indicating the borders of the vessel is also stored together with the texture map. Again, it is important to have maps containing vessels with varying characteristics since this allows the generation of a diverse set of artificial images.

<figure id="f:vessel_models" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Examples of vessel texture maps.</figcaption>
</figure>

## Vessel geometries

Vessel geometries were generated by randomly drawing the initial and final points $`\mathbf{p}_i`$ and $`\mathbf{p}_f`$ of the Bézier curves such that the Euclidean distance $`d_e`$ between the points was in the range $`[500,1376]`$ pixels. Figure <a href="#f:different_strokes">15</a> shows examples of geometries that can be generated. $`q=30`$ control points were used for the curve in Figure <a href="#f:different_strokes">15</a>(a), while $`q=2`$ was used for the curve in Figure <a href="#f:different_strokes">15</a>(b). The number of control points has a large impact on tortuosity. Figures <a href="#f:different_strokes">15</a>(c), <a href="#f:different_strokes">15</a>(d), and <a href="#f:different_strokes">15</a>(e) show curves with $`q=6`$ and different values of $`d_m`$, which controls the maximum distance between control points and a straight line between the initial and final points. Lower $`d_m`$ values lead to straighter vessels.

<figure id="f:different_strokes" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Examples of generated vessel geometries. Curves (a) and (b) have, respectively <span class="math inline"><em>q</em> = 30</span> and <span class="math inline"><em>q</em> = 2</span> control points. Curves (c), (d), and (e) were generated with <span class="math inline"><em>q</em> = 6</span> and, respectively, maximum control point displacements of <span class="math inline"><em>d</em><sub><em>m</em></sub> = 1</span>, <span class="math inline"><em>d</em><sub><em>m</em></sub> = 100</span>, and <span class="math inline"><em>d</em><sub><em>m</em></sub> = 250</span>.</figcaption>
</figure>

For the remainder of the experiments, we used $`q=6`$ and $`d_m=500`$ for generating the geometries.

## Artificial image generation

Several artificial images with realistic textures were generated using Algorithm <a href="#a:generation">11</a>. We considered different situations according to the number of annotations $`m`$ (and respective vessel maps) required for generating the images. We considered situations where only a single annotation is used ($`m=1`$), as well as situations where $`m=5`$, $`m=10`$, and $`m=160`$ annotations are used. The annotations used were randomly drawn from the $`\mathcal{M}`$ set of 343 annotations. The number of vessels $`n_v`$ inserted into each image was randomly drawn with uniform probability in the range $`[20,50]`$ and the minimum vessel length used was $`s_l=500`$.

The results are images containing $`n_v`$ artificial vessels. Figure <a href="#f:artificial_maps_created_from_1_map">16</a> shows examples of images generated from a single annotated vessel segment ($`m=1`$). In this case, all vessels in the image have the same texture but different geometries. Interestingly, these images can be used for investigating the performance of segmentation algorithms when only the geometry of the blood vessels changes throughout the images.

<figure id="f:artificial_maps_created_from_1_map" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Examples of images generated from a single annotated vessel segment.</figcaption>
</figure>

Figure <a href="#f:background_with_artificial_vessels">17</a> shows images generated from different numbers of annotated vessels. Figures <a href="#f:background_with_artificial_vessels">17</a>(a), <a href="#f:background_with_artificial_vessels">17</a>(b), <a href="#f:background_with_artificial_vessels">17</a>(c) and <a href="#f:background_with_artificial_vessels">17</a>(d) were generated from, respectively, $`m=1`$, $`m=5`$, $`m=10`$ and $`m=160`$ vessel annotations. The images show a large diversity of geometries and vessel textures, as well as different degrees of vessel density.

<figure id="f:background_with_artificial_vessels" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Examples of artificial images generated from (a) <span class="math inline"><em>m</em> = 1</span>, (b) <span class="math inline"><em>m</em> = 5</span>, (c) <span class="math inline"><em>m</em> = 10</span>, and (d) <span class="math inline"><em>m</em> = 160</span> vessel texture maps.</figcaption>
</figure>

It is possible to use additional data augmentation transforms to further diversify the images. For instance, the images can have the size, brightness, and contrast randomly changed. Gaussian noise can also be added to the images (an example is shown in Figure <a href="#f:img_noise_gauss">18</a>). Since there are many distinct approaches for data augmentation, our method is defined independently of any specific data augmentation pipeline.

<figure id="f:img_noise_gauss" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Example of Gaussian noise insertion in an artificial image. (a) Original image. (b) Image after noise inclusion.</figcaption>
</figure>

## Validation using neural networks

The artificial images were used to verify if the proposed methodology can assist in training neural networks. The neural network used is based on the U-net architecture (<a href="#ref-ronneberger2015u">Ronneberger et al. 2015</a>) using residual blocks. Figure <a href="#f:resunet">19</a> illustrates the architecture. The network was trained using a learning rate of 0.01 with a polynomial learning rate scheduler with an exponent of 0.9 and a batch size of 8. The performance was quantified using the Dice and centerlineDice (clDice) (<a href="#ref-paetzold2019cldice">Paetzold et al. 2019</a>) metrics.

<figure id="f:resunet" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Neural network architecture used.</figcaption>
</figure>

The first experiment concerns the CORTEX dataset. The dataset was randomly split into 40 training images and 10 validation images. A set of vessel texture maps, $`\mathcal{M}`$, was extracted from the 40 training images. A respective set $`\mathcal{I}`$ of 100 artificial images was generated using a fixed subset of $`m`$ texture maps from $`\mathcal{M}`$. To make sure that the results are not due to the specific maps used, $`n_d`$ different sets were generated. The specific procedure used to generate the artificial datasets is described in Algorithm <a href="#a:packs">20</a>. The results presented in this section are always averaged over the $`n_d`$ datasets.

We considered cases where $`m=1`$, $`m=5`$, $`m=10`$, and $`m=160`$ vessels were annotated. For $`m=1`$, $`m=5`$ and $`m=10`$, $`n_d=10`$ datasets were generated. Only one dataset of 100 images was generated for $`m=160`$ maps since this set already has a large number of maps and random fluctuations due to the specific maps used are not expected.

<figure id="a:packs" data-latex-placement="!h">
<div class="algorithm">

</div>
<figcaption>Data generation for the neural network experiments</figcaption>
</figure>

For comparison, a baseline model was trained on the 40 original training images using all available vessels. This model represents the best result that can be obtained with the training parameters considered since all pixels are annotated and used for training. As an additional comparison, the network was also trained on 100 randomly selected images generated by the TREE method, and the training was repeated 10 times on different sets of randomly selected images to asses the performance variability when using different images. This dataset is used as a baseline comparison with artificial images containing realistic vessel geometries without real textures.

In all experiments the networks were trained for 300 epochs. The performance of all trained models was measured on the validation set consisting of 10 real images. The models with the lowest validation loss found during training were used. The only data augmentation used was the inclusion of Gaussian noise and the darkening of the image according to the distance from the center of the sample to simulate microscope illumination.

The results are shown in Table <a href="#t:result_table1_cortex">2</a>. In the table, average and standard deviation values among the 10 sets of 100 images are shown for $`m=1`$, $`m=5`$, and $`m=10`$ annotated maps. No statistics were calculated for the 160 maps and baseline images because they have a single set of images. The results show that the performance difference between 5 and 160 maps is small. Thus, 5 maps seem to be enough to reach the best accuracy for this dataset when using artificial images. Interestingly, while we observe a 0.05 difference in Dice score between the artificial images using 5 maps and the baseline, the clDice difference is only around 0.03. Thus, the centerlines of the vessels are being segmented with an accuracy that is very close to the baseline. Training on the TREE images results in a Dice score of 0.79. Thus, our method achieves a 10.1% Dice score improvement over the usual pre-training strategy using artificial tree generation.

| Experiment     | Dice                    | clDice                  |
|:---------------|:------------------------|:------------------------|
| Baseline       | 0.92                    | 0.94                    |
| TREE           | 0.79$`\pm`$<!-- -->0.02 | 0.85$`\pm`$<!-- -->0.04 |
| Using 1 map    | 0.85$`\pm`$<!-- -->0.03 | 0.89$`\pm`$<!-- -->0.05 |
| Using 5 maps   | 0.87$`\pm`$<!-- -->0.02 | 0.91$`\pm`$<!-- -->0.02 |
| Using 10 maps  | 0.88$`\pm`$<!-- -->0.01 | 0.91$`\pm`$<!-- -->0.01 |
| Using 160 maps | 0.88                    | 0.91                    |

Average and standard deviation of the Dice and clDice metrics obtained for neural networks trained on artificial vessel images and evaluated on the CORTEX dataset. {#t:result_table1_cortex}

Example segmentations of a network trained with artificial images generated from 1 annotation are shown in Figure <a href="#f:predicion_1_map_pack3">21</a>. The images indicate the reason for the difference observed between the Dice and clDice metrics. The network trained on the artificial images tends to generate segmented vessels with the same caliber. Thus, while the centerline of the vessels is correctly identified, the caliber is not always correct. Adding more annotations with high-caliber vessels may mitigate this problem.

<figure id="f:predicion_1_map_pack3" data-latex-placement="!h">
[Image omitted for text-only version]
<figcaption>Example segmentation of a network trained on artificial images. Three images of the validation set (real images) are shown in (a), (b), and (c). The corresponding ground truth annotations are shown in (d), (e), and (f). The results of the network are shown in (g), (h), and (i).</figcaption>
</figure>

To verify if the method is general enough to be applied to different datasets, the experiments done for the CORTEX dataset were repeated on the DRIVE dataset. The artificial images were created using exactly the same procedure used for the CORTEX dataset. In short, a set of vessel texture maps was extracted from the 20 training images of the original train/test split of the DRIVE dataset. Algorithm <a href="#a:packs">20</a> was then applied to create artificial images using $`m=1`$, $`m=5`$, $`m=10`$, and $`m=160`$ maps. $`n_d=10`$ sets of 100 images were created for the $`m=1`$, $`m=5`$, and $`m=10`$ cases, and one set of 100 images was created for $`m=160`$. A baseline model was trained on the 20 training images of the dataset for comparison. An additional model was trained 10 times on different sets of 100 randomly selected images generated by the TREE method. The performance of all trained models was evaluated on the original test split of the dataset.

There were two changes in network training compared to the CORTEX dataset. The networks were trained for 1000 epochs, which was necessary for convergence. A lightweight data augmentation consisting of random flipping, random rotation, and random resized crops was necessary to avoid overfitting[^3].

The results of the experiments are shown in Table <a href="#t:result_table1_drive">3</a>. For the DRIVE dataset, 10 maps resulted in a Dice score of 0.74, which is close to the baseline score of 0.81. The TREE images led to poor results since the method used to generate the images was originally developed for microscopy images. In contrast, our method can naturally adapt to different imaging modalities.

| Experiment     | Dice                    | clDice                  |
|:---------------|:------------------------|:------------------------|
| Baseline       | 0.81                    | 0.80                    |
| TREE           | 0.21$`\pm`$<!-- -->0.01 | 0.23$`\pm`$<!-- -->0.06 |
| Using 1 map    | 0.68$`\pm`$<!-- -->0.05 | 0.67$`\pm`$<!-- -->0.06 |
| Using 5 maps   | 0.68$`\pm`$<!-- -->0.05 | 0.68$`\pm`$<!-- -->0.06 |
| Using 10 maps  | 0.74$`\pm`$<!-- -->0.02 | 0.73$`\pm`$<!-- -->0.02 |
| Using 160 maps | 0.75                    | 0.75                    |

Average and standard deviation of the Dice and clDice metrics obtained for neural networks trained on artificial vessel images and evaluated on the DRIVE dataset. {#t:result_table1_drive}

The amount of manual annotation required by our methodology was measured for the CORTEX and DRIVE datasets. For each map used in the experiments, the length of the vessel represented by the map was measured. This is given by the length of the curve represented by the set of medial axis points $`P_m`$ used to create the map. For the CORTEX dataset, 5 maps required, on average, annotating a total vessel length of 513 pixels. For comparison, the vasculature of the 40 training images has a total length of 1555520 pixels. Thus, the Dice score of 0.87 is reached using 0.03% of the annotation effort used on the full dataset.

For the DRIVE dataset, the creation of 10 maps required on average annotating a total vessel length of 510 pixels. The number is similar to annotating 5 maps on the CORTEX dataset because the annotations on the DRIVE dataset for each map were usually shorter since vessel segments on the DRIVE dataset tend to have shorter lengths. The 20 training images have a total vessel length of 174080 pixels. Thus, the Dice score of 0.74 is reached using 0.29% of the vessels annotated on the full dataset. Note that the method is used only for pre-training the network. After pre-training, the network can be refined on the real samples.

# Conclusion

We presented a methodology for generating artificial blood vessel images with realistic textures. The method is built upon the fact that vessel segments can be used as a reference for a coordinate system, which allows the definition of texture maps. The method is fast, being able to generate hundreds of images from a single vessel segment annotation, and can significantly decrease the manual annotation effort when segmenting novel datasets.

We showed that by annotating only 5 vessel segments of the CORTEX dataset, corresponding to only 0.03% of the vessels, the generated artificial images can be used for training a neural network with a Dice score of 0.87$`\pm`$<!-- -->0.02 and a clDice score of 0.91$`\pm`$<!-- -->0.02, which are very close to the baseline values of, respectively, 0.92 and 0.94 obtained when annotating the whole dataset. The annotation can be done in under 2 minutes by a trained annotator. For the DRIVE dataset, annotating only around 0.29% of the vessels, or 10 segments, led to a Dice score of 0.74$`\pm`$<!-- -->0.02 and a clDice score of 0.73$`\pm`$<!-- -->0.02, which are also close to the baseline values of, respectively, 0.81 and 0.8.

The results were compared with those obtained by a common procedure used for pre-training neural networks in recent works (<a href="#ref-tetteh2020deepvesselnet">Tetteh et al. 2020</a>; <a href="#ref-mou2021cs2"><span class="nocase">Mou et al.</span> 2021</a>; <a href="#ref-todorov2020machine"><span class="nocase">Todorov et al.</span> 2020</a>; <a href="#ref-wijethilake2023deep">Wijethilake et al. 2023</a>), which we called the TREE method. For the CORTEX dataset, the TREE method resulted in a Dice score of 0.79$`\pm`$<!-- -->0.02 and a clDice score of 0.85$`\pm`$<!-- -->0.04, which are lower than the values obtained by our method. For the DRIVE dataset, a Dice score of 0.21$`\pm`$<!-- -->0.01 and a clDice score of 0.23$`\pm`$<!-- -->0.06 were obtained using the TREE method. The reason for the weak performance is that the method was developed with a focus on microscopy images. The advantage of our method is that it can adapt to different imaging modalities since the appearance of the vessels is extracted directly from small segments of the real images. Thus, the method worked well for both fluorescence microscopy and fundus photography images.

The developed methodology also allows to easily add new annotations to the dataset. For instance, one can annotate a specific blood vessel that was not segmented correctly by the network and generate a new set of artificial images containing blood vessels with the same appearance but different geometries. The network can then be refined on the images.

The analysis was done with lightweight pre-processing and augmentation procedures to avoid biasing the results. That is, it is important to verify that the performance of the method does not come from a complex augmentation pipeline, but from the representation capability of the textures captured from the real images. For future analyses, more elaborate pre-processing steps, such as histogram equalization (<a href="#ref-vijayalakshmi2023strategic">Vijayalakshmi and Nath 2023</a>, <a href="#ref-vijayalakshmi2022novel">2022</a>), and augmentation procedures may be used to improve the results of the method.

The generation of vessel texture maps also opens new possibilities for disentangling the influence of shape and texture on neural network training. It allows identifying the importance of each aspect and to apply data augmentation and style transfer techniques separately for them. This is a promising research direction with some recent interesting results regarding the texture bias of CNNs (<a href="#ref-geirhos2018imagenet">Geirhos et al. 2018</a>).

# Funding

C. H. Comin thanks FAPESP (grant no. 21/12354-8) for financial support. M. V. da Silva thanks FAPESP (grant no. 23/03975-4) and the Google PhD Fellowship Program for financial support.

# CRediT author statement

**Adriano dos Reis Carvalho**: Methodology, Software, Validation, Investigation, Visualization, Data Curation, Writing - Original Draft. **Matheus Viana da Silva**: Software, Methodology, Writing – review and editing. **Cesar H. Comin**: Conceptualization, Investigation, Methodology, Software, Supervision, Project Administration, Writing - Original Draft, Writing – review and editing.

<div id="refs" class="references csl-bib-body hanging-indent">

<div id="ref-almotiri2018multi" class="csl-entry">

Almotiri, Jasem, Khaled Elleithy, and Abdelrahman Elleithy. 2018. “A Multi-Anatomical Retinal Structure Segmentation System for Automatic Eye Screening Using Morphological Adaptive Fuzzy Thresholding.” *IEEE Journal of Translational Engineering in Health and Medicine* 6: 1–23.

</div>

<div id="ref-chen2023all" class="csl-entry">

Chen, Cheng, Kangneng Zhou, Zhiliang Wang, Qian Zhang, and Ruoxiu Xiao. 2023. “All Answers Are in the Images: A Review of Deep Learning for Cerebrovascular Segmentation.” *Computerized Medical Imaging and Graphics*, 102229.

</div>

<div id="ref-dolati2015pre" class="csl-entry">

Dolati, Parviz, Alexandra Golby, Daniel Eichberg, et al. 2015. “Pre-Operative Image-Based Segmentation of the Cranial Nerves and Blood Vessels in Microvascular Decompression: Can We Prevent Unnecessary Explorations?” *Clinical Neurology and Neurosurgery* 139: 159–65.

</div>

<div id="ref-eladawi2018early" class="csl-entry">

Eladawi, Nabila, Mohammed Elmogy, Fahmi Khalifa, et al. 2018. “Early Diabetic Retinopathy Diagnosis Based on Local Retinal Blood Vessel Analysis in Optical Coherence Tomography Angiography (OCTA) Images.” *Medical Physics* 45 (10): 4582–99.

</div>

<div id="ref-freitas2022unbiased" class="csl-entry">

Freitas-Andrade, Moises, Cesar H Comin, Matheus Viana da Silva, Luciano da F Costa, and Baptiste Lacoste. 2022. “Unbiased Analysis of Mouse Brain Endothelial Networks from Two-or Three-Dimensional Fluorescence Images.” *Neurophotonics* 9 (3): 031916.

</div>

<div id="ref-galarreta2013three" class="csl-entry">

Galarreta-Valverde, Miguel A, Maysa MG Macedo, Choukri Mekkaoui, and Marcel P Jackowski. 2013. “Three-Dimensional Synthetic Blood Vessel Generation Using Stochastic l-Systems.” *Medical Imaging 2013: Image Processing* 8669: 86691I.

</div>

<div id="ref-geirhos2018imagenet" class="csl-entry">

Geirhos, Robert, Patricia Rubisch, Claudio Michaelis, Matthias Bethge, Felix A Wichmann, and Wieland Brendel. 2018. “ImageNet-Trained CNNs Are Biased Towards Texture; Increasing Shape Bias Improves Accuracy and Robustness.” *arXiv Preprint arXiv:1811.12231*.

</div>

<div id="ref-hughes2013computer" class="csl-entry">

Hughes, John F, Andries van Dam, Morgan McGuire, et al. 2013. *Computer Graphics: Principles and Practice*. Addison-Wesley Professional.

</div>

<div id="ref-islam2021shape" class="csl-entry">

Islam, Md Amirul, Matthew Kowal, Patrick Esser, et al. 2021. “Shape or Texture: Understanding Discriminative Features in Cnns.” *arXiv Preprint arXiv:2101.11604*.

</div>

<div id="ref-kocinski20123d" class="csl-entry">

Kociński, Marek, Artur Klepaczko, Andrzej Materka, Martha Chekenya, and Arvid Lundervold. 2012. “3D Image Texture Analysis of Simulated and Real-World Vascular Trees.” *Computer Methods and Programs in Biomedicine* 107 (2): 140–54.

</div>

<div id="ref-li2021blood" class="csl-entry">

Li, Zhenwei, Mengli Jia, Xiaoli Yang, and Mengying Xu. 2021. “Blood Vessel Segmentation of Retinal Image Based on Dense-u-Net Network.” *Micromachines* 12 (12): 1478.

</div>

<div id="ref-lindenmayer1968mathematical" class="csl-entry">

Lindenmayer, Aristid. 1968. “Mathematical Models for Cellular Interactions in Development i. Filaments with One-Sided Inputs.” *Journal of Theoretical Biology* 18 (3): 280–99.

</div>

<div id="ref-ma2019neural" class="csl-entry">

Ma, Chunwei, Zhanghexuan Ji, and Mingchen Gao. 2019. “Neural Style Transfer Improves 3D Cardiovascular MR Image Segmentation on Inconsistent Data.” *Medical Image Computing and Computer Assisted Intervention–MICCAI 2019: 22nd International Conference, Shenzhen, China, October 13–17, 2019, Proceedings, Part II 22*, 128–36.

</div>

<div id="ref-mookiah2021review" class="csl-entry">

Mookiah, Muthu Rama Krishnan, Stephen Hogg, Tom J MacGillivray, et al. 2021. “A Review of Machine Learning Methods for Retinal Blood Vessel Segmentation and Artery/Vein Classification.” *Medical Image Analysis* 68: 101905.

</div>

<div id="ref-mou2021cs2" class="csl-entry">

<span class="nocase">Mou, Lei, Yitian Zhao, Huazhu Fu, et al.</span> 2021. “CS2-Net: Deep Learning Segmentation of Curvilinear Structures in Medical Imaging.” *Medical Image Analysis* 67: 101874.

</div>

<div id="ref-nair2020blood" class="csl-entry">

Nair, Arun T, and K Muthuvel. 2020. “Blood Vessel Segmentation and Diabetic Retinopathy Recognition: An Intelligent Approach.” *Computer Methods in Biomechanics and Biomedical Engineering: Imaging & Visualization* 8 (2): 169–81.

</div>

<div id="ref-ouellette2020vascular" class="csl-entry">

<span class="nocase">Ouellette, Julie, Xavier Toussay, Cesar H Comin, et al.</span> 2020. “Vascular Contributions to 16p11. 2 Deletion Autism Syndrome Modeled in Mice.” *Nature Neuroscience* 23 (9): 1090–101.

</div>

<div id="ref-paetzold2019cldice" class="csl-entry">

Paetzold, Johannes C, Suprosanna Shit, Ivan Ezhov, et al. 2019. “clDice—a Novel Connectivity-Preserving Loss Function for Vessel Segmentation.” *Medical Imaging Meets NeurIPS 2019 Workshop*.

</div>

<div id="ref-popescu2021retinal" class="csl-entry">

Popescu, Dan, Mihaela Deaconu, Loretta Ichim, and Grigore Stamatescu. 2021. “Retinal Blood Vessel Segmentation Using Pix2pix Gan.” *2021 29th Mediterranean Conference on Control and Automation (MED)*, 1173–78.

</div>

<div id="ref-roda2021blood" class="csl-entry">

Roda, Niccolò, Giada Blandano, and Pier Giuseppe Pelicci. 2021. “Blood Vessels and Peripheral Nerves as Key Players in Cancer Progression and Therapy Resistance.” *Cancers* 13 (17): 4471.

</div>

<div id="ref-ronneberger2015u" class="csl-entry">

Ronneberger, Olaf, Philipp Fischer, and Thomas Brox. 2015. “U-Net: Convolutional Networks for Biomedical Image Segmentation.” *Medical Image Computing and Computer-Assisted Intervention–MICCAI 2015: 18th International Conference, Munich, Germany, October 5-9, 2015, Proceedings, Part III 18*, 234–41.

</div>

<div id="ref-sangeethaa2018intelligent" class="csl-entry">

Sangeethaa, SN, and P Uma Maheswari. 2018. “An Intelligent Model for Blood Vessel Segmentation in Diagnosing DR Using CNN.” *Journal of Medical Systems* 42 (10): 175.

</div>

<div id="ref-schneider2012tissue" class="csl-entry">

Schneider, Matthias, Johannes Reichold, Bruno Weber, Gábor Székely, and Sven Hirsch. 2012. “Tissue Metabolism Driven Arterial Tree Generation.” *Medical Image Analysis* 16 (7): 1397–414.

</div>

<div id="ref-StaalDRIVE" class="csl-entry">

Staal, J., M. D. Abramoff, M. Niemeijer, M. A. Viergever, and B. van Ginneken. 2004. “Ridge-Based Vessel Segmentation in Color Images of the Retina.” *IEEE Transactions on Medical Imaging* 23 (4): 501–9. <a href="https://doi.org/10.1109/TMI.2004.825627">https://doi.org/10.1109/TMI.2004.825627</a>.

</div>

<div id="ref-sule2020effects" class="csl-entry">

Sule, Olubunmi Omobola, Serestina Viriri, and Abdultaofeek Abayomi. 2020. “Effects of Image Enhancement Techniques on CNNs Based Algorithms for Segmentation of Blood Vessels: A Review.” *2020 International Conference on Artificial Intelligence, Big Data, Computing and Data Communication Systems (icABCD)*, 1–6.

</div>

<div id="ref-tagliasacchi20163d" class="csl-entry">

Tagliasacchi, Andrea, Thomas Delame, Michela Spagnuolo, Nina Amenta, and Alexandru Telea. 2016. “3d Skeletons: A State-of-the-Art Report.” *Computer Graphics Forum* 35: 573–97.

</div>

<div id="ref-tetteh2020deepvesselnet" class="csl-entry">

Tetteh, Giles, Velizar Efremov, Nils D Forkert, et al. 2020. “Deepvesselnet: Vessel Segmentation, Centerline Prediction, and Bifurcation Detection in 3-d Angiographic Volumes.” *Frontiers in Neuroscience* 14: 592352.

</div>

<div id="ref-tmenova2019cyclegan" class="csl-entry">

Tmenova, Oleksandra, Rémi Martin, and Luc Duong. 2019. “CycleGAN for Style Transfer in x-Ray Angiography.” *International Journal of Computer Assisted Radiology and Surgery* 14: 1785–94.

</div>

<div id="ref-todorov2020machine" class="csl-entry">

<span class="nocase">Todorov, Mihail Ivilinov, Johannes Christian Paetzold, Oliver Schoppe, et al.</span> 2020. “Machine Learning Analysis of Whole Mouse Brain Vasculature.” *Nature Methods* 17 (4): 442–49.

</div>

<div id="ref-vijayalakshmi2022novel" class="csl-entry">

Vijayalakshmi, Dhurairajan, and Malaya Kumar Nath. 2022. “A Novel Multilevel Framework Based Contrast Enhancement for Uniform and Non-Uniform Background Images Using a Suitable Histogram Equalization.” *Digital Signal Processing* 127: 103532.

</div>

<div id="ref-vijayalakshmi2023strategic" class="csl-entry">

Vijayalakshmi, Dhurairajan, and Malaya Kumar Nath. 2023. “A Strategic Approach Towards Contrast Enhancement by Two-Dimensional Histogram Equalization Based on Total Variational Decomposition.” *Multimedia Tools and Applications* 82 (13): 19247–74.

</div>

<div id="ref-wijethilake2023deep" class="csl-entry">

Wijethilake, Navodini, Mithunjha Anandakumar, Cheng Zheng, Peter TC So, Murat Yildirim, and Dushan N Wadduwage. 2023. “DEEP-Squared: Deep Learning Powered de-Scattering with Excitation Patterning.” *Light: Science & Applications* 12 (1): 228.

</div>

<div id="ref-wong2019blood" class="csl-entry">

Wong, Sau May, Jacobus FA Jansen, C Eleana Zhang, et al. 2019. “Blood-Brain Barrier Impairment and Hypoperfusion Are Linked in Cerebral Small Vessel Disease.” *Neurology* 92 (15): e1669–77.

</div>

<div id="ref-wu2022vessel" class="csl-entry">

Wu, Chulin, Heye Zhang, Jiaqi Chen, et al. 2022. “Vessel-GAN: Angiographic Reconstructions from Myocardial CT Perfusion with Explainable Generative Adversarial Networks.” *Future Generation Computer Systems* 130: 128–39.

</div>

<div id="ref-zhao2018synthesizing" class="csl-entry">

Zhao, He, Huiqi Li, Sebastian Maurer-Stroh, and Li Cheng. 2018. “Synthesizing Retinal and Neuronal Images with Generative Adversarial Nets.” *Medical Image Analysis* 49: 14–26.

</div>

</div>

[^1]: <a href="https://github.com/AdrianoCarvalh0/texture_codes.git">https://github.com/AdrianoCarvalh0/texture_codes.git</a>

[^2]: Regarding the mathematical notation used in the text, vectors and points are represented in bold. Sets of vectors and points are represented by uppercase letters. Matrices are represented by uppercase letters and sets of matrices are represented by letters using a calligraphic font.

[^3]: The transformation RandomAffine(angle=45, scale=(0.95, 1.20)) from the *torchvision* Python package was used for the rotation and random resized crop.
