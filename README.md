# negatives/positive

**Convert your digital film negatives (linear TIFF files) to positive images.** Other features:

- **Gamma correction** for adjusting midtones
- **Sigmoidal contrast** for highlights and shadows adjustment
- **Grayscaling:** B/W lovers save up to 60% disk space. 
- **Resizing**  when megapixels are not everything
- **Zipped TIFF or JPEG conversion** 

**What happens inside?** [ImageMagick](https://www.imagemagick.org/script/index.php)  does the image editing. [GNU Parallel](https://www.gnu.org/software/parallel/) mangles multiple images in parallel, depending on your CPU cores.

*These first releases of this package are developed and tested with B/W negatives.*



## Homebrew Installation (MacOS)


The *positive* bash script can be installed by a [Homebrew](https://brew.sh/) formula, which itself is part of the [tomkyle/homebrew-negatives](https://github.com/tomkyle/homebrew-negatives) tap. 

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

Open your terminal application and go to your images directory. *positive* will work in the current working directory. Run `positive --help` or `-h` to display help text. 


```bash
$ positive [options] [-a | file(s)]
```

See [Options](#options) · [Examples](#xamples) · [Gamma correction](#gamma-correction) · [Sigmoidal contrast](#sigmoidal-contrast) · [Workflow Recommendations](#workflow-recommendations)

## Options

Run *positive* without parameters to get a short help text.

### Files and output


#### -a, --all
All images (batch mode). Process any TIFF file in working directory.

#### -f, --flipflop *direction*  
Mirror the image vertically and/or horizontally. Possible values are `flop` horizontal, `flip` vertical or even `flipflop` (guess what). Example: `-f flop`


#### -j, --jpeg *quality*
Save image as JPG with given quality. Example: `-j 90`


#### -o, --output *path*  
Output directory — default is current working directory. Example: `-o results`


#### -r, --resize *pixel*
Resize  image — pixel width for larger side, preserving aspect ratio. Example: `-r 3000`

#### -v, --verbous
Verbous mode — show some more information under way.



### Color and Contrast


#### -d, --desaturate
Desaturate colors, recommended for B/W negatives. Bonus: The TIFF will be converted to 16 bit Grayscale, saving up to 60% in file size; The image will have linear gamma 1.0 ICC profile applied (Gray-elle-V4-g10.icc).

#### -g, --gamma *value*
Gamma correction value to apply. It is highly recommended to use this parameter with linear TIFF images, together with **-n**. Pass a float value like `2.4` or `auto`. *Auto* will apply a calculated gamma adjustment based on the mean values of an image. Examples: `-g 2.2` and `--gamma auto`

#### -n, --normalize
Stretch histogram to reach black and white points. Recommended for most images.

#### -s, --sigmoidal *value*
Sigmoidal contrast value around 50% middle-gray. Increases the contrast without saturating highlights or shadows. Quoted from ImageMagick docs: “3 is typical and 20 is a lot.” Example: `-s 5`

See [Usage](#usage) · [Examples](#xamples) · [Gamma correction](#gamma-correction) · [Sigmoidal contrast](#sigmoidal-contrast)

## Examples

### Single mode: Turn some TIFF files into positive.
Given your photographed negative is a B/W image, so we are using desaturation and grayscaling here. The result will be a grayscale TIFF with approriate ICC profile. *Please note that the positive image gets a `-positive` suffix when it is stored in the same directory like the original image.*

```bash
# These are equal:
$ positive --desaturate DSC_0123.tif
$ positive -d DSC_0123.tif
```

```
------------------------------------------
DSC_0123.tif
Create positive image: Done.
Result: DSC_0123-positive.tif
```

**Second example:** Again, we use desaturation, and this time the positives go into a subdirectory. The results are now stored in a `foobar` directory (Note: no `-positive` suffix).

```bash
$ positive -d --output foobar DSC_0123.tiff DSC_0124.tiff DSC_0125.tiff
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

Just pass `--all` or `-a` option, and the script will mangle everything that has a TIFF extension: `tif, tiff, TIF, TIFF` etc.

The batch mode uses [GNU Parallel,](https://www.gnu.org/software/parallel/) so every single CPU core will be used. So if you have a Quadcore CPU, four images will be processed at the same time. Great timesaver!

Since normalizing is a good idea, we use the `--normalize` or `-n` option in addition to desaturation with `-d`.

```bash
$ positive --all -d -n -o results
$ positive -adn -o results
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

See [Usage](#usage) · [Options](#options) · [Examples](#xamples) · [Gamma correction](#gamma-correction) · [Sigmoidal contrast](#sigmoidal-contrast)

## Tweaking colors

If you are working on linear TIFFs, e.g. as produced by [Dave Coffin's dcraw](http://cybercom.net/~dcoffin/dcraw/dcraw.1.html) by its `-4` flag, both the negative and the positive will look somehow flat, due to their linearity. The dark negative becomes a very light positive. 

The *positive* utility inverses the negative TIFF to positive, *and exactly now* we are able to perform the very gamma correction that did not take place when `dcraw` created the linear TIFF—with the difference that we can choose the gamma as we like. 

### Gamma correction

**Adjust the midtones:** In fact, the light positive must be darkened, using a gamma value lesser than 1. Since such small gamma values are “uncommon” in human-driven image editing, it is easier to talk about  “common” gamma values like 2.2 here. Internally, the script then calculates the reciprocal value (0.45) for darkening.

The `--gamma` or `-g` option carries the Gamma correction value to apply. It is highly recommended to use this parameter with `--normalize` or `-n`. 



```bash
positive -adn --gamma 2.2 -o gamma-corrected
positive -adn -g 2.2 -o gamma-corrected
```
 
**Which gamma value to choose?** In fact, the gamma value is completely up to your taste. An appropriate gamma value will be mainly influenced by these two factors:

- **The negative density** which results from your film development workflow,  
  i.e. its natural contrast and blackness
- **The digicam exposure** chosen during digitalization,  
  i.e. how light or dark is your photo from the negative.

Examples for typical values (according to my personal workflow):
 
gamma | description
:-----|:----------
auto | Calculated gamma, based on mean values. Recommended for ‘real life images’.
1.0 | No Gamma for linear TIFFs. This is the default value if omitted.
1.8 | Moderate darkening, resulting in overall light images
2.2 | Well-known software standard; This is what common photo software would apply.
3.6 | Smooth tones, not too light
4.4 | Somehow darker, still mellow. Not too contrasty.
... | Try your own!


Read more about Gamma correction: [Wikipedia](https://en.wikipedia.org/wiki/Gamma_correction) and ImageMagick: [Gamma Adjustments](http://www.imagemagick.org/Usage/color_mods/#level_gamma) and [auto-gamma.](http://www.imagemagick.org/script/command-line-options.php?#auto-gamma)

### Sigmoidal contrast

**Adjust highlights and shadows:** While the gamma correction mainly affects the midtones, the sigmoidal contrast control works on the highlights and shadows, leaving the 50% midtone alone and resulting in a non-linear, s-like curve. Use the `--sigmoidal value` or `-s value` option to enhance the contrast. Quoted from ImageMagick docs: “3 is typical and 20 is a lot.” – Example:

```bash
positive -adn --g 3.6 --sigmoidal 5 -o nice-contrasts
positive -adn --g 3.6 -s 5 -o nice-contrasts
```

Read more: [ImageMagick](http://www.imagemagick.org/Usage/color_mods/#sigmoidal)

See [Usage](#usage) · [Options](#options) · [Examples](#xamples) · [Gamma correction](#gamma-correction) · [Sigmoidal contrast](#sigmoidal-contrast)





# Workflow Recommendations

## When to crop images?

When photographing your negatives, you'll probably use a negatives (film) holder. 

- The edges of your RAW photo usually will then have a black frame.  
- If the window of your negative holder is larger than the negative, your image will also show unlit film areas (i.e., spaces between the single negative frames), resulting in another “white“ frame. These unlit areas will become ‘absolute black’ in your positives version.

### Valid approaches

The following workflow examples assume your linear TIFFs are framed black and white, having used a negative holder with larger frame window.

#### Approach: Crop first

Start cropping your images and save them as TIF. Be sure to preserve the ICC profile!

Run *positive* with gamma and sigmoidal contrast as needed. When using the `-n` parameter for normalizing colors, both the black and white points will be determined by *the information in your image.* Consequences are: 

0. The higher gamma value or sigmoidal contrast you use, the earlier deepest shadows and lights will start blocking.
0. Underexposured or overexposured images are normalized, that is ‘stretched to histogram edges‘, and may loose their generic underexposured or overexposured look. Images being flat in original may show more film grain.


#### Approach: Do not crop

Run *positive* with gamma and sigmoidal contrast as needed. When using the `-n` parameter for normalizing colors, both the black and white points will be determined by *the darkest grays of the negative holder edges and the lightest parts in the unlit negativ areas.* Deepest shadows and lights are now in the (surfluous) frames. Consequences are: 

0. You may choose way higher gamma and sigmoidal contrast values.
0. The look of underexposured or overexposured images will overall stay natural.



# Changelog

## New Features

### v1.1.0
- **Long option names** for those preferring self-explanatory options like `--resize`, `--desaturate` and so on.
- **Improved code quality:** Wrapped main features in single functions; Adhere to Bash best practices and coding standards.



## Upcoming Features

These features go into the current major version 1:

- **Star rating filter:** Many photo managers like Lightroom or Bridge let their users reject bad images or rate better ones with ‘stars’. *positive* should get a new CLI option flag to set a minimum rating level. Feel free to discuss this in [issue #6.](https://github.com/tomkyle/negatives-positive/issues/6)

- **Custom configuration files:** Would it not be fine if users could store their favourite options in a configuration file? `~/.negativesrc` or ` ~/positive.conf` or even an *INI, YAML* or *JSON?* Head over to the corresponding [issue #7.](https://github.com/tomkyle/negatives-positive/issues/7)



## Roadmap to version 2


- **The *-r* option will be renamed to *-w*,** as *-r* is more natural for the upcoming *Rating filter*, and so is *-w* for *width*. 

- **The *-f* option will be renamed to *-m*,** as the actually performed image action is *mirroring horizontally or vertically*. The option values *flip, flop* and *flipflop* will then become something like *V*, *H* or *VH*.

- **New batch mode trigger:** New sub-command `all` will replace the current `-a` flag, like so: `linear-tiff batch <options>`. 


# Issues and FAQ

To see the full list, head over to the [issues page.](https://github.com/tomkyle/negatives-positive/issues)


**I get a error message “mogrify: delegate library support not built-in”**  
ImageMagick must be compiled with litte-cms2 support. See [issue#1](https://github.com/tomkyle/negatives-positive/issues/1) for details. 



# Development and Contribution

```bash
$ git clone https://github.com/tomkyle/negatives-positive.git
```
