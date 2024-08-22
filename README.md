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

The core concept of binary descriptors is to rely solely on the intensity information (i.e. the image itself) and to encode the information around a keypoint in a string of binary numbers. This allows for highly efficient comparison during the matching process when searching for corresponding keypoints.  The most widely used binary descriptors today include BRIEF, BRISK, ORB, FREAK, and KAZE, all of which are available in the OpenCV library. A comparison of several region detectors can be found in the paper titled [Image matching using SIFT, SURF, BRIEF, and ORB: performance comparison for distorted images](https://arxiv.org/pdf/1710.02726), by (Karami E. et al., 2017).
From a high-level perspective, binary descriptors consist of three major parts:
1.	A sampling pattern which describes where sample points are located around the location of a keypoint.
2.	A method for orientation compensation, which removes the influence of rotation of the image patch around a keypoint location.
3.	A method for sample-pair selection, which generates pairs of sample points which are compared based on their intensity values. If the intensity of the first point is greater than that of the second, a '1' is added to the binary string; otherwise, a '0' is added. After comparing all point pairs in the sampling pattern, a complete binary string is formed (hence the family name of this descriptor class).

#### Binary Robust Invariant Scalable Keypoints (BRISK)
BRISK is a keypoint detection method that combines a FAST-based detector with a binary descriptor, which is generated through intensity comparisons from a specific sampling of each keypoint's neighborhood. Points of interest are identified across both the image and scale dimensions using a saliency criterion similar to the AGAST method.

The sampling pattern of BRISK is composed of a number of sample points (blue), where a concentric ring (red) around each sample point denotes an area where Gaussian smoothing is applied to avoid aliasing. As opposed to some other binary descriptors such as ORB or BRIEF, the BRISK sampling pattern is fixed. 

![image](https://github.com/user-attachments/assets/e648d380-2afb-46dd-b5a6-d129717101a9)

During sample pair selection, the BRISK algorithm differentiates between long- and short-distance pairs. The long-distance pairs (i.e. sample points with a minimal distance between each other on the sample pattern) are used for estimating the orientation of the image patch from intensity gradients, whereas the short-distance pairs are used for the intensity comparisons from which the descriptor string is assembled.

Mathematical description of pairs for BRISK algorithm:

![image](https://github.com/user-attachments/assets/88cdd477-9ef8-43fd-ad4c-59a978b27858)


First, we define set A of all possible pairings of sample points. Then, we extract the subset L from A for which the euclidean distance is above a pre-defined upper threshold. This set is the long-distance pairs used for orientation estimation. Lastly, we extract those pairs from A whose euclidean distance is below a lower threshold. This set S contains the short-distance pairs for assembling the binary descriptor string.
The following figure shows the two types of distance pairs on the sampling pattern for long pairs (left) and short pairs (right).

![image](https://github.com/user-attachments/assets/35e1c0b1-1a7d-4223-91d8-926e6568aa7f)


![image](https://github.com/user-attachments/assets/3a4c7fa2-4844-4229-9039-5d5fdd5a67a9)

First, the gradient strength between two sample points is computed based on the normalized unit vector that gives the direction between both points multiplied with the intensity difference of both points at their respective scales. Then, the keypoint direction vector g is computed from the sum of all gradient strengths.
Based on g, we can use the direction of the sample pattern to rearrange the short-distance pairings and thus ensure rotation invariance. Based on the rotation-invariant short-distance pairings, the final binary descriptor can then be constructed


![image](https://github.com/user-attachments/assets/5bb7c68c-148a-4980-adb5-eae9437a43fe)


After computing the orientation angle of the keypoint from g, we use it to make the short-distance pairings invariant to rotation. Then, the intensity between all pairs in SS is compared and used to assemble the binary descriptor we can use for matching.

#### Oriented FAST and Rotated BRIEF (ORB)

ORB is a combination of a keypoint detector and descriptor algorithms. It works in two steps:
1.	Keypoint detection using FAST - ORB starts by finding keypoints. Once the key points in the image had been located, ORB then calculates a corresponding feature vector for each keypoint in the image. The ORB algorithm creates feature vectors that only contain ones and zeros. For this reason, they're called the binary feature vectors.

![image](https://github.com/user-attachments/assets/190b7c07-268a-4cdf-84e5-197157b8b289)

Keypoints around the cat's eyes and at the edges of its facial features.
2.	Description using BRIEF - ORB uses BRIEF, which in turn identifies the objects in images using the feature vectors created which represents the patterns of intensity around a key point and vary according to a specific keypoint and its surrounding pixel area. Multiple feature vectors can be used to identify a larger area and even a specific object in an image.
ORB, is incredibly fast, and impervious to noise illumination, and image transformations such as rotations. 
A comparison of several region detectors can be found in the paper titled [Image matching using SIFT, SURF, BRIEF, and ORB: performance comparison for distorted images](https://arxiv.org/pdf/1710.02726), by (Karami E. et al., 2017).

### Descriptor Matching

Once you have located and described a set of keypoints for each image in a sequence of frames, the next step in the tracking process is to find the best fit for each keypoint in successive frames. In order to do so, you need to implement a suitable similarity measure so that your tracking algorithm can uniquely assign keypoint pairs. 
#### Sum of Absolute Differences (SAD)
The first distance function is the "Sum of Absolute Differences (SAD)". As shown in the equation below, the SAD takes two descriptor vectors, da and db, as input. The SAD is calculated by subtracting each component in db from the corresponding component in da. The absolute values of these differences are then summed up. The SAD norm is also known as the L1-norm.

#### Sum of Squared Differences (SSD)
The second distance function is the "Sum of Squared Differences (SSD)," which, like the SAD, calculates the differences between individual components of two descriptor vectors. The key distinction between SAD and SSD is that SSD sums the squared differences rather than the absolute differences. The SSD norm is also known as the L2-norm. 
Both norms are illustrated in the following figure.

![image](https://github.com/user-attachments/assets/56d626d1-4400-475a-b2f7-c461a987b54d)




