# Image Segmentation of Femur Bone
Final Project of Bachelor of Engineering at K.N.Toosi University of Technology.

The objective of this project was to segment the femur bone from radiographic images. Initially, there was insufficient data available to employ machine learning techniques. Consequently, we endeavored to design an algorithm to detect the femur bone's edges within its entire surrounding environment using only a few sample images. Subsequently, we gained access to a larger dataset and proceeded to train a machine learning model.

---

## Classic Method

In this method, femur extraction was performed in three stages:
1. **Detecting the Circle**
2. **Identifying the Femur Shaft**
3. **Locating the Femoral Neck and Trochanters**

First, using a neural network, we cropped the region containing the femur from the entire image. Then, we cropped the area containing the femoral head using a U-Net model. A total of 570 labeled images were used for training.

The U-Net model used in this step resized the images to 512x512 pixels, which distorted the original aspect ratio and made it harder to detect circular shapes in the femur. To avoid this issue, we first added black padding to the image edges to make them square. This approach preserved the aspect ratio during resizing, simplifying circle detection. The output of this network was as follows:

![Initial U-Net Output with Aspect Ratio Preservation](assetts/shape_1.png)

Using the Hough Transform, we isolated the femoral head circle. The parameters for the Hough Transform were set as follows:
- `dp = 2`
- `minDist = 5`
- `param1 = 70`
- `param2 = 35`

The minimum radius for circle detection was set to one-third of the number of columns in the femoral head region, and the maximum radius was set to two-thirds of the number of columns in this region.

![Circle Detection Using Hough Transform](assetts/shape_2.png)

---

### Edge Detection for the Femur Shaft
We aimed to identify the edge points of the femur shaft. Initially, we selected points in each row based on intensity differences with adjacent pixels, but this approach was not always reliable. To improve it, we refined the method by selecting the top four pixels with the highest intensity differences in every tenth row.

![Selected Edge Points](assetts/shape_3.png)

We then used an error function to choose the optimal points in each row, calculating cumulative errors and refining the selection to minimize outliers.

![Filtered and Refined Edge Points](assetts/shape_4.png)

Outliers were removed by grouping points into clusters, computing average positions, and using a best-fit line as a reference to filter out unwanted points.

![Final Edge Detection with Outlier Removal](assetts/shape_7.png)

---

### Contour Following
We implemented a contour-following algorithm to trace the femur's edge. Starting from a known point on the right edge, the algorithm used pixel brightness differences to select the next point along the edge.

![Contour Following Schematic](assetts/shape_5.png)
![Contour Following Implementation](assetts/shape_6.png)

Different filters were applied based on the direction of the edge, allowing accurate tracing of the femur shaft and upper femur contours.

![Edge Completion Using Contour Following](assetts/shape_8.png)

To detect the upper edge, a rectangular region adjacent to the femoral head was analyzed to identify boundary points.

![Upper Edge Detection Region](assetts/shape_9.png)

The algorithm included stopping conditions, such as reaching the bottom edge of the femoral head region or the perimeter of the femoral head circle.

![Final Contour Output](assetts/shape_10.png)

---

## Femur Contour Extraction Using Neural Network
For this task, we used a U-Net model, trained on 583 labeled images (90% for training, 10% for testing). Labels were created using Photoshop, and broken bones were excluded as our goal was to analyze healthy bone structures for finite element analysis.

The network achieved an accuracy of 99.73% after 100 epochs, evaluated using the `bce_dice_loss` metric. The mask output was applied to original images with OpenCV.

![Neural Network Output](assetts/shape_11.png)

---

## Note on Contour Extraction
Both methods extracted the contour of only one femur (on the right side of the image). To obtain the other femur's contour, we mirrored the images, reducing both computational and labeling effort by half.
