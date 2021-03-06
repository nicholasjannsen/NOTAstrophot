# NOTAstrophot
This python class is specifically made for imaging with the instrument ALFOSC at the Nordic Optical Telescope (NOT) located at Roque De Los Munchacos on La Palma, Canary islands, Spain. Please carefully read the this instructions page in order to successfully run this piece of software. Any problems related to running the code should be added as a github debug or directed to Nicholas E. Jannsen (email: nej@not.iac.es).

The Software is seperated into 4 modules performing 
  1. Image reduction 
  2. Cosmic rays removal
  3. Image alignment
  4. Filter combining and show priliminary color image result 
  
With the fully reduced, aligned, and combined images the idea is to use these for post-processing in e.g. Photoshop, PixInsight, StarTools, Gimp, etc. Hence it should not be expected that this software will produce a good color image, since it's an art in itself. For each module-step each image file is saved and loaded by the next module, as the modules are intended to work independently of each other although they are interlinked within the same class. 

Dependencies
---
The following software works only in Python 3. For consistency it is best to have the newest version of Python installed (by this writing this is Python 3.6). Moreover (preperably the newest versions) the following packages are required:
  - numpy
  - matplotlib
  - astropy
  - scipy
  - skimage
  - colorama
  - astroalign
  - math
  - sys
  - os
  - time
  - glob
  - pylab
  - heapq
  - getopt

With the newest installation of pip, all of the packages above can be installed using `pip install <package>`. Be aware if you have several pyhton versions installed make sure to use python3, e.g. by `python3 -m pip install <package>`.


Usage
---
There is a few requierements for this software which will be stated in the following together with a small example of how to use the code:
  - Place `notastrophot` in a suitable folder on your pc (`~/path/to/software`)
  - Make the software executeable by `chmod +x notastrophot`. Now running the software is simply done with `./notastrophot` or add your path by `PATH=$PATH: ~/path/to/software/` and run the software simply by `notastrophot`.
  - Place your files in a suitable folder on your pc (`~/path/to/data/`)
  - Calibration data (bias and flats) are not a nescessity but highly recommented! If calibration' are available data from each NOT-date (indicated by 6 first initial of the files; e.g. `ALCa19`) needs to be placed in a layer below in a folder called `calibs/ALxxxx/`, e.g. `~/path/to/data/../ALCa19/`, etc. The reason for this locked location is simply to have a minimum number of fits file on the system in the case of several targets have been observed the same night. However, if placing the date elsewhere the corresponding path has to be changed (see section **Parameter tweaking**). Notice no dark frames are needed since the ALFOSC CCD is cooled down to ~ -120 degC. If calibrations' are included, for optimal reduction use:
    - Bias (no.>2) frames (optional) 
    - Flat (no.>2) frames in each filter from the same night (optional)
  - `notastrophot` takes 5 arguments: 1 required; `/path/to/data/`, and 4 optional; `-p`, `-r`, and `-s`, `-c` standing for plot, redo, skip, and clip, respectively. The optional pramaters are by default `False` which means their action will not be performed. If `True` they will be performed, hence, adding
    - `-p` will plot images during each of the 4 steps for each filter available and date.
    - `-r` will redo the calibrations even if saved calibration images already exist. Thus, this allows to add new data together with older data to get a higher S/N, as more data might become available over time. 
    - `-s` will skip the image reduction step. Hence, if no bias and flats are available for a given night, this is a quick work around this problem. However, again it should be stressed that this is not optimal, since optical filters shows many different artefacts and/or sensitivity damages. However, be completely sure that the offending optical filter haven't been re-installed if you're using flats from another night.
    - `-c <clip>` will allow you to add a customized sensitivity level for the detection of cosmic rays. Hence, if `-c` is invoked, a number needs to be added as an input parameter directly after it. Here a low clip value increases the sensitivity and a high value decreases the sensitivity, since clip is exactly equal to the sigma-clipping of detecting a cosmic ray above the noise floor in your images. Reasonable value for images with ALFOSC is between 0.2--2.0 and default is 0.5. It has been tested that for images of low S/N levels `-c 0.7` is good, and for high S/N regimes `-c 0.3` is a better choice. Note that this reduction step is the most time consuming and the lower the threshold, the longer the computation time of the software as more cosmics will be detected.
  - The software is build upon saving all images during all 4 steps of the image processing. Each file is saved as the following: `<step>_<NOTfilename>_<filter>.fits` e.g. `calib_ALCa190097_B_Bes_440_100.fits`. This is done to keep the structure of the module independent.

Usage example
---
NOTAstrophot has a build-in Usage Help function which is automatically invoked if no input parameters are parsed. Hence, try open a terminal go to the folder where the software is placed (`cd /path/to/software/`) and simply run `./notastrophot`. Thus, if you forget the input parameters you can likewise easily find help from the command line.  

As an usage example, lets say you have acquiered 4 images of M1 from the same night in R, G, B, and Halp, and that you want to see each reduction step plotted. The following will do the job from a terminal:
```
notastrophot -p -r -c 1.0 /path/to/m1/
```
Parameter tweaking
---
Under the `class` description of this software the "Global Variables" are defined, and in python these are defined as `self.<variable>`. These may be tweaked to get the best result depending on filters/CCD position of target, sky background level, noise, blended stars, etc. Please have a look at these and tweak them if you feel like it. Only the following parameters should be tweaked:
  - `pathc`: This is the input path pointing to the calibration data. It is recommended to not change this, however, another suitable option is to place both target and calibration data within the same folder and change `pathc` to this folder. 
  - `borders`: Takes integers of `[y1, y2, x1, x2]` as input to select different borders, where the full frame CCD readout with ALFOSC is equal to `[1, 2148, 1, 2102]`. The optical filters used with ALFOSC all have a square or round border (hence regions with low light transmission) and therefore this needs to be removed. By default this is already done by default using `[300, 1900, 300, 1900]` for the square filters, however, with narrow-band filters the images still shows clear edge effects, which needs to removed in a post-processing software. Thus, depending on the object size the following borders are recommended
    - `[150, 1950, 200, 2000]` : Without narrow-band filter
    - `[300, 1900, 300, 1900]` : Defualt
    - `[450, 1800, 300, 1800]` : With narrow-band filters  
  - `sigfrac`: Fractional detection limit for neighbouring pixels of cosmics. This can be an important parameter to tweak in cases of a low/high number of cosmic rays are present in the images and the image is of low/high S/N quality.
  - `objlim`: Contrast limit between cosmics and underlying (object/sky) background. 
  - `maxiter`: The number of iterations that will be used when looking for cosmics. Default is 3 which should be sufficient in many cases. Note increasing the iterations also prolong computation time.
  - `starsigma`: This is a threshold parameter for identifying stars within the images, and for which is used to match the coordinates within each image to shift the images so they add up. If no matching stars are found, decrease this parameter. Default is 2.
- `pixelshift`: Maximum pixel distance offset that the software looks for between the sequence of images. If images of different pointings, coordinates, or dithering is added, this parameter needs to be increased considerably, however, be aware that it might make a false coordinate match between one or more stars. Default is 20 pixel, since usually with guiding the offset is `< 5 pixels`.   
  
Debugging and Development
---
- Allowing the cosmic detection thresholds depend on a function of exposure time, magnitude, surface brightness. This will make the software much more efficient. 
- If data before 2017 is processed the image header are missing the keyword `IMAGETYP` which is used to seperate the image files. A future solution could be to make an if-statement and seperate these images by another keyword.
- If more than approx 15 bias or flat frames are feed to the program, the numpy array structure changes and the program crashes. It might be an internal problem with fits files and numpy, but the problem haven't been solved.
- A future update could be to make better corrections for the sky background in the narrow band filters. This could e.g. be done by using the info of gaussian noise from the cosmic detection algorithm.
- The image registration module first tries to use the Python library `astroalign` which solves images using trigometric stellar coordinate solutions, however, it seems to crash from time to time upon being exposed to ALFOSC images. If this happens the software will continue with a simple alignment using stellar coordinates; by finding stars and identifying identical object within a relatively short pixel distance apart. If image registration with astroalign fails, this limits the act of combining old frames with different coordinates, hence, a future update could be to try other image registration python repositories (e.g., `alipy`, `imagereg`, etc.) before falling back to simple alignment.
