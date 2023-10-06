# S2DS project: EPRI – Weld Dilution Characterization
_End of project: April 6th 2023_ 

### Team members:
Youssef Assaf, Gediminas Pažėra, Sarah Shannon, Moran Sharon and Dan Shaked-Renous


### Disclaimer
This repository contains the research notebooks from a project done for EPRI (our client). The source code and the final output of the project are the properties of EPRI. EPRI were generous to allow us to present our notebooks and some of the sample images as evidence of our capabilities.

## Introduction
### The challenge 
Our client was the Electric Power Research Institute (EPRI). They are an independent non-profit research organisation based in the US that carries out research in the areas of nuclear power, electricity infrastructure, and sustainable energy. EPRI has around 1,400 members, including leaders and technical experts from the electricity sector, academia, and government. 
EPRI wanted to provide a service to their members to automatically measure the dimensions and properties of welds. Welds are used in the nuclear power industry, where weld integrity is critical for safety. The motivation behind the tool development was to enable EPRI's members to efficiently evaluate the quality of welds. One of the key properties they wanted to calculate was the weld dilution ratio because this can be used as a proxy for weld strength. The downstream impact of our study was to enable EPRI members to explore correlations between the materials used in the welding process and the strength of the weld. 
The previous approach to the problem was to manually calculate the weld properties. This took a technician approximately 5 minutes to do and was prone to human error.  EPRI used a software package that required a licence and could not be automated to process multiple images.  To develop the automated tool, we were provided with 627 cross-sectional images of welds. A csv file containing the height, width, depth, and weld dilution ratio associated with each image was also provided.  
 
### The Approach
Facing the challenge of processing a dataset of images and spreadsheets with limited time and a wide range of image quality, our approach was structured to ensure we were able to deliver a solution that meets the project's objectives.

Firstly, we cleaned and solidified the dataset images and spreadsheets to eliminate any inconsistencies that could have a negative impact on our analysis. We then proceeded to read the sample ID from the image and measure its scale bar. We also identified the reference line of the base metal line, segmented and identified the weld from the background, contoured the weld, and measured the dimensional parameters of the weld.

Given the low number of dataset images (601 after cleaning), the abstract shape of the welds, and their different levels of contrast compared to the background, we opted not to use ML-based supervised models. Instead, we chose to use traditional image processing packages like scikit-image and openCV, settling on the former in the end. Our pipeline contained two parallel processes: ID and scale extraction, and weld measurement. The former uses a pre-trained OCR tool. And the latter comprises binary filtering, contouring, and watershed filling of the weld region, which proved to be effective for over 60% of the images.

The main constraint we encountered was the low-contrast images. We tried several techniques, including edge enhancement, colour spaces and channel sharpening, and line-profiling, but none provided an immediate solution. Our latest method investigation uses a difference of two Gaussian blurrings of the grayscale image and searches a continuous path between two points on the reference line using scikit-image's path_through_array, which shows great promise, but we didn't have enough time to integrate it into our pipeline as an alternative method.


### The results and impact 
The overall feedback from EPRI was positive and we achieved the goals of the project in the allocated time. Our tool was able to successfully measure the weld dimensions and dilution ratio for ~65% of the images when compared to the observations. For the remaining images, we recommended to EPRI some alternative image segmentation methods that could further improve the success rate. 

With our tool, weld dimensions and dilution ratio can now be measured in a standardised manner, eliminating the potential for human error. In retrospect, we thought we should have leveraged the domain knowledge of EPRI to a greater extent early on in the process. Specifically, during the kick-off meeting, we could have proactively inquired about their prior experiences to gain a better understanding of the approaches they had tried. 

Our results are summarised in the figures below. The figures show the weld dilution (%), depth (mm), and area (squared mm).

In conclusion, our approach was effective in delivering a solution that met the project's objectives within the given constraints. By using traditional image processing tools and carefully considering available options, we overcame the constraint of low-contrast images. Our approach can be replicated in similar situations to achieve positive results or even improved upon by investigating the recommended alternative methods.


# Installation of required packages
## pyTesseract
Installation for Amazon WorkSpaces Linux
```
sudo yum install epel-release
wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo rpm -Uvh epel-release-latest-7.noarch.rpm
sudo yum install tesseract tesseract-langpack-eng pytesseract
sudo pip3 install pytesseract
pytesseract --version
```

## Installing development requirements
```
    pip install -r requirements.txt
```

# Project Description

Create an automatic tool for Electric Power Research Institute (EPRI) members to calculate the dimensions and properties of welds.

Good quality welds are important for the safety and functioning of nuclear power plants. 

Given a weld:

<img width="785" alt="230331931-c74bb953-a13b-4812-939e-dc7fd132ea53" src="https://user-images.githubusercontent.com/13516235/231093640-c961809c-d7bb-43cb-8baf-212818e1cc36.png">


We need to calculate the following:
- ${\color{red}Width}$
- ${\color{blue}Height}$
- ${\color{purple}Penetration}$ ${\color{purple}Depth}$
- Weld Dilution which is given by A_substrate / (A_substrate + A_filler) or Bottom Area / (Bottom Area + Top Area)


# Pipeline
__Reminder:__ This repository doesn't contain the source code with the pipeline as it is a property of EPRI.

Here is a visual sketch of the pipeline:

<img width="742" alt="230374728-2876d969-a72a-4112-a119-6f9636123fda" src="https://user-images.githubusercontent.com/13516235/231093895-9561793b-f27d-4da5-bf0b-a6a6043f696d.png">


The pipeline:
1) Reads the weld image and splits it into the top part that contains the weld and the bottom part which contains the text and scale bar.
2) For the bottom bit, the Image ID and the scale is read and output in a .csv file
3) For the top bit, there are a few steps that the pipeline takes:
    - Detects the reference (hough) line
    - Does "Countour + Fill" 
    - Applies a Binary filter and crops to find the final weld area
4) To decide which images were read correctly, the pipeline goes through failure criteria and discards the images that do not meet this criteria. The failure criteria will be described below in a different section.
5) Finally, an image with all the visual information is created and important measures are output to the .csv file

# Outputs in .csv

All the data of the pipeline is output to a "results.csv" file with the corresponding collumns:
- _image_file_name_: The name of the image file
- _id_text_: The read ID of the image by the OCR.
- _same_as_filename_: A flag that writes "True" if id_text is identical to the image filename
- _suggested_name_change_: If _same_as_filename_ is "False", then the model suggest an improved filename that checks some of the common mistaken letters.
- _dim_value_: The dimension of the scale bar. 
- _scale_read_success_: A flag that outputs "True" if the dimension is read correctly.
- _line_length_: The length of the scale bar in pixels
- _line_read_success_: A flag that outputs "True" if the the scale bar was read correctly.
- _hough_line_not_found_: A flag that determines the sucess of finding the hough line. Returns "False" if the hough line was found correctly.
- _units_: The units that the measurements are in (default is mm but changes to pixels if the scale bar was not read correctly)
- _height_: The computed height of the weld above the hough line.
- _width_: The computed width of the weld.
- _depth_: The computed depth below the hough line.
- _top_area_: The area of the upper weld (above the hough line).
- _bottom_area_: The area of the bottom weld (bellow the hough line).
- _weld_dilution_: The weld dilution that is calculated by the formula Weld Dilution = bottom area / (bottom area + top area).
- _object_in_img_: True if some object was detected, False if no object was detected.
- _failing_criteria1_: False if too many black pixels above the reference line, True otherwise.
- _failing_criteria2_: False if too many black pixels along the reference line, True otherwise.
- _failing_criteria3_: False if too many black pixels below the reference line, True otherwise.

# Output image format

![229575211-d1ece6c8-4abc-45fc-b2bb-0ea78625b92f](https://user-images.githubusercontent.com/13516235/231093959-757d4aa3-3d9b-4544-a39b-9a853ce9a960.png)

Index:
- Hough line/Width of the weld
- Weld area
- Width of the weld output on the image
- Area of the top weld output on the image
- Area of the bottom weld output on the image
- Weld dilution output on the image

# Failure Criteria

We proposed 3 criteria, calculating the __percentage (%) of black pixels__ in the weld (1) above, (2) on and (3) below 
the reference line. As shown in the picture:

<img width="900" alt="230335855-caaa8ec3-66f5-4bc3-879e-72b536a1f1c3" src="https://user-images.githubusercontent.com/13516235/231094011-2ad55c6e-915a-413c-af72-354662eea451.png">

The area above the reference line is defined as the area between the reference line (bottom) and the upper countour edges (top). Default value for the upper edge is 200 pixels above the reference line. 
The area of the reference line is defined as the area between the right and the left edges of the line.
The area below the reference line is defined as the area between the reference line (top) and 100 pixels below (bottom).

The criteria pass if the percentage of black pixels is more than 20% above the reference line, more than 10% on the reference line, and more than 15% below the reference line.

For the picture to be succesfully read by our pipeline, two out of the three criteria have to pass (our recommendation).

# Recommendations
## Image Quality

The current pipeline works best on images with high contrast between the weld and the background/base metal.<br>
Two recommendations come in to play here:
- Make sure that the background of the weld is darker than the weld.
- Make sure that the weld is well illuminated.

To verify good contrast, the techician that takes the image can check its histogram using a basic image processing tool.
See the below examples of low- and high-contrast images. The high contrast image can be identified by a well pronounced peak in the higher values of the pixel intensity.<br><br>


<img width="575" alt="230363817-6369df16-4400-4acc-bf3f-fc075e8ee343" src="https://user-images.githubusercontent.com/13516235/231094085-f831d662-5c82-4b56-9dbd-5559de84b15a.png">

<img width="600" alt="230363881-bdb3c161-89e2-4e41-aa76-e422e6a40389" src="https://user-images.githubusercontent.com/13516235/231094157-efbcb870-68de-4449-a401-d151b2507f37.png">


## Possible next steps
Several methods were tested to find a solution for the low-contrast images. Some of them managed to contour the region close to the weld, but require further optimization or possibly a combination with other methods.
These are:
- Contouring using line-profiling based on hysteresis. ([nb1](https://github.com/S2DSLondon/Spring23_EPRI_Blue/blob/main/notebooks/DS_lineProfile_exploration.ipynb), [nb2](https://github.com/S2DSLondon/Spring23_EPRI_Blue/blob/main/notebooks/DS_laplacTransform.ipynb))
- Upsharp of the Hue channel in the HSV and then producing a mask on the Saturation channel. ([here](https://github.com/S2DSLondon/Spring23_EPRI_Blue/blob/main/notebooks/DS_upsharp_method.ipynb))
- Difference of Gaussians and skimage.graph.path_through_array ([here](https://github.com/S2DSLondon/Spring23_EPRI_Blue/blob/main/notebooks/DS_path_finder.ipynb)) 

__Update 11.04.2023:__ On the day we handed over our project, Meta released its AI-based "Segment Anything Model" (SAM). This model has the potential to segment _anything_ and might be useful to test is on the low-contrast images.
