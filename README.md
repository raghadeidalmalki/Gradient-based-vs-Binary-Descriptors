## Tracking Image Features

We identify matching keypoints across a series of images to calculate the Time-To-Collision (TTC) with a leading object, such as a vehicle. Thus, we require a reliable method to accurately pair keypoints based on a similarity metric.
### Detectors vs Descriptors

•	A keypoint detector is an algorithm that chooses points from an image based on a local maximum of a function, such as the "cornerness" metric with the Harris detector. Keypoints are also known as referred to as interest points or salient points.
•	A descriptor is a vector of values, which describes the image patch around a keypoint. Techniques for generating descriptors range from comparing raw pixel values to more advanced methods like histograms of gradient orientations.
Descriptors help us to assign similar keypoints in different images to each other. As shown in the figure below, a set of keypoints in one frame is assigned keypoints in another frame such that the similarity of their respective descriptors is maximized and (ideally) the keypoints represent the same object in the image. In addition to maximizing similarity, a good descriptor should also be able to minimize the number of mismatches, i.e. avoid assigning keypoints that do not correspond to the same object.

![image](https://github.com/user-attachments/assets/9030b57f-156d-44a8-bda7-5a4dcee24d43)

### Histograms of Oriented Gradients (HOG)

The basic idea behind Histograms of Oriented Gradients (HOG) is to describe the structure of an object by the distribution of its intensity gradients in a local neighborhood. To achieve this, an image is divided into cells in which gradients are computed and collected in a histogram. The set of histograms from all cells is then utilized as a similarity measure to uniquely identify an image patch or object.
### Scale-Invariant Feature Transform (SIFT)
One of the best-known examples of the HOG family is the Scale-Invariant Feature Transform (SIFT), introduced in [1999 by David Lowe](https://home.cis.rit.edu/~cnspci/references/dip/feature_extraction/lowe1999.pdf).  SIFT combines a keypoint detector and a descriptor, and it operates through a five-step process, which is briefly summarized below.

1.	The initial step involves detecting keypoints in the image using a method known as "Laplacian-of-Gaussian (LoG)," which relies on second-order intensity derivatives. The LoG is applied across multiple scales of the image, primarily detecting blobs rather than corners. Along with a distinct scale level, keypoints are also assigned an orientation based on the intensity gradients in a local neighborhood around the keypoint.
Laplacian-of-Gaussian (LoG): 
The LoG operation works as follows: you start by slightly blurring the image using a Gaussian kernel. Next, you compute the sum of the second-order derivatives, known as the "Laplacian." This process helps to identify edges and corners within the image, which are useful for detecting keypoints. Since we're aiming to create a keypoint detector, additional steps are taken to suppress the edges. LoG is frequently utilized for detecting blobs.

2.	Next, the area around each keypoint is transformed by removing the orientation and thus ensuring a canonical orientation. Additionally, the area is resized to a 16 x 16 pixel patch, resulting in a normalized region.

![image](https://github.com/user-attachments/assets/04555c99-4614-4b1e-96ef-d9517ae2ec7a)



3.	Then, the orientation and magnitude of each pixel within the normalized patch are computed based on the intensity gradients Ix and Iy.
4.	In the fourth step, the normalized patch is divided into a grid of 4 x 4 cells. Within each cell, the orientations of pixels which exceed a threshold of magnitude are collected in  an 8-bin histogram.

![image](https://github.com/user-attachments/assets/dbcdc842-f8ee-429f-a3cc-82b66ec01b1a)

![image](https://github.com/user-attachments/assets/c4e12309-8bdd-426d-ac45-a57f7199d09c)

5.	Last, the 8-bin histograms of all 16 cells are concatenated into a 128-dimensional vector (the descriptor) which is used to uniquely represent the keypoint.


#### Advantages of SIFT: 
-	Its ability to robustly identify objects even among clutter and under partial occlusion. 
-	It is invariant to uniform changes in scale, to rotation, to changes in both brightness and contrast 
-	Partially invariant to affine distortions.
#### Drawbacks of SIFT:
-	Low speed.


### Binary Descriptors
A much faster alternative to HOG-based methods is the family of binary descriptors, which provide a fast alternative at only slightly worse accuracy and performance. The problem with HOG-based descriptors is that they are based on computing the intensity gradients, a computationally expensive process. This is why SIFT is generally not employed on devices with limited processing power, such as smartphones. Although there have been improvements, like the development of the similar SURF algorithm, which uses the less costly integral image instead of intensity gradients, real-time applications are still uncommon.




