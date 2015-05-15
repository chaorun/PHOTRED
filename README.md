# PHOTRED
Automated DAOPHOT-based PSF photometry pipeline

PHOTRED is an automated DAOPHOT-based PSF photometry pipeline.  It's very generic but does require reduced flat images.
PHOTRED does WCS fitting, aperture photometry, single-image PSF photometry (ALLSTAR), source matching across multiple images,
forced PSF photometry across multiple exposures using a master source list created from a deep stack of all exposures
(ALLFRAME), aperture correction, calibration using photometric transformation equations, and dereddening.
STDRED is also included and can be used to derive the transformation equations from exposures of standard star fields.

PHOTRED does require the following that need to be installed separately:
-IRAF
-IDL
-The IDL Astronomy Uers's Library
-The stand-alone fortran version of DAOPHOT/ALLFRAME
-SExtractor

PDF manuals for PHOTRED and STDRED are included in the doc/ directory.

IDL "sav" files of all the main PHOTRED programs are in the sav/ directory.  These can be used with the IDL virtual
machine for people who don't have an IDL license.

# Installation Instructions

## 1. Download the PHOTRED IDL programs
Download the PHOTRED IDL programs tar file (last updated 06/02/08). Copy this to your IDL directory (most likely ~/idl/) and unpack it:

```
gunzip photred_idl.tar.gz tar -xvf photred_idl.tar
```

Let it overwrite any older programs by the same name. You need the new
versions!  You will also need the IDL Astro User's Library. The
programs are automatically available on UVa Astronomy. If you don't
have the IDL Astro User's Library then you should download the
programs from their download site (get "astron.tar.gz").

There can be a problems if you have older copies of the IDL Astro
User's programs in your IDL directory or programs with the same
name. I've included a program called "checkidlastro.sh" in the
photred_idl.tar file that will print out the names of programs in your
~/idl/ directory (and subdirectories) that have the same name as
programs in the IDL Astro User's Library (run it by typing
"./checkidlastro.sh" in your ~/idl/ directory; it needs the file
"idlastro.lst" which is also included). If it finds anything I would
erase the offending program or rename it (e.g. proname.pro ->
proname.pro.bak).

## 2. Download the PHOTRED scripts

Download the PHOTRED scripts tar file (last updated 06/02/08). Make a
directory where these scripts will reside (i.e. ~/photred/), and copy
the tar file to the directory and unpack it:

gunzip photred_scripts.tar.gz tar -xvf photred_scripts.tar

There are two fortran codes (lstfilter.f and makemag.f) that need to
be compiled. The tar file includes compiled versions that were
compiled on a Linux system. If you are planning to run PHOTRED on a
Sun machine or are having problems with the programs recompile them:

```
gfortran lstfilter.f -o lstfilter
```

## 3. Make sure IDL/IRAF are available

PHOTRED needs IDL and IRAF run. If you don't have IDL then you might
consider buying a license from ITT Visual Information
Solutions. Student licenses are available and full licenses have come
down in price. If these are too expensive, then you might consider
installing the free version of IDL, the GNU Data Language. I have not
tested PHOTRED on GDL, but I think GDL should have all of the
functionality needed to run PHOTRED. However, some PHOTRED programs
might need to be tweaked to work with GDL.  You can also download the
IDL Virtual Machine for free and run IDL "sav"

You can also download the IDL Virtual Machine for free and run IDL
"sav" files. I have made IDL sav files for the PHOTRED pipeline so
they can be run with the IDL Virtual Machine. Download the tar file
(last updated 05/12/08) and put them in your ~/idl/ directory. To run
a program type "idl -vm=progname.sav".  IRAF is freely downloadable
from iraf.net.

## 4. Make sure DAOPHOT/ALLFRAME is installed

Make sure that DAOPHOT/ALLSTAR/ALLFRAME and SExtractor are installed. Type:

```
which daophot
which allstar
which daomaster
which daogrow
which allframe
which sex
```

They should all return the name of the program. If you get an error
then you will need to install that program.

Currently daophot and allstar are hardwired to /net/astro/bin/ in the
daophot.sh and getpsf.sh scripts. If this is not where your version of
daophot/allstar are located then change those lines or comment them
out.

The newest version of ALLFRAME had an error in it that caused it to
print out zero's for chi and sharp. The fixed version on the UVa Astro
machines is called "/net/halo/bin/allframe.2004.fixed". This is
currently hardcoded in photred_allframe.pro and allframe.pro. Make
sure that this program exists.

## 5. Schlegel Maps

PHOTRED uses the Schlegel maps to deredden the photometry. The FITS
files can be downloaded from here. On the UVa Astro machines the files
are located here: /net/grass/catalogs/reddening/

You need to update your ".cshrc" file to set the Unix environment
variable "DUST_DIR" to the directory where the Schlegel dust maps are
located (actually one directory up in the directory tree). At UVa
Astro this is the line you should add to your ".cshrc" file:

```
setenv DUST_DIR /net/grass/catalogs/reddening/
```

Now test that it works (you must already have installed the PHOTRED
IDL files):

```
mycomputer % idl IDL>print,dust_getval(10,10)
 ￼￼0.472163
```

If you get an error here, then there is a problem. Check that all the
files and required programs are there.

## 6. Setup Your IRAF Login File

PHOTRED calls some IRAF programs. In order for this to work you need
to edit your IRAF "login.cl" file so that it doesn't print out any
messages to the screen. Comment out the 9 lines following "# Set the
terminal type." near the top of the file, and possibly also the 4
lines following "# Delete any old MTIO lock (magtape position) files."
(these give problems on the Pleione cluster). If that still doesn't
work, create a blank file called ".hushiraf" in your iraf directory.

```
touch .hushiraf
```

Start IRAF by typing "cl" in your IRAF directory and see what
happens. Nothing should be written to the screen except "cl>" or maybe
"ecl>". If it's still printing other things to the screen, then you'll
need to comment out more lines from the "login.cl" file.

# Running Instructions

## 1. Data

Start by putting all the final flat frames in their nights
directory. Process them with IRAF's CCDRED, MSCRED or Armin Rest's
SuperMacho pipeline to get final flat images. PHOTRED only runs on one
night's worth of data. If you have 5 nights of data then you will need
to run PHOTRED 5 times (they can be running simultaneously in separate
directories).

FIX BAD PIXELS!!!. Make sure to fix any bad pixels or columns in the
images because otherwise they might get detected as "sources" and mess
up WCS, DAOPHOT, MATCH, and ALLFRAME.

## 2. Transformation Equations

In order to do calibration step (CALIB) PHOTRED needs the photometry
transformation equations which means doing the standard star
reduction. There is a separate pipeline for the standard star
reduction called STDRED that you should run first to get the
transformation equations for the entire run. You can run all of the
PHOTRED stages before CALIB (which include the cpu intensive DAOPHOT
and ALLFRAME stages) without the transformation equations, and then do
the final stages once you have transformation equations.

The transformation file needs to be in this format. Each filter gets
two lines. The first line should contain: filter, color, nightly
zero-point term, airmass term, color term, airmass*color term, color^2
term. The second line gives the uncertainties in the five
terms. Normally the last two terms (airmass*color and color^2 are
0.0000).

n1.trans

```
M M-T  -0.9990 0.1402 -0.1345 0.0000 0.0000
       1.094E-02 5.037E-03 2.010E-03 0.0000 0.0000
T M-T   -0.0061 0.0489 0.0266 0.0000 0.0000
       6.782E-03 3.387E-03 1.374E-03 0.0000 0.0000
D M-D   1.3251 0.1403 -0.0147 0.0000 0.0000
       1.001E-02 5.472E-03 2.653E-02 0.0000 0.0000
```

## 3. Setup File

PHOTRED needs a "photred.setup" file to run. This file specifies a few
important parameters. Here's an example of a "photred.setup" file. The
various parameters are described below.

```
##### REQUIRED #####
scriptsdir  /idl/idllocal/photred/PHOTRED/scripts/
irafdir     /home/smash/
telescope   Blanco
instrument  DECAM
observatory CTIO
nmulti      10
filtref     g,i,r,z,u
trans       nocalib.trans
##### OPTIONAL #####
sepfielddir 1
keepmef     0
redo        0
#skipwcs    0
#wcsup      N
#wcsleft    E
#pixscale   0.50
#wcsrefname USNO-B1
#searchdist 60
#wcsrmslim  1.0
#hyperthread 1
psfcomsrc   1
psfcomglobal 1
#psfcomgauss 1
#mchmaxshift  50.0
finditer    2
#alfdetprog  sextractor
#alfnocmbimscale 0
#alfexclude  F1,F3
ddo51radoffset  1
keepinstr   1
avgmag      1
avgonlymag  0
todered     M,T,D,M-T,M-D
#toextadd    M,T,D,M-T,M-D
#cmd2cdaxes  M,M-T,M-D
stdfile	     /home/smash/SMASH/standards.txt
##### STAGES #####
rename
split
wcs
daophot
match
allframe
apcor
astrom
calib
combine
deredden
save
html
```

Parameter  |  Description
------------ | -------------
*REQUIRED* |  *REQUIRED*
scriptsdir |  The absolute path to the directory that contains the PHOTRED scripts (i.e. daophot.sh, etc.)
irafdir | The absolute path to your IRAF directory (that contains your login.cl file)
telescope | The name of the telescope (e.g. Blanco, Swope)
instrument | The name of the instrument (e.g. MOSAIC)
observatory | (OPTIONAL) The name of the observatory. This is needed if the header does not contain the AIRMASS and it needs to be calculated from the date, ra/dec and observatory location.
nmulti |  The number of processors PHOTRED should use for the DAOPHOT and ALLFRAME stages (only relevant if on the Pleione cluster).
filtref |  The shortname of the filter (specified in the "filters" file) to be used as the reference frame (e.g. the M frame). This can be a comma-delimited priority-ordered list of filters (e.g. g,i,r,z,u). If there are multiple observations in this filter then the longest exposure in this filter will be used.
trans | The name of the file that contains the photometric transformation equations.
keepmef | OPTIONAL. Multi-extension files (MEF) are split by PHOTRED. Do you want PHOTRED to keep the MEF files: YES=1, NO=0 (i.e. erase them).
*OPTIONAL*  |  *OPTIONAL*
sepfielddir |  Put each field in a separate directory (this is now the default option), otherwise everything will go in the main directory and can slow down processing because a very large (~100,000) number of fileds.
keepmef  |  Keep the original multi-exension files (MEF).
redo | PHOTRED will NOT reprocess files that have already been processed unless "redo" is set. This can also be set as a keyword on the command line (i.e. IDL>photred,/redo).
skipwcs | Set this if your images already have correct WCS in their headers and you don't want the WCS to be refit in the WCS stage.
wcsup | What cardinal direction (i.e. N, S, E or W) is "up" in the image? This is only used for non-standard setups.
wcsleft | What cardinal direction (i.e. N, S, E or W) is "left" in the image? This is only used for non-standard setups.
pixscale | The plate scale in arcseconds/pixel. This is ONLY used for non-"standard" imagers (i.e. not MOSAIC, IMACS, LBC or Swope) where the pixel scale cannot be determined from the image headers.
wcsrefname | The name of the WCS reference catalog to use. The two options are 'USNO-B1' and '2MASS-PSC'. USNO-B1 is the default. The astrometric accuracy of the 2MASS catalog is better (~0.170 arcsec) than USNO-B1 (~0.270 arcsec), but it does not go as deep (R~18) as USNO-B1 (R~20). So if you have deep images then definitely use USNO-B1, but if you have moderately deep images then 2MASS-PSC is probably better (and faster).
searchdist | This sets the search distance (in arcmin) for WCS fitting (PHOTRED_WCS). Normally this is not needed. The default is 2*image size > 60 arcmin (i.e. whichever is greater). This is normally sufficient. If the WCS isn't fitting correctly then try setting "searchdist" to a larger value.
wcsrmslim | This is the maximum RMS (in arcseconds) allowed for an acceptable WCS fit. The default is 1.0 arcseconds. Normally the RMS values are ~0.2-0.3 arcseconds.
hyperthread | This allows multiple jobs to be running (daophot and allframe only) on a computer (such as halo or stream) that has multiple processors. It's similar to running it on a cluster.
psfcomsrc | If this is set to 1 then only sources detected in other frames of the same field are used as PSF stars. This makes sure that your PSF stars are not contaminated by cosmic rays or other junk. HIGHLY RECOMMENDED.
psfcomglobal | This finds PSF stars in other exposures of the same field in a "global" manner that works better with large dithers.  HIGHLY RECOMMENDED.
psfcomgauss | This fits Gaussians to each potential PSF star and requires the Gaussian parameters to be "reasonable".
mchmaxshift | This sets a maximum constraint on the X/Y shifts found in the MATCH stage. WARNING, only use this if you ABSOLUTELY known that your frames are well aligned already and do not have large dithers.
finditer | The number of times to iteratively find sources in ALLFRAME (allfprep). The default is 2.
alfdetprog | The program to use for source detection in the ALLFRAME stage (allfprep). The options are "sextractor" and "daophot". The default is "sextractor". SExtractor is generally better at finding faint sources and returns a stellaricity probability value which is very useful. HOWEVER, SExtractor fails in VERY crowded regions. It's best to use DAOPHOT for very crowded images.
alfnocmbimscale | Do not "scale" the combined images in the ALLFRAME prep stage.
alfexclude  |  Comma-delimited list of fields (e.g., F1, F3, F5) to exclude from ALLFRAME processing.
ddo51radoffset | There is a photometric offset in the DDO51 filter that depends on the radial distance from the center of the field. Currently this is only observed in the CTIO+MOSAIC data. Setting this parameter will remove this offset (done in CALIB). If you use this make sure to also use it in STDRED.
keepinstr | CALIB should keep the instrumental magnitudes in the final output file.
avgmag | CALIB should calculate average magnitudes in filters that were observed multiple times. The individual magnitudes are also kept.
avgonlymag | Same as "avgmag" but only keeps the average magnitudes.
todered | The magnitudes and colors the DEREDDEN stage should deredden. The short names of the filters and colors should be used (i.e. M, T, M-T). The dereddened magnitudes and colors will have the same names but with a "0" (zero) after it (i.e. M0 for M). The dash is removed for the color names.
toextadd | The extinction and reddening values to add. The short names of the filters and colors should be used (i.e. M, T, M-T). The extinctions will have the same name as the magnitude but with an "A" at the beginning (i.e. AM for M), and the reddening values will have the same name as the color names but with the dash removed and an "E" at the beginning (i.e. EMT for M-T).
cmd2cdaxes | The magnitudes and colors to use for the CMD and 2CD plots in PHOTRED_HTML.PRO. It should be in this format: MAG, MAG1-MAG2, MAG3-MAG4, i.e. M,M-T,M-D. For this example the CMD would be M vs. M-T and the 2CD would be M-D vs. M-T. The CMD and 2CD will be connected (CMD on top, 2CD on bottom) and will share the x-axis which be the first color. The magnitude will also be used for other plots (i.e. error vs. magnitude, chi vs. magnitude, etc).
stdfile  |  List of standard star fields to exclude from processing and move to the "standards/" directory.

Then add all the stage names that you want to process from the
following list: rename, split, wcs, daophot, match, allframe, apcor,
calib, astrom, combine, deredden, and save

The number of ALLFRAME iterations can be set in the "allframe.opt" file, the MA parameter. The default is 50.

## 4. Make sure your Imager is in the "imagers" file

There is an "imagers" file in your scripts directory. If the imager
you are using is not in the list then add it at the end. You need the
telescope name, instrument names, observaotry name (as found in the
OBSERVATORY.PRO IDL program), the number of amplifiers, and the
separator character (only necessary for multi-amplifier imagers). The
TELESCOPE and INSTRUMENT names in your "photred.setup" file needs to
match the ones in the "imagers" file.

```
# TELESCOPE   INSTRUMENT  OBSERVATORY  NAMPS  SEPARATOR
blanco        decam       CTIO         62     _
blanco        mosaic      CTIO         16     _
mayall        mosaic      KPNO         8      _
baade         imacs       LCO          8      c
lbt           lbc         MGIO         4      _
swope         ccd         LCO          1
rrrt          sbig        FMO          1      -
2p2           wfi         LASILLA      8      _
```

The case is not important. The separator character is used to separate
the amplifier number from the main frame name for multi-amplifier
images (i.e. "obj1034_1.fits" is the filename of the first amplifier
FITS file of the "obj1034" image). The separator character is almost
always the underscore "_". The only exception (for now) is IMACS which
uses a "c". If your images are in the multi-extension format (MEF)
then they will be split up into amplifier files in the SPLIT stage of
PHOTRED with the IRAF task MSCSPLIT which uses the underscore as the
separator character.

## 5. Check "filters" file

PHOTRED uses short names for filters, and these are stored in the
"filters" file. One is provided in the scripts tar file. This is what
it looks like:

```
'M Washington c6007'    M
'M Washington k1007'    M
'M'                     M
'I c6028'               T
'I Nearly-Mould k1005'  T
'T'                     T
'T2'                    T
'DDO 51 c6008'          D
'D51 DDO c6008'         D
'D51 DDO 51 c6008'      D
'D51 DDO 51 k1008'      D
'D'                     D
'D51'                   D
'DDO51'                 D
'DDO-51'                D
'DDO_51'                D
'DDO 51'                D
'51'                    D
'T1'                    T1
'C'                     C
'B Harris c6002'        B
'B-BESSEL'              B
'B'                     B
'V'                     V
'R-BESSEL'              R
'R'                     R
'u DECam c0006 3500.0 1000.0'         u
'g DECam SDSS c0001 4720.0 1520.0'    g
'r DECam SDSS c0002 6415.0 1480.0'    r
'i DECam SDSS c0003 7835.0 1470.0'    i
'z DECam SDSS c0004 9260.0 1520.0'    z
'Y DECam c0005 10095.0 1130.0'        Y
'u'                     u
'g'                     g
'r'                     r
'i'                     i
'z'                     z
'Y'                     Y
```

The first column is the text that is found in the FILTER keyword in
the FITS header. The second column is the shortname. These can be
repeated because different observatories have different names for the
same filter. These shortname will be used in the transformation file
and the "extinction" file.

Make sure that your filters appears in the list (leading/trailing
spaces are not important. If they don't then PHOTRED will make a new
entry in the "filters" file for your filter and make a new
shortname. This is NOT desirable because the shortname probaby won't
match what you have in your transformation file or in the "extinction"
file.

Here's a way to double-check if your filter is in the "filters"
file. Copy the "filters" file from your scriptsdir to the directory
where your data is. Then run PHOTRED_GETFILTER on one FITS file per
filter. Change the shortname to a single capital letter if possible.

```
IDL>print,photred_getfilter('ccd1001.fits')
D
```

PHOTRED_GETFILTER will print out the shortname of the filter. It will
tell you if it didn't find the filter in the "filters" file and what
new entry it added. It's preferable to have the most up to date
"filters" file in the scriptsdir directory so that it can be used for
the next run. PHOTRED uses the "filters" file in the main directory
(where "photred.setup" and the data are located) if there is one,
otherwise it will copy the "filters" file from the scriptsdir
directory.

## 6. Check "extinction" file

If you want your photometry dereddened then PHOTRED needs to know what
extinction value to use for each filter. This is stored in the
"extinction" file. The first column has the filter shortname and the
second column A(filter)/E(B-V) for that filter. The reddenings for
colors can be derived from these values, so no entries for colors are
needed.

```
# Filter    A(filter)/E(B-V)
M           3.43
T           1.83
D           3.37
u           4.239
g           3.303
r           2.285
i           1.698
z           1.263
```

Make sure your filter and its appropriate extinction value appears in
this file. Try to update the "extinction" As with the "filters" file
PHOTRED uses the "extinction" file in the data directory if there is
one, otherwise it will copy the "extinction" file from the scriptsdir
directory.


## 7. Run PHOTRED_RENAME

Okay, now you're ready to PHOTRED. PHOTRED has 13 stages and there is
a separate IDL program for each stage (e.g. PHOTRED_DAOPHOT). Each
stage can be run on it's own. The PHOTRED program is actually just a
giant wrapper for the 13 stages (and the ones specified in the
"photred.setup" file) run in the correct order. If you ever want to
just run ONE stage then it's probably easier to just run that stage at
the command line instead of editing the "photred.setup" file and
running photred.

It's preferable to run the very first stage, PHOTRED_RENAME, by itself
from the command line and double-check the results. This stage
prepends a string to each FITS filename that indicates what field it
is (i.e. obj1101.fits -> F1-obj1101.fits), and it creates a "fields"
file that has the actual field name for each field "shortname". This
is information that PHOTRED_MATCH needs to match up the various ALS
files for each field/chip group. It's very important that the files
are renamed properly. So check closely the text that is
output. PHOTRED_RENAME uses the first "word" in the OBJECT FITS
keyword as the field name. Any
zero/flat/twilight/sky/pointing/focus/test frames and standard star
frames (SA98, SA110, SA114 and NGC3680) are moved to a "calib/"
directory.

It's a good idea to run PHOTRED_RENAME in "testing" mode, so you can
see how it will rename files without it actually doing anything. Just
type "photred_rename,/testing". The first thing PHOTRED_RENAME does is
check that all of the FITS header parameters can be found (readnoise,
gain, ut-time, filter, exposure time, ra, dec, date, and airmass). If
any of these cannot be found then it will spit out errors. Watch for
these! Check that the actual field names (not the "shortnames") in
"fields" are correct. This information is not used until PHOTRED_SAVE
renames the final photometry files. **Make sure to change the "fields"
file ONLY AFTER running PHOTRED_RENAME in the NORMAL mode.** In testing
mode PHOTRED_RENAME will write the field information to
"fields.testing" instead of "fields".

If the files were not renamed properly, rename them by hand and update
the "fields" file. Also, update the "logs/RENAME.outlist" file. It
might be easiest to delete the "logs/RENAME.outlist" file and remake
it by typing "ls F*.fits > logs/RENAME.outlist".