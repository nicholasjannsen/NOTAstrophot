NOTAstrophot - Version 0.1
---
This is a README file for the astrophotography software "notastrophot" made for optical photometry with the instrument ALFOSC at the Nordic Optical Telescope (NOT). Please read carefully the following instructions (Dependencies, Description, and Usage) in order to be successful running this piece of software. Any problems related to running this software should be directed to Nicholas Jannsen (email: nej@not.iac.es).   

DEPENDENCIES:
---
The following software works only in Python 3. For consistency it's best to have the newest version of Python installed (by this writing this is Python 3.6). Moreover (preferably the newest versions) the following packages are required:
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
With the newest installation of pip, all of the packages above can be installed using e.g.: "pip install numpy".


DESCRIPTION:
---
This python class is specifically made for imaging with the instrument ALFOSC at the NOT located at observatory 'Roque De Los Munchacos' on Canary island La Palma, Spain. The Software is separated into 4 modules performing (1) Image reduction, (2) Cosmic rays removal, (3) Image alignment, and (4) Filter combining which lastly producing a simple color image. With the fully reduced, aligned, and combined images the idea is to use these for post-processing in e.g. Gimp, Photoshop, PixInsight, StarTools, etc. Hence it should not be expected that this software will produce a good color image, since it's an art in itself. After reduction either use the "color*.png" image or the load each fits filter image, those called "final*.fits", into your favourite post-processing software. 

USAGE:
---
There is a few requirements for this software which will be stated in the following together with a small example of how to use the code:
- Put all your files (both target and calibration images) in a suitable folder on your pc (`/path/to/data/"`). It is possible to add images from different dates by simply adding these to the same folder as well. 
- Download the `notastrophot` software a place it a convenient place. If you wish to reduce images from several different targets from one night, it it recommended to place the software in the folder prior to each target folder.
- If it's the first time running the software, make it executable. Hence, first open a terminal (on Linux this is `Ctrl-Alt-t`), secondly, go to the folder where the software is situated (`cd /path/to/software/`) and, third, make the software executable by writing `chmod +x notastrophot`.
- You will need to have bias' (`2<no.<11`) images. Usually 11 bias images are made from the daily morning calibrations at the NOT.  
- Flats are not a necessity but recommended (`2<no.<11`) images in each filter from the same night is recommended. 
- `notastrophot` takes 5 arguments: 1 required; `/path/to/data/`, and 4 optional; `-p`, `-r`, and `-s`, `-c` standing for plot, redo, skip, and clip, respectively. The optional pramaters are by default `False` which means their action will not be performed. If `True` they will be performed, hence, adding:
`-p` will plot images during each of the 4 steps for each filter available.
`-r` will redo the calibrations even if saved calibration images already exist. Thus, this allows to add new data together with older data to get a higher S/N, as more data might become available over time. 
`-s` will skip flat fielding. Hence, if no flats are available for a given night this is a quick work around this problem. However, again it should be stressed that this is not optimal, since optical filters shows many different artefacts and/or sensitivity damages.
`-c <clip>` will allow you to add a customized sensitivity level for the detection of cosmic rays. Hence, if `-c` is invoked a number needs to be added as an input parameter directly after it. Here a low clip value increase the sensitivity and a high value decreases the sensitivity, since clip basically is the sigma-clipping of detecting a cosmic ray above the noise floor in your images. Reasonable value for images with ALFOSC is between 0.2-2.0 and default is 0.5. It has been tested that images of low S/N levels `clip = 0.7` is good, and for high S/N regimes `clip = 0.3` is a better choice. Note if that the lower the number, the longer the computation time of the software!
- The software is build upon saving all images during all 4 steps of the image processing. Each file is saved as the following: `<step>_<NOTfilename>_<filter>.fits` e.g. `calib_ALCa190097_B_Bes_440_100.fits`. This is done to keep the structure of the module independent.
- Try open a terminal and go to the folder where the software is places (`cd /path/to/software/`) and simply run `./notastrophot`. Running the software without any input parameters will call the software "Usage" help description. Thus, if you forget the input parameters you can easily find help here.   

USAGE EXAMPLE:
---
As an usage example, lets say you have acquired 4 images of M1 from the same night in B, V, R, and Halp, and that you want to both wants to plot the result of each step, wants redo the image reduction, wants to select a cosmic ray sensitivity level of 1.0. The following will do the job from a terminal:
```
notastrophot -p -r -c 1.0 /path/to/m1/
```

**---------------- ONLY FOR EXPERTS -------------------**

PARAMETER TWEAKING:
---
Under the "class" description of this software the "Global Variables" are defined, and in python these are defined as `self.<variable>`. These may be tweaked to get the best result depending on filters/CCD position of target, sky background level, noise, blended stars, etc. Please have a look at these and tweak them if you feel like it. Only the following parameters should be tweaked:
- `borders`: Takes integers of `[y1, y2, x1, x2]` as input to select different borders. The optical filters used with ALFOSC all have a square or round borders/low transmission regions and therefore this needs to be removed. By default this is already done by default using `[300,1900,300,1900]`. However, good borders for ALFOSC imaging are also `[150,1950,200,2000]` or `[450,1800,300,1800]`.  
- `sigfrac`: Fractional detection limit for neighbouring pixels of cosmics. This can be an important parameter to tweak in cases of a low/high number of cosmic rays are present in the images and the image is of low/high S/N quality.
- `objlim` : Contrast limit between cosmics and underlying object. 
- `maxiter`: The number of iterations that will be used when looking for cosmics. Default is 3 which should be sufficient in many cases. Note increasing the iterations also prolong computation time.
- `starsigma`: This is a threshold parameter for identifying stars within the images, and for which is used to match the coordinates within each image to shift the images so they add up. If no matching stars are found, decrease this parameter. Default is 2.
- `pixelshift`: Maximum pixel distance offset that the software looks for between the sequence of images. If images of different pointings, coordinates, or dithering is added, this parameter needs to be increased considerably, however be aware that it might make a false coordinate match between one or more stars. Default is 20 pixel, since usually with guiding the offset is `<5` pixels. 

*
Have fun! Cheers,

Nicholas Jannsen
Student Support Astronomer
Nordic Optical Telescope
*
