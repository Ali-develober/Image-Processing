## ARTI403 – Lab 6
## Filtering in the Frequency Domain

In this lab, I worked on filtering in the frequency domain. Instead of sliding a kernel over the image, I moved the image to the frequency domain with the Fast Fourier Transform, multiplied it by a filter and came back with the inverse FFT. The main objective was to see that this gives the same result as a convolution in the spatial domain, and to understand what an edge filter looks like in terms of frequencies.

## Image Used

I used the cameraman from `skimage.data`, a 512 × 512 grayscale image, as the manual asks. I saved a copy as `camera.png` in the `images` folder of this lab and the notebook reads it from there, and I checked that it is identical to `data.camera()`. Following the manual, I resized it to 256 × 256, which also converts it to floating point values between 0 and 1.

## Laplacian in the Frequency Domain

I started with the code of the manual. It builds a grid of frequencies with `np.fft.fftfreq()`, from -0.5 to about 0.5 cycles per pixel, and defines the Laplacian filter as -4π²(u² + v²). The image is transformed with `np.fft.fft2()`, multiplied by the filter and transformed back with `np.fft.ifft2()`.

The filter is 0 at the zero frequency and becomes more negative as the frequency grows, down to -19.7 in the corners. So it removes the average brightness completely and amplifies the high frequencies, which are the edges and the fine detail. The output image was gray everywhere with the edges of the man, the tripod and the buildings drawn as thin lines, with values from -2.04 to 1.94 and a mean of exactly zero.

The imaginary part left after the inverse FFT was around 10⁻¹⁵, only rounding error, so taking `.real` loses nothing.

I also compared this filter with the frequency response of the classic 3 × 3 Laplacian kernel. The two curves are the same near the zero frequency and separate at high frequencies, where the kernel stops at -4 while the formula keeps going. The two output images still had a correlation of 0.97. Finally, I subtracted a quarter of the Laplacian from the image and got a sharper version of the cameraman.

## Sobel Filters in the Frequency Domain

For the task I applied the two Sobel kernels to the same resized cameraman. I used the function from the hints: it puts the 3 × 3 kernel in the middle of an empty 256 × 256 array and then uses `ifftshift()` to move its center to position (0, 0) before the FFT. Without this step the result would be shifted.

I multiplied the spectrum of the image by the spectrum of each kernel, came back with the inverse FFT and combined the two results into the gradient magnitude. I displayed the original, the spectra of the two filters, the x and y results and the magnitude.

- Sobel x showed the vertical edges, such as the tripod legs and the side of the coat
- Sobel y showed the horizontal edges, such as the camera and the line of the horizon
- The magnitude showed all the edges together, with values from 0 to 3.45

The spectrum of Sobel x is dark along a vertical line through the center and bright on both sides, so it blocks every change along y and keeps the changes along x. The Sobel y spectrum is the same picture turned by 90 degrees. Both spectra are 0 at the zero frequency and reach a peak of 8.

To check the result, I computed the same filters with a normal spatial convolution using `convolve2d()` with wrap-around borders. The largest difference was about 3 × 10⁻¹⁵, so the two methods gave the same image.

## Observations

The most interesting thing was seeing that a convolution and a multiplication of spectra really give the same image, down to rounding error. The only condition was that the borders wrap around, because the FFT treats the image as if it repeats forever. This is why thin bright lines appear along the borders of the filtered images, where the top of the image meets the bottom.

I also noticed that the spectra of the Sobel kernels have no real part at all. The kernels are antisymmetric, so their Fourier transforms are purely imaginary.

Finally, both the Laplacian and the Sobel filters remove the zero frequency, because their weights sum to 0. This is why their outputs have a mean of zero and look gray in the flat regions.

## Conclusion

This lab showed that filtering can be done in two equivalent ways. In the spatial domain, a small kernel slides over the image, while in the frequency domain, the whole image is multiplied by the transform of that kernel at once.

The frequency domain also makes the behavior of a filter easy to read. The Laplacian is a high-pass filter that grows with the frequency in every direction, while each Sobel filter is a high-pass filter in one direction only, which is why it finds edges with a specific orientation.
