## ARTI403 – Lab 10
## Image Segmentation and Feature Extraction

In this lab, I worked on image segmentation, which splits an image into the objects and the background. I solved the assessment task of the lab, Otsu's thresholding, where the threshold is not chosen by hand but found automatically from the histogram of the image. The task asked to show the original image, its histogram and the thresholded image. The main objective was to understand how Otsu chooses the threshold and when it works well.

## Image Used

I solved the task inside the lab notebook, under the Q1 section, and used the same coins image that the lab uses for simple thresholding (`data.coins()` from skimage). It is a 384 × 303 grayscale photo of 24 old coins in four rows on a dark background. The coins are bright and the background is dark, so it is a good image for a method that expects two groups of pixels. The lighting is not even, though: the top of the background is brighter than the rest, and some coins in the bottom rows are darker.

## Otsu's Thresholding

I applied Otsu with `cv2.threshold()` and the `THRESH_OTSU` flag, and displayed the original image, its histogram with the threshold marked on it, and the binary image.

- The histogram has a big dark peak around 35 to 50 for the background and a wider bright group around 120 to 200 for the coins
- Otsu chose T = 107, which falls in the valley between the two groups
- The background class had 61.2% of the pixels with a mean of 60.3, and the coins class 38.8% with a mean of 154.6
- In the thresholded image the coins are white and the background is black, and most coins are clearly separated from each other

## Checking the Threshold by Hand

To make sure I understood the method, I computed it myself from the histogram. For every threshold from 0 to 255, I split the pixels into two classes and calculated the between-class variance from the weights and means of the two classes. The maximum was at T = 107, the same as OpenCV, and the binary image I made with it was exactly the same as OpenCV's. The maximum between-class variance was 2115.1 out of a total variance of 2796.3, so the split explains about 75.6% of the variance in the image.

## Comparison with Other Thresholds

I also compared Otsu with the fixed threshold T = 127 that the lab used for simple thresholding, and with Otsu after a 5 × 5 Gaussian blur:

- The fixed threshold left more black holes inside the darker coins and gave 119 separate white objects
- Otsu with T = 107 kept more of each coin and gave 96 white objects
- Gaussian blur then Otsu chose T = 104, removed most of the noise and gave solid coins with only 34 white objects

## Observations

The most surprising thing was how close the hand-made calculation was to the library. It is only a few lines with cumulative sums over the histogram, and it gave the exact same threshold and the exact same image.

I also noticed that Otsu is not perfect on this image. The brighter background at the top became white and joined the top row of coins, because Otsu uses one global threshold for the whole image and cannot handle uneven lighting.

Finally, the blur made a big difference. The noise inside the coins created many small black spots, and smoothing the image first made the histogram cleaner and the coins almost solid.

## Conclusion

This lab showed that Otsu's method is a simple and reliable way to choose a threshold automatically, as long as the histogram has two clear groups like the coins and the background here. It picks the threshold that separates the two classes the most, and it gave a better result than the fixed threshold of the lab. However, as a global method it struggles with uneven lighting, and it works best when the noise is removed first with a Gaussian filter.
