# Detection analysis and modeling of radiomic features

The project centers around extracting, analyzing, and modeling radiomic features from medical images. 
This work constitutes a part of the efforts for my master's thesis.

## Data

For the project, radiomic features were extracted from CT images showing kidney tumors. The images are part of a set 
used to create a model using artificial neural networks that determines from the image whether a tumor is malignant or 
benign \[1\]. The images from the CT scanner were provided as DICOM files. The three-dimensional image obtained from the 
CT scanner is represented using a series of DICOM files, with a single file corresponding to a single cross-section.

When the images were originally used, a section of the image containing the kidney with the tumor was analyzed. This area 
had a square shape and a fixed size regardless of the image, which was 130x130 pixels. It was selected in such a way that 
the kidney with the tumor was in the center. These sections were used to select the area of interest for which radiomic 
features would be determined.

## Analysis

Using a fixed-size square as a segmentation renders the shape features meaningless. They are independent of gray levels, 
so they will have the same value for each image.

In order to investigate the effect of the size of the square region of interest on the values of radiomics features, a series 
of masks of increasing size were generated: 3x3, 5x5, ..., 143x143. These squares all share a common central pixel selected 
based on a slice of the image. The figure below displays some of the results obtained using these masks.

![Radiomics features graph](/docs/radiomics-features-graph-edited.png)

As can be seen, some of the characteristics, such as Gray Level Non Uniformity, behave in a manner similar to a linear 
function. In contrast, there are clear extremes for Skewness and Busyness. The values of Sum Avarge increase abruptly 
for side lengths between 60 and 70. A feature common to some of the graphs are abrupt changes for small mask side 
lengths. However, even this does not apply to all cases. Thus, it is difficult to draw any general conclusions based 
on the results obtained.

The parts of the images used to determine the area of interest were provided only for some files. Since tumors are 
usually spherical in shape, adjacent images in a series should be similar to each other. Therefore, the dependence of 
the feature values on the mask size was investigated for 5 neighboring images. Some of the results obtained are shown below.

![Radiomics features layers graph](/docs/radiomics-features-layers-graph.png)

In a large number of cases, larger differences are seen at small mask sizes. This is because the difference in the value of 
a single pixel is more significant if the total number of pixels in the area is small. However, this does not change the 
fact that for most cases the values obtained are similar but not identical. This means that there is a potential opportunity 
to use several neighboring images to increase the number of data that can be used to create a model.

## Model

The final task was to create two models using the XGBoost package. Based on the radiomic feature values obtained from the 
analysis of images in the arterial phase, they decide whether a kidney tumor is malignant or not. They were trained on two 
datasets, the size of which is given in the table below. The first is smaller and contains only radiomic features from the 
images for which clippings were provided. The second set is enriched with values from neighboring images obtained using the 
same mask. The four closest images were considered, while not all of them always existed.

| Data set | Malignant | Benign | Sum |
| :---: | :---: | :---: | :---: |
| Images | 1274 | 294 | 1568 |
| Images and adjacent | 1675 | 644 | 2319 |

At this point, it should be noted that the resulting models are Proof of Concept. The image fragments used for extraction 
include the kidney, tumor, and sometimes some background. In accepted practice, segmentation is used to show only the tumor. 
Therefore, the models were limited to the number of iterations = 2 and the maximum depth of the tree = 4. In addition, 
outlier points were left in the dataset and the number of features was 92. The results from the cross-validation of these 
models are shown in the table below. The result shows that radiomic features can be used to create a model to classify 
kidney tumors into malignant and benign.

| Metrics | Images | Images and adjacent |
| :---: | :---: | :---: |
| accuracy train | 0.889 | 0.936 |
| accuracy test | 0.870 | 0.922 |
| precision train | 0.997 | 0.994 | 
| precision test | 0.991 | 0.990 |
| f1 score train | 0.936 | 0.957 |
| f1 score test | 0.925 | 0.948 |

