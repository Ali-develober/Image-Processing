## ARTI403 – Lab 8
## Edge Detection

In this lab, I worked on edge detection, which finds the places in an image where the intensity changes sharply. I applied first-derivative operators (Sobel, Roberts and Prewitt), the Canny detector on gray and colour images, second-derivative filters (LoG and DoG) and a gradient detector for colour images. The main objective was to understand how each detector works and when one is better than another.

## Images Used

The lab came as one notebook with four images and a task sheet. The notebook used a photo of two kids for every detector, but the task says not to use the image provided in the notebook, so I replaced it everywhere with my own image. The `images` folder of this lab has:

- `zebra.jpg`, a 600 × 556 colour photo of a zebra from Wikimedia Commons, which I chose and used for all the main sections and the tasks
- `bird.png`, a gray photo of a bird on a branch
- `Butterfly.jpg`, a colour photo of a monarch butterfly with flowers and leaves
- `Coins.png`, ten coins on a dark background inside a white frame

## First-Derivative Detectors

- Sobel uses two 3 × 3 kernels for the x and y gradients, and I combined them with `cv2.magnitude()` after a 5 × 5 Gaussian blur
- Roberts uses two 2 × 2 diagonal kernels, so it is fast but very sensitive to noise
- Prewitt is like Sobel but gives the same weight to all three rows

## Canny

I ran `cv2.Canny()` with thresholds of 100 and 200 on the gray image of the zebra. I also ran the multi-channel version, which applies Canny to the B, G and R channels separately and joins the three edge maps with a bitwise OR, so it keeps edges that exist only in the colour.

## LoG, DoG and Colour Gradient

- The LoG blurs the image and then applies the Laplacian, which is a second derivative, so an edge is where the result crosses zero
- The DoG subtracts two Gaussian blurs with sigma 1 and 2, which gives almost the same result as the LoG with less computation
- The colour gradient computes Sobel on each channel and combines the three magnitudes into one map

## Task 1: Sobel, Prewitt and Canny on a New Image

The task asked for an image that is not the one in the notebook, so I chose a zebra photo from Wikimedia Commons that I did not use in any lab before. Its black and white stripes have very strong edges, while the bushes and the grass behind it have weaker ones. I followed the same steps as the notebook: gray, a 5 × 5 Gaussian blur, then Sobel and Prewitt combined with the magnitude, and Canny with thresholds of 100 and 200.

Sobel and Prewitt gave almost the same map, where every stripe became a thick bright edge and the background became a weak gray texture. Prewitt was a little darker, with a maximum of 347 against 464 for Sobel, because its kernels do not give a weight of 2 to the middle row. Canny gave thin white lines on a black background, which made every stripe and the outline of the zebra sharp.

## Task 2: Effect of the Canny Thresholds

I ran Canny with the three threshold pairs and displayed them side by side, with a zoom on the head under each one:

- With (30, 90), 93,085 pixels were edges, which is 27.9% of the image
- With (50, 150), it dropped to 70,541 pixels (21.2%)
- With (100, 200), only 50,267 pixels were left (15.1%)

Canny keeps a pixel above the high threshold as a strong edge, drops anything below the low threshold, and keeps the pixels in between only when they connect to a strong edge. So raising the thresholds removes the weak edges first. With the low pair the bushes and the soil were full of edges and the muzzle of the zebra was lost in them. With the high pair the bushes almost disappeared and the stripes, the ears and the outline of the head stayed clear, while the middle pair was a balance between the two.

## Extra Experiments

After the tasks, I tested the detectors further:

- I tried Sobel on the original, gray and blurred image, as the comment in the notebook suggests. On the colour image, each channel gives its own edges, and blurring reduced the strong pixels from 27.9% to 15.7%, removing most of the grass texture
- I found that `np.uint8()` does not clip, so values above 255 wrap around and become dark. On the zebra this affected 4.5% of the Sobel image and 36.6% of the LoG image, and clipping first fixed it
- I checked the Roberts vs Sobel comparison on a synthetic step edge. Roberts gave a 1-pixel edge and was the fastest, but it had the lowest edge-to-noise ratio (9.6 against 11.6 for Sobel and 12.3 for Prewitt)
- I changed the Canny thresholds with and without blur. The edge pixels dropped from 21.1% with (50, 150) and no blur to 3.8% with (150, 300) and blur, where only the stripes and the outline were left
- On the butterfly, the multi-channel Canny found 10,272 more edge pixels than the gray Canny, around the pink flower, the leaves and the orange wing cells
- I ran every detector on the bird, butterfly and coins images
- I computed the LoG zero crossings, which gave thin single-pixel edges, and displayed the DoG binary map that the notebook calculates but never shows
- I counted the coins with the Hough circle transform, which uses Canny edges inside, and it found all 10 coins

## Observations

The most surprising thing was the LoG output in the notebook. On the zebra it looked like random noise and the zebra almost disappeared, and this was not a property of the filter. It was the same overflow problem I found in Lab 7, because the values reached 2,722 and `np.uint8()` keeps them modulo 256.

I also noticed that the comparison table in the notebook was correct on every point, but Prewitt came out slightly more noise-resistant than Sobel on Gaussian noise. Also, the DoG binary map split each stripe edge into a black and a white side, because the threshold of 127 is exactly the zero level of the normalized DoG.

Finally, the bird image is saved as a colour PNG, but all three of its channels are identical, so the multi-channel methods give the same result as the gray ones on it.

## Conclusion

This lab showed that each edge detector is a trade-off between detail and noise, and the Canny thresholds control that trade-off directly: lower thresholds show more detail, higher thresholds keep only the main outlines. Roberts is the fastest and gives the thinnest edges, Sobel and Prewitt resist noise better, and Canny gives the cleanest result because it smooths, thins and links the edges, as long as the thresholds and the blur are chosen well.

Colour also matters. Converting to gray loses the edges between colours of the same brightness, so the multi-channel and colour gradient methods found edges that the gray detectors missed.
