## ARTI403 – Lab 9
## Morphological Operations

In this lab, I worked on morphological image processing, which changes the shape of the objects in a binary image using a small kernel called the structuring element. I applied erosion, dilation, opening and closing, and I extracted the boundaries of an object with the morphological gradient and with the internal and external boundaries. The main objective was to understand what each operation does to the objects, and how the shape and the size of the structuring element change the result.

## Images Used

The lab came as one notebook with five images and a task sheet. I moved the images into the `images` folder of this lab:

- `Erosion.jpg`, a chip with thin wires, used for erosion and dilation
- `Open_Closing.jpg`, white blobs with small noise, used for opening
- `Opening2.jpg`, a white shape with gaps, used for closing
- `BoundaryExtraction.jpg`, the silhouette of a head, used for the boundaries
- `Erosion2.jpg`, the alphabet in black letters, which the notebook ships but never uses, so I used it in the extra experiments
- `balls.jpg`, a 960 × 720 photo of five silicon nitride bearing balls from Wikimedia Commons, which I chose for the tasks

## Erosion and Dilation

- Erosion keeps a white pixel only when the whole kernel fits inside the white region, so the objects shrink. On the chip, the thinnest wires disappeared with 3 × 3 and the thicker ones with 7 × 7 and 11 × 11, until only the thick middle wire was left
- Dilation turns a pixel white when the kernel touches any white pixel, so the objects grow and the small gaps are filled
- A bigger kernel gives a stronger effect, and the notebook used 3 × 3, 7 × 7 and 11 × 11 squares

## Opening and Closing

- Opening is an erosion followed by a dilation. With a 5 × 5 kernel it removed the small white dots and kept the bigger blobs, but their round corners became a little square because of the square kernel
- Closing is a dilation followed by an erosion. With a 45 × 45 kernel it filled the slot on the left, the small black square in the middle and the notches on top of the shape. The black square on the right stayed, because the gap around it is wider than the kernel

## Boundary Extraction

I extracted the outline of the head in three ways: the morphological gradient (dilation minus erosion), the internal boundary (the image minus its erosion) and the external boundary (the dilation minus the image). The gradient gives the thickest outline, because it combines the other two.

## Task 1: Morphological Operations on a New Image

The task suggested balls, so I chose a photo of bearing balls of five sizes. They are dark on a white background, so I binarized the photo with Otsu and an inverted threshold to make the balls white. The binary image was not clean: it had 62 white objects, which are the 5 balls and 57 small specks from the shadows, and 58 black holes inside the balls from the reflections.

- Erosion removed the specks and made the holes bigger. With 11 × 11 only 5 objects were left, but the smallest ball, about 14 pixels wide, almost disappeared
- Dilation made the balls bigger and reduced the holes from 58 to 4, but it also joined the specks near the balls to them
- Opening 5 × 5 removed all the specks and kept exactly the 5 balls with almost the same area
- Closing 21 × 21 filled every hole inside the balls. The big reflection on the large ball was not filled, because it reaches the edge of the ball, so it is a gap in the outline and not a hole

## Task 2: Shapes and Sizes of the Structuring Element

I created the square, cross and ellipse with `cv2.getStructuringElement()` in sizes 3 × 3, 5 × 5 and 7 × 7, and applied erosion and dilation with all nine of them on the balls image. At 7 × 7 the square has 49 ones, the ellipse 33 and the cross 13, so for the same size the square had the strongest effect and the cross the weakest. Erosion 7 × 7 kept 72,429 white pixels with the square, 74,467 with the ellipse and 75,678 with the cross, and dilation gave the opposite order.

The bigger the kernel, the stronger the effect for every shape. The shape changed the look of the result: the square made the outlines blocky, the ellipse kept the balls round, and the cross missed the diagonals, so it split the shadow specks into pieces instead of removing them. The difference images showed that the shapes only differ on the edges of the balls and the holes.

## Extra Experiments

After the tasks, I tested the operations further:

- On `Erosion2.jpg`, the threshold of the notebook made the white background the foreground, so an erosion made the letters thicker. With an inverted threshold, erosion made the letters thin and dilation joined them together
- Erosion with a 3 × 3 square repeated 3 times gave exactly the same result as one 7 × 7 erosion, and 5 times the same as 11 × 11. This does not work for the cross, which grows into a diamond
- I checked that opening and closing are idempotent, that opening only removes pixels and closing only adds them, and that eroding the white is the same as dilating the black
- I counted the balls after an elliptical opening and closing and got exactly 5 objects instead of 62, and their diameters went from 282 down to 14 pixels

## Observations

The most surprising thing was that OpenCV's 3 × 3 ellipse is exactly the same as the 3 × 3 cross, so they gave the same numbers in Task 2. The 5 × 5 ellipse is also not very round, and the ellipse only looks like a circle from 7 × 7 up.

I also noticed that the polarity of the binary image decides everything. The notebook always uses `THRESH_BINARY`, which works for its images because their objects are white, but on black letters or dark balls it makes the background the foreground and every operation does the opposite of its name.

Finally, the dilation cell of the notebook still names its results `eroded_img` and its comments say "Apply erosion", and the closing cell says "Apply Opening", because they were copied from the cell before. The results are correct, only the names are wrong.

## Conclusion

This lab showed that morphological operations are simple min and max filters on a binary image, but together they are powerful: opening removes small objects and closing fills small holes without changing the size of the main objects much. The structuring element controls the result: its size decides how big the removed or filled details are, and its shape decides how the outlines look, so a round object like a ball is best processed with an ellipse.

Using the balls photo also showed how these operations are used in practice. A simple Otsu threshold gave 62 objects, and one opening and one closing were enough to get exactly the 5 balls and measure them.
