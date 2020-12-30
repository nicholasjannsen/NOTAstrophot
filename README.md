# NOTAstrophot
This python class is specifically made for imaging with the instrument ALFOSC at the Nordic Optical Telescope (NOT) located at Roque De Los Munchacos on La Palma, Canary islands, Spain. Please carefully read the this instructions page in order to successfully run this piece of software. Any problems related to running the code should be added as a github debug or directed to Nicholas Jannsen (email: nej@not.iac.es).

The Software is seperated into 4 modules performing 
  1. Image reduction 
  2. Cosmic rays removal
  3. Image alignment
  4. Filter combining and show priliminary color image result 
  
With the fully reduced, aligned, and combined images the idea is to use these for post-processing in e.g. Photoshop, PixInsight, StarTools, Gimp, etc. Hence it should not be expected that this software will produce a good color image, since it's an art in itself. For each module step each image file is saved and loaded by the next module, as the modules are inteded to work independently of each other although they are interlinked within the same class. 

Usage
---
There is a few requierements for this software which will be stated in the following together with a small example of how to use the code:
  - Place `notastrophot` in a suitable folder on your pc (`~/path/to/software`)
  - Make the software executeable by `chmod +x notastrophot`. Now running the software is simply done with `./notastrophot` or add your path by `PATH=$PATH: ~/path/to/software/` and run the software simply by `notastrophot`.
  - Place your files in a suitable folder on your pc (`~/path/to/data/`)
  - Calibration data (bias and flats) are not a nescessity but highly recommented!  Notice no dark frames are needed since the ALFOSC CCD is cooled down to ~ -120 degC.
    **If calibration data IS available:** 
      - Running by default, calibration data needs to be placed in a layer below in a folder called `calibs`: `~/path/to/data/../calibs/`. The reason for this locked location is simply to have a minimum number of fits file on the system. If the calibration data is placed elsewhere, the corresponding path has to be changed, which revised in the "Parameter tweaking" section. If calibrations' are included, for optimal exraction use:
      - Bias (no.>3) frames (optional) 
      - Flat (no.>3) frames in each filter from the same night (optional)
    **2. If calibration data IS NOT available:** 
  - `notastrophot` takes 5 arguments: 1 required; `/path/to/data/`, and 4 optional; `-p`, `-r`, and `-s`, `-c` standing for plot, redo, skip, and clip, respectively. The optional pramaters are by default `False` which means their action will not be performed. If `True` they will be performed, hence, adding
    - `-p` will plot images during each of the 4 steps for each filter available and date.
    - `-r` will redo the calibrations even if saved calibration images already exist. Thus, this allows to add new data together with older data to get a higher S/N, as more data might become available over time. 
    - `-s` will skip the image reduction step. Hence, if no bias and flats are available for a given night, this is a quick work around this problem. However, again it should be stressed that this is not optimal, since optical filters shows many different artefacts and/or sensitivity damages. However, been completely sure that the offending optical filter haven't been re-installed if you're using flats from another night - hence when in doub leave it out!
    - `-c <clip>` will allow you to add a customized sensitivity level for the detection of cosmic rays. Hence, if `-c` is invoked, a number needs to be added as an input parameter directly after it. Here a low clip value increases the sensitivity and a high value decreases the sensitivity, since clip is exactly equal to the sigma-clipping of detecting a cosmic ray above the noise floor in your images. Reasonable value for images with ALFOSC is between 0.2--2.0 and default is 0.5. It has been tested that for images of low S/N levels `-c 0.7` is good, and for high S/N regimes `-c 0.3` is a better choice. Note that this reduction step is the most time consuming and the lower the threshold, the longer the computation time of the software as more cosmics will be detected.
  - The software is build upon saving all images during all 4 steps of the image processing. Each file is saved as the following: `<step>_<NOTfilename>_<filter>.fits` e.g. `calib_ALCa190097_B_Bes_440_100.fits`. This is done to keep the structure of the module independent.

Usage example
---
NOTAstrophot has a build-in Usage Help function which is automatically invoked if no input parameters are parsed. Hence, try open a terminal go to the folder where the software is placed (`cd /path/to/software/`) and simply run `./notastrophot`. Thus, if you forget the input parameters you can likewise easily find help from the command line.  

As an usage example, lets say you have acquiered 4 images of M1 from the same night in R, G, B, and Halp, and that you only want to see the cosmic removal and final image plotted. The following will do the job from a terminal and plot the results:
```
notastrophot -p -r -c 1.0 /path/to/m1/
```
Parameter tweaking
---
Under the `class` description of this software the "Global Variables" are defined, and in python these are defined as `self.<variable>`. These may be tweaked to get the best result depending on filters/CCD position of target, sky background level, noise, blended stars, etc. Please have a look at these and tweak them if you feel like it. Only the following parameters should be tweaked:
  - `borders`: Takes integers of `[y1, y2, x1, x2]` as input to select different borders, where the full frame CCD readout with ALFOSC is equal to `[1, 2148, 1, 2102]`. The optical filters used with ALFOSC all have a square or round border (hence regions with low light transmission) and therefore this needs to be removed. By default this is already done by default using `[300, 1900, 300, 1900]` for the square filters, however, with narrow-band filters the images still shows clear edge effects, which needs to removed in a post-processing software. Thus, depending on the object size the following borders are recommended
    - `[150, 1950, 200, 2000]` : Without narrow-band filter
    - `[300, 1900, 300, 1900]` : Defualt
    - `[450, 1800, 300, 1800]` : With narrow-band filters  
  - `sigfrac`: Fractional detection limit for neighbouring pixels of cosmics. This can be an important parameter to tweak in cases of a low/high number of cosmic rays are present in the images and the image is of low/high S/N quality.
  - `objlim`: Contrast limit between cosmics and underlying (object/sky) background. 
  - `maxiter`: The number of iterations that will be used when looking for cosmics. Default is 3 which should be sufficient in many cases. Note increasing the iterations also prolong computation time.
  - `starsigma`: This is a threshold parameter for identifying stars within the images, and for which is used to match the coordinates within each image to shift the images so they add up. If no matching stars are found, decrease this parameter. Default is 2.
- `pixelshift`: Maximum pixel distance offset that the software looks for between the sequence of images. If images of different pointings, coordinates, or dithering is added, this parameter needs to be increased considerably, however, be aware that it might make a false coordinate match between one or more stars. Default is 20 pixel, since usually with guiding the offset is < 5 pixels.   
  
Dependencies
---
The following software works only in Python 3. For consistency it is best to have the newest version of Python installed (by this writing this is Python 3.6). Moreover (preperably the newest versions) the following packages are required:
  - numpy
  - math
  - matplotlib
  - astropy
  - scipy
  - skimage
  - colorama
  - time
  - glob
  - pylab
  - sys
With the newest installation of pip, all of the packages above can be installed using `pip install <package>`. Be aware if you several pyhton versions are installed, make sure to use python3 by e.g. `python3 -m pip install <package>`.

Debugging and Development
---
- Allowing the cosmic detection thresholds depend on a function of exposure time, magnitude, surface brightness. This will make the software much more efficient. 
- If data before 2017 is processed the image header are missing the keyword `IMAGETYP` which is used to seperate the image files. A future solution could be to make an if-statement and seperate these images by another keyword.
- If more than approx 15 bias or flat frames are feed to the program, the numpy array structure changes and the program crashes. It might be an internal problem with fits files and numpy, but the problem haven't been solved.
- A future update could be to make better corrections for the sky background in the narrow band filters. This could e.g. be done by using the info of gaussian noise from the cosmic detection algorithm.
- The image alignment of this software is developed from finding stars and identification of identical object within a relatively short pixel distance apart. This limits the act of combining old frames with different coordinates, hence a future update could be to simple use the python repositories `astroalign`, `alipy`, `imagereg`, etc., which both can take care of rotation, shift, and add operations.

