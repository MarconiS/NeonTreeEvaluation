# A multi-sensor benchmark dataset for detecting individual trees in airborne RGB, Hyperspectral and LIDAR point clouds

Individual tree detection is a central task in forestry and ecology. Few papers analyze proposed methods across a wide geographic area. This limits the utility of tools and inhibits comparisons across methods. This benchmark dataset is the first dataset to have consistant annotation approach across a variety of ecosystems. 

If you would prefer not to clone this repo, a static version of the benchmark is here: [insert url later]

Mantainer: Ben Weinstein - University of Florida.

Description: The NeonTreeEvaluation dataset is a set of bounding boxes drawn on RGB imagery from the National Ecological Observation Network (NEON). NEON is a set of 45 sites (e.g. [TEAK](https://www.neonscience.org/field-sites/field-sites-map/TEAK)) that cover the dominant ecosystems in the US.

# How do I evaluate against the benchmark?

We have built an R package for easy evaluation and interacting with the benchmark evaluation data.

See https://github.com/weecology/NeonTreeEvaluation_package

# How were images annotated?

Each visible tree was annotated to create a bounding box that encompassed all portions of the vertical object. Fallen trees were not annotated. Visible standing stags were annotated. Trees which were judged to have less than 50% of biomass in the image edge were ignored.

<img src="figures/rectlabel.png" height="400">

For the point cloud annotations, the two dimensional bounding boxes were [draped](https://github.com/weecology/DeepLidar/blob/b3449f6bd4d0e00c24624ff82da5cfc0a018afc5/DeepForest/postprocessing.py#L13) over the point cloud, and all non-ground points (height < 2m) were excluded. Minor cosmetic cleanup was performed to include missing points. In general, the point cloud annotations should be seen as less thoroughly cleaned, given the tens of thousands of potential points in each image.

# Sites ([NEON locations](https://www.neonscience.org/field-sites/field-sites-map/list))

SJER: "Located at The San Joaquin Experimental Range, in the western foothills of the Sierra Nevada, this 18.2 kilometer terrestrial field site is a mix of open woodlands, shrubs and grasslands with low density cattle grazing." 
* 2533 training trees,	293 test trees

TEAK: "The site encompasses 5,138 hectares (12,696 acres) of mixed conifer and red fir forest, ranging in elevation from 1,990 to 2,807 m (6,529 – 9,209ft). The varied terrain is typical of the Sierra Nevada, with rugged mountains, meadows and prominent granite outcrops." 

* 3405 training tees,	747 test trees.

NIWO: The “alpine” site is Niwot Ridge Mountain Research State, Colorado (40.05425, -105.58237). This high elevation site (3000m) is near treeline with clusters of Subalpine Fir (Abies lasciocarpa) and Englemann Spruce (Picea engelmanii). Trees are very small compared to the others sites.

* 9730 training trees, 1699 test trees.

MLBS: The “Eastern Deciduous” site is the Mountain Lake Biological Station. Here the dense canopy is dominated by Red Maple (Acer rubrum) and White Oak (Quercus alba). 

* 1231 training trees, 489 test trees.

For more guidance on data loading, see /utilities.

# How can I add to this dataset?

Anyone is welcome to add to this dataset by cloning this repo and labeling a new site in [rectlabel](https://rectlabel.com/). NEON data is available on the [NEON data server](http://data.neonscience.org/home). We used the NEON 2018 “classified LiDAR point cloud” data
104 product (NEON ID: DP1.30003.001), and the “orthorectified camera mosaic” (NEON ID:
105 DP1.30010.001). Please follow the current folder structure, with .laz and .tif files saved together in a single folder, with a unique name, as well as a single annotations folder for the rect label xml files. See /SJER for an example.

For ease of access, we have added two unlabeled sites, [BART](https://www.neonscience.org/field-sites/field-sites-map/BART), and [UNDE](https://www.neonscience.org/field-sites/field-sites-map/UNDE), we encourage others to label these sites, or use models from the labeled data to predict into new, untested, areas. 

# RGB

```R
library(raster)
source("functions.R")

#Read RGB image as projected raster
rgb<-stack("../SJER/RGB/SJER_021.tif")

#Path to dataset
xmls<-readTreeXML(path="../SJER/")

#View one plot's annotations as polygons, project into UTM
#copy project utm zone (epsg), xml has no native projection metadata
xml_polygons <- xml_to_spatial_polygons(xmls[xmls$filename %in% "SJER_021.tif",],rgb)

plotRGB(rgb)
plot(xml_polygons,add=T)
```

<img src="figures/RGB_annotations.png" height="300">

# Lidar

To access the draped lidar hand annotations, use the "label" column. Each tree has a unique integer.

```R
> r<-readLAS("TEAK/training/NEON_D17_TEAK_DP1_315000_4094000_classified_point_cloud_colorized_crop.laz")
23424 points below 0 found.
> trees<-lasfilter(r,!label==0)
> plot(trees,color="label")
```

<img src="figures/lidar_hand_annotations.png" height="300">

We elected to keep all points, regardless of whether they correspond to tree annotation. Non-tree points have value 0. We highly recommend removing these points before predicting the point cloud. Since the annotations were made in the RGB and then draped on to the point cloud, there will naturally be some erroneous points at the borders of trees.

# Hyperspectral 
For the convienance of future users, we have downloaded, cropped and selected reasonable three band combinations for NEON hyperspectral images. 

```R
> r<-stack("/Users/Ben/Documents/NeonTreeEvaluation/MLBS/training/2018_MLBS_3_541000_4140000_image_crop_false_color.tif")
> nlayers(r)
[1] 3
> plotRGB(r,stretch="lin")
```

<img src="figures/Hyperspec_example.png" height="400">

## Training Tiles

We have uploaded the large training tiles to Zenodo for download. This includes

* The annotated trainings tiles (optionall cropped) for the NIWO, MLBS, SJER, and TEAK sites.
* Unannotated training tiles for the 15 additional sites. Training tiles do not overlap with evaluation plots.

TODO add zenodo link.

# Performance

To submit to this benchmark, please see evaluation.py. The primary evaluation statistic is precision and recall across all sites. It is up to the authors to choose the best probability threshold if appropriate. 

| Author                | Precision | Recall | Description                              |   |
|-----------------------|-----------|--------|------------------------------------------|---|
| Weinstein et al. 2019 <sup>1</sup> | 0.55      | 0.65   | Semi-supervised RGB Deep Learning        |   |
| Silva et al. 2016     |       0.09    | 0.23        | Unsupervised LiDAR raster  |   |
| Dalponte et al 2016   |           |        | Unsupervised LidAR raster  |   |                      |   |
| Li et al. 2012        |           |        | Unsupervised LiDAR point cloud|                      |   |

To reproduce the analysis for the benchmark comparison see https://github.com/weecology/NeonTreeEvaluation_analysis

## Cited
<sup>1</sup> Weinstein, Ben G., et al. "Individual tree-crown detection in RGB imagery using semi-supervised deep learning neural networks." Remote Sensing 11.11 (2019): 1309. https://www.mdpi.com/2072-4292/11/11/1309
Thanks to the lidR R package for making algorithms accessible for comparison.

Please submit a pull request, or contact the mantainer if you use these data in analysis and would like the results to be shown here.
