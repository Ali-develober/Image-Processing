## ARTI403 – Lab 4
## Intensity Transformations and Filtering in the Spatial Domain

In this lab, I worked on two families of intensity transformations: thresholding, which turns a grayscale image into a binary one, and histogram processing, which changes the distribution of the gray levels to improve the contrast. The main objective was to understand how the histogram of an image describes its contrast, and how reshaping it changes the image.

## Images Used

For the thresholding I used `Parrot.png`, a 453 × 340 color photograph of a macaw from the Hands-On Image Processing with Python book, which the manual refers to but does not include. I loaded it in grayscale.

For the histogram processing I used three images from `skimage.data`: the moon, a 512 × 512 grayscale image with very low contrast, the rocket as the reference image, and Chelsea the cat as the source image. All four images are saved in the `images` folder of this lab.

The manual reads the parrot from `../images/`, outside the lab folder. Since the submission requires the images to be inside the lab folder, I changed the path to `./images/` and kept the original line as a comment.

## Thresholding

I applied `cv2.threshold()` with the five fixed values from the manual. Instead of opening a separate window for each result, I displayed them all in one figure.

The results showed how sensitive thresholding is to the chosen value:

- At 0, every pixel was white, so the threshold separated nothing
- At 50, still 99% of the image was white
- At 100, about 63% was white and the parrot's dark feathers started to separate from the background
- At 150, only 19% was white, mostly the face of the parrot, the beak and the bright background spots
- At 200, only 3% remained, the brightest highlights

I also plotted the histogram with the thresholds on it and tried Otsu's method, which chose a threshold of 128 automatically from the histogram, between the two thresholds that looked best to me.

## Contrast Stretching

The histogram of the moon showed that almost all its pixels were squeezed into a narrow band: the 2nd percentile was 78 and the 98th was 129. This is why the image looks gray and flat.

Using `exposure.rescale_intensity()`, I mapped that range onto the full 0 to 255 range. The histogram spread out over the whole axis and the craters became clearly visible. The 2% darkest and brightest pixels were clipped to 0 and 255, which is the price of this method.

## Stretching Between the 3rd and 80th Percentiles

For the first task I repeated the stretching with the 3rd and 80th percentiles, which were 87 and 118. Because this range is narrower, the mid tones gained even more contrast than with the 2 to 98 stretch.

However, the 80th percentile is a strong cut. In total 16.5% of the pixels were above it and all became 255, which showed up as a tall spike at the right end of the histogram and as burned out white areas on the surface of the moon. Only 2.8% were clipped at the dark end.

## Histogram Equalization

For the second task I used `exposure.equalize_hist()`. It returns floating point values between 0 and 1, not 0 to 255, so the manual's histogram code with a range of 0 to 255 would have shown everything in the first bin. I converted the result back to 8-bit with `img_as_ubyte()` before plotting.

The standard deviation went from 13.3 to 73.9, so the contrast increased more than five times. The cumulative histogram of the equalized image became almost a straight line, never more than 0.085 away from the ideal uniform line.

The histogram itself was not flat but made of separate bars with gaps between them. This is because all the pixels sharing one gray level are mapped together, so equalization can move the bars apart but it can never split one.

## Histogram Matching

For the third task I used `match_histograms()` with Chelsea as the source and the rocket as the reference, matching each color channel separately. The matched cat kept all its shapes and edges, but took the dark, bluish colors of the rocket image.

The numbers confirmed it. The mean of the red channel went from 147.7 in the source to 52.1 after matching, almost the 52.3 of the reference, and the green and blue channels followed in the same way. The cumulative histograms of the matched image and of the reference differed by less than 0.02 on every channel. The method works even though the two images have different sizes, because it only uses their histograms.

## Observations

The most surprising thing was how much a histogram tells about an image. Just by looking at the moon's histogram, squeezed between 78 and 129, it was clear why the image looked flat, and every method in this lab was simply a different way of reshaping that histogram.

I also noticed that each method has a cost. Fixed thresholding depends completely on the chosen value, contrast stretching clips the extreme pixels, and equalization leaves gaps in the histogram and can over enhance noise in flat regions.

## Conclusion

This lab showed that thresholding and histogram processing are both point operations, where each output pixel depends only on the value of the same input pixel. Thresholding reduces the image to two levels to separate objects, while histogram processing spreads the gray levels to improve contrast.

Contrast stretching gives control over which part of the range is enhanced, histogram equalization works automatically and increased the moon's contrast more than five times, and histogram matching transfers the tone of one image onto another.
