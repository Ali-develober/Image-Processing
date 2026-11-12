## ARTI403 – Lab 9
## Morphological Operations

In this lab, I worked on morphological image processing, which changes the shape of the objects in a binary image using a small kernel called the structuring element. I solved the two tasks of the lab: applying erosion, dilation, opening and closing to a new binary image, and comparing structuring elements of different shapes and sizes. The main objective was to understand what each operation does to the objects, and how the structuring element changes the result.

## Image Used

I used one image for both tasks, `Balls.png` in the `images` folder. It is a clean black and white drawing of the fifteen billiard balls racked in a triangle, where each ball is a black ring with a white circle and its number in the middle. The picture has a black frame, so I cut it off first, which left a 1224 × 1173 image.

The balls are black on a white background, so I binarized the image with Otsu and an inverted threshold to make the balls white. In the binary image:

- All 15 balls touch each other, so the rack is one big white object
- The 21 digits are separate white objects inside the black number circles
- There are 38 holes: the 15 number circles, the 16 small gaps between the balls and the 7 holes inside the digits 4, 6, 8, 9 and 0

## Task 1: Erosion, Dilation, Opening and Closing

I applied the four operations with a 21 × 21 square kernel. The image is large, so a 3 × 3 kernel barely changes it, while 21 × 21 is bigger than the thickness of the digit strokes.

- Erosion made the white rings thinner and removed all the digits, and the balls started to separate at the points where they touch
- Dilation made the rings thicker, filled the gaps between the balls and made the digits grow into blobs that fill most of the number circles
- Opening (erosion then dilation) removed all the digits and kept the rings at almost the same thickness, so only the clean rack was left as one object
- Closing (dilation then erosion) filled the 16 gaps between the balls, but it also joined the digits to the rings, which split some of the number circles into smaller holes

I then repeated the four operations with 11 × 11, 21 × 21 and 35 × 35 kernels. The effect grew with the kernel size, and with 35 × 35 erosion broke the rings apart while dilation and closing turned the rack into an almost solid shape with only a few holes left.

## Task 2: Shapes and Sizes of the Structuring Element

I created the square, cross and ellipse with `cv2.getStructuringElement()` in sizes 3 × 3, 5 × 5 and 7 × 7, and applied erosion and dilation with all nine of them on the same binary image. At 7 × 7 the square has 49 ones, the ellipse 33 and the cross 13, so for the same size the square had the strongest effect and the cross the weakest:

- Erosion 7 × 7 kept 368,888 white pixels with the square, 384,130 with the ellipse and 389,618 with the cross
- Dilation 7 × 7 gave 511,907 white pixels with the square, 498,120 with the ellipse and 493,261 with the cross

With these small kernels the number of objects stayed at 22 and the holes changed by at most 3, because the image is clean and large, so the kernels only moved the outlines by a few pixels. To see the difference, I zoomed on balls 8 and 9. The square made the corners of the gaps and the digits blocky, the ellipse kept them round, and the cross changed the outlines the least, only in the horizontal and vertical directions, because it has no diagonal. The difference images showed that the shapes only differ on the edges.

## Observations

The most surprising thing was that OpenCV's 3 × 3 ellipse is exactly the same as the 3 × 3 cross, so they gave the same numbers in Task 2. The 5 × 5 ellipse is also not very round, and the ellipse only starts to look like a circle at 7 × 7.

I also noticed that the right kernel size depends on the size of the image. The 3 × 3 to 7 × 7 kernels of Task 2 changed only the edges of this 1224 × 1173 image, while 21 × 21 was needed to remove the digits or fill the gaps.

Finally, closing did not only fill holes as I expected. It filled the gaps between the balls, but it also joined the digits to the rings, so the number of holes went up from 38 to 46 instead of down.

## Conclusion

This lab showed that morphological operations are simple operations on a binary image, but together they are powerful: opening removes objects that are thinner than the kernel, like the digits, and closing fills gaps that are smaller than the kernel, like the spaces between the balls. The structuring element controls the result: its size decides how big the removed or filled details are, and its shape decides how the outlines look, so round objects like the billiard balls are best processed with an ellipse.
