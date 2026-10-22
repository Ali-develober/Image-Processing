## ARTI403 – Lab 7
## Laplacian Filter and Mathematical & Statistical Filters

In this lab, I worked on two kinds of spatial filters. The first part was the Laplacian, a second-derivative filter that finds edges and can be used to sharpen an image. The second part was image restoration: I added four types of noise to an image and tried to remove them with the mean filters (arithmetic, geometric, harmonic and contra-harmonic) and the order-statistic filters (median, max, min and midpoint). The main objective was to understand which filter suits which noise and why.

## Images Used

The lab came as two notebooks, 7(A) and 7(B), with four images, which I put in the `images` folder of this lab:

- `moon.jpg`, a 438 × 401 photo of the moon, for the Laplacian part
- `noise.jpg`, a clean synthetic image with a dark background, a gray square and a bright disc, to add the noise to
- `salt&Pepper.jpg` and `salt&Pepper2.jpg`, two real images that already have impulse noise

## Laplacian Filter

I applied the 3 × 3 Laplacian kernel with `cv2.filter2D()`, using `CV_64F` so the negative values are kept. The weights of the kernel sum to 0, so flat regions give 0 and only the edges and the craters respond, with values from -123 to 110. I then took the absolute value with `convertScaleAbs()` and added it to the original.

I also ran the second version that blurs the image with a 3 × 3 Gaussian first and then calls `cv2.Laplacian()`. The blur reduced the range to -66 to 53, so fewer tiny details were picked up as edges. I checked that `cv2.Laplacian()` without blur gives exactly the same result as the manual kernel.

## Adding Noise

I generated four noisy versions of `noise.jpg`: Gaussian noise with a variance of 0.01, salt & pepper noise on 4% of the pixels, exponential noise with a scale of 10 and uniform noise between -20 and 20. I fixed the random seed so the results are the same every time. The histograms of the gray square showed the shape of each noise: a bell for Gaussian, a flat block for uniform, a one-sided tail for exponential and two spikes at 0 and 255 for salt & pepper.

## Mean Filters

- The arithmetic mean with `cv2.blur()` and with a manual kernel of ones divided by 9 gave exactly the same image
- The geometric mean is the exponent of the average of the logarithms
- The harmonic mean is the window size divided by the sum of the reciprocals
- The contra-harmonic mean uses a parameter Q, where Q > 0 removes pepper and Q < 0 removes salt

## Order-Statistic Filters

- The median filter with `cv2.medianBlur()` replaces each pixel with the middle value of its window
- The max filter (`cv2.dilate()`) keeps the brightest pixel and the min filter (`cv2.erode()`) the darkest
- The midpoint filter is the average of the max and the min

## Extra Experiments

The notebooks have no written tasks, so I tested the filters further:

- I fixed the sharpening of the moon. Adding two uint8 images wrapped 541 pixels past 255 into black specks, so I computed the original minus the Laplacian in float and clipped it, and also tried the diagonal kernel with -8 in the center, which sharpened more
- I applied every filter to every noise type and measured the PSNR against the clean image. The arithmetic mean was best on Gaussian noise (30.0 dB), the median on salt & pepper (49.6 dB), the min filter on exponential noise (33.5 dB) and the midpoint on uniform noise (37.1 dB)
- I added salt only and pepper only. With salt, Q = -1.5 reached 37.3 dB and Q = 1.5 dropped to 10.4 dB, and with pepper it was the opposite
- I changed the window size from 3 × 3 to 9 × 9. The mean kept improving on Gaussian noise until 5 × 5, but always blurred the edges, while the median kept them sharp
- I cleaned the two real salt & pepper images. The first one needed a 5 × 5 median, but the dots in the second one are about 16 pixels wide, so an 11 × 11 window did nothing and only a 35 × 35 median removed them

## Observations

The most surprising thing was that the manual's sharpened moon had black specks on the brightest areas. This was not a property of the Laplacian but an overflow, because NumPy adds uint8 values modulo 256.

I also noticed that the manual runs every filter on the Gaussian noise only, even where the titles say salt & pepper. With the right noise, the differences became very clear. On salt & pepper, the geometric and harmonic means made the image worse than the noisy one, because a single 0 in the window pulls the result to black.

Finally, a filter with the wrong sign is worse than no filter. The max filter on salt and the min filter on pepper turned every dot into a 3 × 3 square.

## Conclusion

This lab showed that there is no single best denoising filter. The Laplacian is a high-pass filter that highlights edges and sharpens an image when it is subtracted from it. For noise removal, averaging filters work on noise that spreads around the true value, while order-statistic filters work on noise that produces extreme values.

The median was the most reliable filter overall, since it removed impulse noise completely and kept the edges sharp, as long as its window was more than twice as wide as the noise.
