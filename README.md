# negatives/positive

**Convert your digital film negatives (linear TIFF files) to positive images,
allowing basic image editing.**

 These utilities are used:

- [GNU Parallel:](https://www.gnu.org/software/parallel/) Use all CPU cores for maximum speed
- [ImageMagick:](https://www.imagemagick.org/script/index.php)  Negative inversion, B/W grayscaling, contrast enhancements, resizing, ZIP or JPEG compression

*These first releases of this package are developed and tested with B/W negatives.*



Requirements:

- [Homebrew](https://brew.sh/) Package manager for OS X
- [tomkyle/homebrew-negatives](https://github.com/tomkyle/homebrew-negatives) Homebrew tap for negatives-related scripts.

## Homebrew Installation (OS X)


The *positive* bash script can be installed by a Homebrew formula, which itself is part of the [tomkyle/homebrew-negatives](https://github.com/tomkyle/homebrew-negatives) tap. 

```bash
# Install tap, optionally
$ brew tap tomkyle/negatives

# Install formula
$ brew install positive
```

As “tapping” first is not neccessarily needed, you can install the formula directly:

```bash
$ brew install tomkyle/negatives/positive
```


# Usage

Open your terminal application and go to your images directory. *positive* will work in the current working directory.

```bash
$ positive [options] [-a | file(s)]
```

## Options

Run *positive* without parameters to get a short help text.

### Files and output

Option | Value | Description
:------|:------|:------------
-a     | | All images (batch mode). Process any TIFF file in working directory.
-f     | value | Mirror the image vertically with `flip`, or horizontally using `flop`. *flop* is useful when you photographed the emulsion side of your negatives. Example: `-f flop`
-j     | quality | Save image as JPG with given quality. Example: `-j 90`
-o     | path  | Output directory. Default is current working dir. Example: `-o positives`
-r     | pixel | Resize larger side to this pixel length, preserving aspect ratio. Example: `-r 3000`
-v     |       | Turn on verbous mode

### Color and Contrast

Option | Value | Description
:------|:------|:------------
-d     | | Desaturate colors, recommended for B/W negatives. Bonus: The TIFF will be converted to 16 bit Grayscale, saving up to 60% in file size; The image will have linear gamma 1.0 ICC profile applied (Gray-elle-V4-g10.icc).
-g     | gamma | Gamma correction value to apply. It is highly recommended to use this parameter with linear TIFF images, together with **-n**. Example: `-g 2.2`
-n     |       | Normalize: Stretch histogram to reach black and white points. Recommended for most images.
-s     | value | Sigmoidal contrast value around 50% middle-gray. Increases the contrast without saturating highlights or shadows. Quoted from ImageMagick docs: “3 is typical and 20 is a lot.” Example: `-s 5`


## Examples

### Single mode: Turn some TIFF files into positive.
Given your photographed negative is a B/W image, we use desaturation.

The result will be a grayscale TIFF with approriate ICC profile. *Please note that the positive image gets a `-positive` suffix when it is stored in the same directory like the original image.*

```bash
$ positive -d DSC_0123.tif
```

```
------------------------------------------
DSC_0123.tif
Create positive image: Done.
Result: DSC_0123-positive.tif
```

Again, we use desaturation, and this time the positives go into a subdirectory. The results are now stored in a `foobar` directory (Note: no `-positive` suffix).

```bash
$ positive -d -o foobar DSC_0123.tiff DSC_0124.tiff DSC_0125.tiff
```

```
------------------------------------------
DSC_0123.tiff
Create positive image: Done.
Result: foobar/DSC_0123.tiff

------------------------------------------
DSC_0124.tiff
Create positive image: Done.
Result: foobar/DSC_0124.tiff

------------------------------------------
DSC_0125.tiff
Create positive image: Done.
Result: foobar/DSC_0125.tiff
```


### Batch mode: Convert *all* TIFF files in the current directory

Just pass `-a` parameter, and the script will mangle everything that has a TIFF file extension: `tif, tiff, TIF, TIFF` etc.

The batch mode uses [GNU Parallel,](https://www.gnu.org/software/parallel/) so every single CPU core will be used. So if you have a Quadcore CPU, four images will be processed at the same time. Great timesaver!

Since normalizing is a good idea, we use the `-n` option in addition to desaturation with `-d`.

```bash
$ positive -a -d -n -o results
```

```
------------------------------------------
Process 3 images, using GNU Parallel:

------------------------------------------
DSC_0123.tiff
Create positive image: Done.
Result: foobar/DSC_0123.tiff

------------------------------------------
DSC_0124.tiff
Create positive image: Done.
Result: foobar/DSC_0124.tiff

------------------------------------------

(...and many more...)

------------------------------------------
Some stats:
CPUs used:        4
Elapsed time:     0min 30sec
Done:             16 images

```



## Tweaking colors

If you are working on linear TIFFs, e.g. as produced by [Dave Coffin's dcraw](http://cybercom.net/~dcoffin/dcraw/dcraw.1.html) by its `-4` flag, both the negative and the positive will look somehow flat, due to their linearity. The dark negative becomes a very light positive. 

The *positive* utility inverses the negative TIFF to positive, and exactly now we are able to perform the very gamma correction that did not take place when `dcraw` created the linear TIFF. Enter `-g gamma` parameter!

### Gamma correction

The `-g gamma` parameter carries the Gamma correction value to apply. It is highly recommended to use this parameter with *-n*. 

(In fact, the light positive must be darkened, using a gamma value lesser than 1. Since such small gamma values are “uncommon” in human image editing, we use “normal” `-g 2.2` here. Internally, the script then calculates the reciprocal value (0.45) for darkening.)


```bash
positive -a -d -n -g 2.2 -o gamma-corrected
```
 
**The gamma values you choose are completely up to your taste and, moreover, the negative density as a result of your film development workflow!** Examples for typical values are:
 
gamma | description
:-----|:----------
1.0 | No Gamma for linear TIFFs. This is the default value if omitted.
1.8 | Moderate darkening, resulting in overall light images
2.2 | Well-known software standard; This is what common photo software would apply.
3.6 | Smooth tones, not too light
4.4 | Somehow darker, still mellow. Not too contrasty.
... | Try your own!


Read more about Gamma correction: [Wikipedia](https://en.wikipedia.org/wiki/Gamma_correction) [ImageMagick](http://www.imagemagick.org/Usage/color_mods/#level_gamma)

### Sigmoidal contrast 

While the gamma correction mainly affects the midtones, the sigmoidal contrast control works on the highlights and lighter shadows, leaving the 50% midtone alone. With the `-s value` parameter you can apply a s-like curve to enhance the contrast.

Quoted from ImageMagick docs: “3 is typical and 20 is a lot.” – Example:

```bash
positive -a -d -n -g 3.6 -s 5 -o nice-contrasts
```

Read more: [ImageMagick](http://www.imagemagick.org/Usage/color_mods/#sigmoidal)






# Workflow Recommendations

## When to crop images?

When photographing your negatives, you'll probably use a negatives (film) holder. 

- The edges of your RAW photo usually will then have a black frame.  
- If the window of your negative holder is larger than the negative, your image will also show unlit film areas (i.e., spaces between the single negative frames), resulting in another “white“ frame.

### Valid approaches

The following workflow examples assume your linear TIFFs are framed black and white, having used a negative holder with larger frame window.

#### Approach: Crop first

Start cropping your images and save them as TIF. Be sure to preserve the ICC profile!

Run *positive* with gamma and sigmoidal contrast as needed. When using the `-n` parameter for normalizing colors, both the black and white points will be determined by *the information in your image.* Consequences are: 

0. The higher gamma value or sigmoidal contrast you use, the earlier deepest shadows and lights will start blocking.
0. Underexposured or overexposured images are normalized and may loose their “natural” look. Flat images may show more film grain.


#### Approach: Do not crop

Run *positive* with gamma and sigmoidal contrast as needed. When using the `-n` parameter for normalizing colors, both the black and white points will be determined by *the darkest grays of the negative holder edges and the lightest parts in the unlit negativ areas.* Deepest shadows and lights are now in the (surfluous) frames. Consequences are: 

0. You may choose way higher gamma and sigmoidal contrast values.
0. The look of underexposured or overexposured images will overall stay natural.








# Problems and FAQ

###Message “mogrify: delegate library support not built-in”

`mogrify: delegate library support not built-in './foobar.tiff' (LCMS) @ warning/profile.c/ProfileImage/837.`

*mogrify* is part of ImageMagick, an the *delegate library* is part of [LitteCMS](http://www.littlecms.com/). Your local ImageMagick must be compiled *with little-cms2* support. Reinstall imagemagick according to this [solution](https://github.com/Homebrew/legacy-homebrew/issues/16619):

```bash
$ brew remove imagemagick
$ brew install little-cms2 imagemagick --with-little-cms2
```

