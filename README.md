# QFitsView DPUser
## Setting Up DPUSER Functions Library
1. Place all libraries (files with “lib\_\*.dpuser” name) in a convenient location (e.g. “/Users/*username*/DPUser/Functions"
2. Place “startup.dpuser” in that location. This assumes that all the library functions are located in the "Function" sub-folder and runs the "lib\_all.dpuser" script. This file must be modified for your own requirements; it also sets the *DPUSER_DIR* environment variable that can be accessed by other scriupts, using the **getenv** function in QFitsView.
3. Create a directory under root “/dpuserlib” (for macOS 10.15+ use the `synthetic.conf` symbolic links - reference [here](https://stackoverflow.com/questions/58396821/what-is-the-proper-way-to-create-a-root-sym-link-in-catalina))
4. Make a file in that directory “startup.dpuser” - this will be automatically run when you start QFitsView. The file consists of a single line:
 `@/Users/*username*/DPUser/startup.dpuser`
i.e. it runs the script set up above. This runs all libraries to make the functions available to QFitsView - you will see a whole bunch of “Stored function…” and “Stored procedure…” plus “Finished General Functions”.
5. If the above folder conventions are not used, the following files must be modified:
    -  /dpuserlib/startup.dpuser
    - /Users/*username*/DPUser/startup.dpuser
    - /Users/*username*/DPUser/Functions/lib\_all.dpuser

##  Global Variables
These are defined internally; they can be overwritten in a QFitsView session.

c          =     299792458.0<br>
pi         =     3.14159<br>
e          =     2.71828<br>
naxis1     =     256<br>
naxis2     =     256<br>
plotdevice =     /XSERVE<br>
method     =     0<br>
tmpmem     =     20971520<br>

## Editing DPUser Code

The main documentation for DPUser is through [this link](https://www.mpe.mpg.de/~ott/dpuser/). DPUser code can be edited with QFItsView *DPUSER > Script Editor*. It can also be edited by various external text editors; e.g. **BBEdit**. To facilitate this, a language module has been implemented - "DPUser.plist", with the following highlighting features: 

- Syntax - both structural commands - e.g. "if", "else" etc. and internal DPUser functions/procedures. As new functions/procdures are implemented in QFitsView, the "BBLMPredefinedNameList" array must be updated.
- Comments (both for "/\*..\*/" and "//").
- Strings ("..")
- Function and procedure prefixes

The file is installed in BBEdit's language module directory - by default "/Users/*username*/Library/Application Support/BBEdit/Language Modules".

Note that after editing code in an external text editor, the script must be executed again with QFitsView.

##  Parameter data types
* FITS or buffer input - "cube" is 3D, "image" is 2D, "spectrum" is 1D, "data" is 1, 2 or 3D
	* Pixel co-ordinates
	* p, p1, p2,… (general pixels co-ordinates)
	* x, x1, x2,…, y, y1, y2,…, z, z1, z2,… (for x, y or z axes)
	* pixel/wavelength masking [x1,x2,x3,x4...] - paired pixel numbers or wavelengths (pairs [x1..x2],[x3..x4] etc.) for spectral masking
* WCS co-ordinates
	* wcs (for axis set [CRPIXn, CRVALn, CDELTn])
	* w, w1, w2,…. (for individual coordinates)
	* l, l1, l2, …. (wavelengths)
* Maps
	* velmap - velocity map cube (either standard or extended format)
	* wvtmap - weighted Voronoi tessellation map (region numbers)
## Libraries
### lib_all

Runs all the following libraries; this just consists of script lines to execute other scripts in the "Functions" sub-folder. Other scripts can be executed by adding appropriate lines, e.g.

`@SomeFolder/SomeScript.dpuser`

### lib_wcs

Transform to and from World Coordinate Systems and Pixels

**function get_WCS_values, spectrum** - create WCS array [1,cv,cd] from dispersion *spectrum*

**function get_WCS_data, data, axis** - return array [CRVAL, CRPIX, CDELT] for *axis* (1,2 or 3) of *data*

**procedure set_WCS_data, data, wcs, axis** - sets WCS values for *data* for *axis* (1,2 or 3)

**function cvt_pixel_WCS, pix, crpix, crval, cdelt** - convert pixel number *pix* to WCS coordinates using *crpix, crval, cdelt*. Assumes linear conversion (cartesian). Future versions will use **worldpos** and **pixpos** functions (to be implemented as QFitsView internal functions in a future version).

**function cvt_WCS_pixel, value, crpix, crval, cdelt** - convert WCS coordinate *value* to pixel using *crpix, crval, cdelt*. 

**function cvt_WCS_pixel_data, data, value, axis** - converts pixel *value* to WCS for *data* for *axis*

**function cvt_pixel_WCS_data, data, value, axis** - reverse of *cvt_WCS_pixel_data*

**function WCS_range, data, p1, p2, axis, prntflag** - returns WCS coordinates as range [*w1,w2*] from pixel values [*p1,p2*] for axis on *data*, if *prntflag*=1, then print range

**function pixel_range, data, w1, w2, axis, prntflag**  - returns pixel values as range [*p1,p2*] from WCS co-ordinates [*w1,w2*] for axis on *data*, if *prntflag*=1, then print range

**function set_WCS_default, data** - checks *data* has minimal WCS keys set (to 1 by default)

**function get_WCS_image, image** - Get axis 1 and 2 WCS data for *image* (can be cube) and calculates CDELT and rotation angle from CD keys.

**function set_WCS_image_scale, image, wcs, xscale, yscale** - Rescales (e.g. for non-integer re-binning) using *xscale* and *yscale* and sets WCS data for *image* (or cube), including CD keys - removes CDELT and CROTA2 keywords

### lib_general
General functions

**function indexreform, index, xsize, ysize, zsize** - returns 3D co-ords from 1D *index*, given dimensions *xsize, ysize, zsize*. Values returned as array.

**function lognan, data** - set log of *data*, setting zero and Nan values to Nan

**function clipnan, data, low, high** - set values outside range [*low..high*] to Nan

**function axiscentroids, image, axis** - returns centroids of each image row/column, row-*axis*=1, column-*axis*=2; used e.g. for finding centroids of pv diagram

**function myhist, data, low, high, bin, normflag** - create a histogram from inbuff data (any dimensions), from _low_ to _high_ values in *bin* bins. Histogram is normalised if *normflag* =1. Output x-axis values are set to range.

**function profile_export, data, scale1, scale2, scale3, offset** - Export 1D profiles from *data* with up to 3 separate scales, e.g. arcsec, pc, Re plus the pixel scale, _offset_=1 offsets by 1/2 a pixel (e.g. for log scale plot) (default 0). Scales default to 1 if not given.

**function butterworth_filter, order, cutoff, size** - Create a Butterworth filter for order _order_ for a square of sides _size_, with _cutoff_ Nyquist frequency

### lib_cube
Data cube functions

**function cube_trim_xy, cube, x1, x2, y1, y2**- sets cube to zero for x<*x1*, x>*x2*, y<*y1*, y>*y2* (**cblank** cube first)

**function cube_trim_wl, cube, w1, w2, value, trimflag** - sets *cube* to *value* (usually zero) for l<*l1*, l>*l2* in axis 3 using WCS (cblank cube first). If *trimflag*=1, then truncate the cube outside the wavelength range.

**function cube_spectrum_mask, cube, mask, level** - mask *cube* on spectral wavelength with *mask* (pixel pairs), set masked pixels to *level*

**function cube_clip, cube, lvl, thresh, mask** - clips *cube* <0 and > *lvl* in image (x-y) plane, does dpixapply using threshold, cube is spectrally masked

**function cube_clip_y, cube, lvl, thresh** - as above, but in the x-z plane    

**function cube_interp_z, cube, x1, x2, y1, y2, z1, z2** - interpolate in image plane over rectangle [*x1:x2,y1:y2*] in each of wavelength plane [*z1:z2*]

**functioncube_interp_x, cube, x1, x2, y1, y2, z1,z2** - interpolate in image plane over rectangle [*y1:y2,z1:z2*] in each of spatial range [*x1:x2*]

**function cube_interp_y, cube, x1, x2, y1, y2, z1,z2** - interpolate in image plane over rectangle [*x1:x2,z1:z2*] in each of spatial range [*y1:y2*]

**function cube_interp_xy, cube, x1, x2, y1, y2, z1,z2** - as above, but interpolate over wavelength [*z1:z2*] in the xy plane

**function cube_set_value, cube, x1, x2, y1, y2, z, xv, yv** - set rectangle [*x1:x2,y1:y2*] at image plane *z* to value at [*xv, yv*]

**function cube_pixfix_xy, cube, pixfixdata, n** - fix cube using cube_interp_.. functions, n sets, pizfixdata are [x1,x2,y1,y2,z1,z2,method] where method=“x”/“y”/“xy”

**function cube_single_pixel_fix, cube, x, y** - cube_interp_xy for all z axis for single spaxel

**procedure cube_bit_nan, cube,x,y**  - set spaxel [x,y] to 0/0 along whole cube

**function cube_clean_dpix, cube, divisor** - Clean cube inbuff by dpixcreate/apply, divisor =  scale set from maximum of median image

**function cube_resize_center, cube, xcen, ycen, xsize, ysize** - resize *cube*  to size *xsize*,*ysize* and center on pixel [*xcen,ycen*]

**function cube_shift_xy, cube, xshift, yshift** - sub-pixel shifts *cube* by [*xshift, yshift*]

**function cube_redisp, cube, val, delt, n** - change the wavelength dispersion of *cube* (axis 3) to new range defined by *val*, *delt* and *n* by interpolation.

**function cube_symm_flip, cube, lambda, width, part** - symmetrically flip *cube* about wavelength *lambda*, *part* =0 (left) or 1 (right), trims cube to *lambda*+-*width*

**function cube_rotate, cube, xcen, ycen, platescale, rot_angle** - rotate cube on center [xcen,ycen] by rot_angle, setting platescale in arcsec/pixel.

**function cube_centroids, cube** - get centroids at each wavelength pixel

**function cube_centroids_gauss, inbuff, xe, ye, we, mask** - get centroid at each pixel layer, with estimated center at [*xe,ye*] over fitting window *we*. The spectrum is masked by *mask* (if not zero or not entered).

**function cube_centroid_gauss_align, inbuff, xc, yc, xe, ye, we, mask** - align cube centroids at each pixel layer, using [*xe, ye, we, mask*] are the estimate parameters of the peak (as for **cube_centroids_gauss**), with the centroids aligned to [*xc, yc*].

**function cube_cont_slope, cube, mask** - returns image with continuum slope, masked by wavelength pairs

**function cube_spectrum_subtract, cube, spectrum** - subtract spectrum for each spaxel

**function cube_spectrum_divide, cube, spectrum** - divide cube by spectrum (e.g. telluric correction)

**function cube_spectrum_multiply, cube, spectrum** - multiply cube by spectrum

**function cube_set_pixlayers, cube, pixl, pix1, pix2** - set cube layers [pix1,pix2] to the values for layer pixl

**function cube_wavelength_correct, cube, correction** - correct wavelength solution at each spaxel by “correction” values (in wavelength)

**function cube_to_2d, cube** - Convert data cube to 2d apertures for IRAF

**function cube_set_flags_nan, cube, layer** - set up flags image for cube_interp_flags, from a data cube (e.g. a velmap) from layer. This sets 1 where pixel in “NaN”, 0 else.

**function cube_interp_flags, cube, flags, xi1, xi2, yi1, yi2, dmax** - interpolate over pixels in cube where flags is set to 1, 0= good values to use for interpolation. [xi1:xi2, yi1:yi2] is region to interpolate (xi1 = 0 - do whole area). dmax is maximum distance from “good” pixels.

**function cube_deslope, cube, mask, wlflag** - deslope cube for each spectrum using “spectrum_deslope”. wlflag = 1 if mask values in wavelength

**function cube_clean_pixels, cube, layer, npix** - Remove singleton pixels surronded by Nan’s, Opposite of “cube_interp_flags", used to clean up boundaries etc. npix is max number of good pixels around each pixel before blanking.

**function cube_radial_spectrum, cube, xc, yc, rstep, nstep, ann** - Radial spectra of cube, centered [xc,yc] radial steps rstep, number of steps nstep. If ann=1, output annular spectra

**function cube_rebin, cube, psize** - Rebin cube to pixel size psize (arcsec). Uses “interpolate” function. Useful for e.g. KCWI data which has rectangular spaxels on the sky.

**function cube_from_image_spectrum, image, spectrum** - Creates a cube from an image and spectrum. Wavelength axis of cube is spectrum scaled by image value
### lib_image
Image functions

**function image_erodenan, image** - erode *image*, pixels set to Nan if any neighbour is Nan

**function image_smooth, image, smooth** - smooth *image* with NaN values - *smooth* integer=boxcar, non-integer=gaussian

**function image_interp_x, image, x1, x2, y1, y2** - as for **cube_interp_xy**, but for single *image* 

**function image_interp_y, image, x1, x2, y1, y2** - as for **cube_interp_x**, but for single *image* 

**function image_interp_xy, image, x1, x2, y1, y2** - as for **cube_interp_y**, but for single *image* 

**function image_from_profile, profile, xp, yp, xc, yc** - create 2D image from 1D *profile*, size of output image is *xp* x *yp* , [*xc, yc*] - center of rebuilt profile

**function image_bfilter, image, order, cutoff** - Butterworth filter an *image*, assume square image, filter order *=order*, *cutoff* =Nyquist cutoff (0-1)

**function image_enclosed_flux, inbuff, xc, yc, r, smth** - Get enclosed flux within radius *r* from [*xc, yc*] (pixels). If *smth*>0, Gaussian smooth the output 

**function image_avg, image, x, y, s** - average value of image in square aperture [*x,y*] +-*s* pixels

**function image_structure, image, psf_image** - Structure map = *image*/(*image* ⊗ *psf*) x *psf*^T, "⊗”=convolution, "^T" = transpose

**function image_interp_flags, image, flags, xi1, xi2, yi1, yi2, dmax** - Interpolate *image* over flagged spaxels, *flags* - 2D data with same x/y axes size as image, with value=1 to be interpolated, value=0 - good pixels, [*xi1:xi2, yi1:yi2*] - co-ordinate range to interpolate over. If not input, then do all spaxels. *dmax* - maximum pixel distance for interpolation (=0 don't test)

**function image_cut, image, x, y, a** - does **twodcut** at [*x,y*] angle *a* and reset WCS correctly

### lib_spectrum
Spectrum functions

**function spectrum_make_disp, val, delt, pix, n** - make 1D vector over range defined by WCS *val*, *delt*, *pix*, *n*.

**function spectrum_make_disp_data, data, axis** - make 1D vector over range defined by WCS values from *data* axis (1,2 or 3)

**function spectrum_make_disp_n, val1, val2, n** - make 1D vector over range [*val1:val2*], number of points *n*

**function spectrum_mask, spectrum, mask, value, wlflag** - *spectrum* set to *value* between pixel pairs in *mask*. Works for 1D or 3D, assuming last axis is spectrum. *wlflag* =0, mask is in pixels, =1, mask is in wavelength. *value* is usually 0/0 to blank masked sections for polynomial fitting.

**function spectrum_cont_slope, spectrum, mask, wlflag** - continuum slope of *spectrum*, masked by wavelength *mask*/*wlflag* (as for **spectrum_mask**)

**function spectrum_deslope, spectrum, mask, wlflag** - deslope spectrum, using **spectrum_cont_slope** and *mask/wlflag* parameters

**function spectrum_polyfit, spectrum, order, mask, wlflag** - fit polynomial of *order* to  *spectrum* with *mask*, *wlflag*. Returns n x 3 array, 1st row=original data masked, 2nd row=polynomial fit, 3rd row = residual

**function spectrum_symm_flip, spectrum, lambda, part** - split *spectrum* at wavelength *lambda*, flip and add, taking left (*part*=0) or right (*part*=1) sections

**function spectrum_wave_to_lambda, wndata** - convert wavenumber spectrum *wndata* to wavelength (nm) with same axis length

**function spectrum_make_gauss, spectrum, bi, bs, h, l, w** - make spectrum with gaussian from spectrum WCS. *bi, bs* - base intercept and slope, *h* -  height, *lc* - center wavelength, *w* - FWHM (creates artificial emission line)

**function spectrum_redisp_lin, spectrum, data, daxis, xmin,delt, npix, zero, norms** - re-disperse a *spectrum* to the same wavelength scale/range as *data* (with wavelength axis *daxis*). If *zero* is >0, all pixels outside of the wavelength range of data are set to zero. If *norms* is >0, the output is normalised. If *data* is not a spectrum (ie. nelements(*data*[*daxis*])=1) redisperse with parameters *xmin, delt, npix*.

**function spectrum_from_xy, spectrum** - re-disperse *spectrum* from 2D x and y bintable to wavelength range and same number of points.

**function spectrum_from_data, xdata, ydata, w1, w2, delt** - re-disperse spectrum from 2D x and y data to wavelength range [*w1, w2*] with step *delt*

**function spectrum_interp, spectrum, x1, x2** - Smooth over bad pixels [*x1:x2*]

**function spectrum_sn, spectrum, window** - Estimate spectrum S/N from itself - not 100% accurate but good for comparisons, *window* is smoothing and noise estimation window. Returns vector of same length as *spectrum* with S/N estimate, blank where *spectrum* is 0.

### lib_io
Input/output to and from text and fits files

**function io_text_FITS_1D, bintable2d** - converts string array buffer *bintable2d* with format of "wavelength, data" to spectrum fits data, setting WCS values. Assumes wavelength is evenly spaced. Note **import** function of QFitsView does very similar (with more parameters).

**function io_text_FITS_3D, bintable2d, nx, ny, nz, blank**  - converts string array buffer *bintable2d*, with format of “i,j,v1,v2..." to fits data cube size [*nx,ny,nz*]. Default value for resulting cube is *blank* (e.g. 0 or 0/0) - can have missing [*i,j*].

**function io_text_FITS_interp, fname, xstart, xdelta, xnum, xscale, yscale, ignore** - converts text from file *fname*, with format of "wavelength, data" to spectrum fits data, setting WCS values. The values are interpolated to the range defined by *xstart*, *xdelta* and *xnum*. Wavelength and data value are scaled by *xscale*, *yscale* (default 1). *Ignore* lines at the start are skipped (e.g. column headers).

**function io_FITS_text_1D, spectrum, prefix, cutoff** - converts *spectrum* to text, CSV format, line 1 = “*prefix*\__Wavelength, *prefix*\_Counts”. Values below *cutoff* (non-zero) are set to “NaN” (Be aware of QFitsView Edit > Copy functionality)

**function io_FITS_text_2D, image, prefix** - converts *image* to text, CSV format, line 1 = “*prefix*\_Wavelength, *prefix*\_Flux_1, *prefix*_Flux_2 …. "

**procedure io_FITS2TXT_1D, fname, cutoff** - converts 1D FITS to text file *fname* assuming file is in  working directory - output is same as input file with “.txt” type. *Cutoff* as for **io_FITS_text_1D**.

**procedure io_FITS2TXT_2D, fname** - as above but for image (2D) file

**function io_cube_from_xyz, cube,bintable2d, n** - make a cube from *bintable2d*, *cube* is template, resized to *n* on axis 3,  first 2 values in data are x,y co-ords, rest are values along z axis

**function io_import_TXT_1D, fname** - import data from file *fname* in text format

### lib_masking
Masking functions for images and cubes

**function mask_from_image, image, level, low** - create a mask from data *image*, setting to 1 if > *level*, to *low* (usually 0) if \<*level*

**function mask_from_image_nan, image, zero** - create a mask from data *image*, setting to 1 if  data value<>Nan. If *zero* = 0 or 1, set mask to Nan or 0 at Nan values.

**function mask_data, image, level, low** - masks data *image*, setting to *low* if < *level*

**function mask_data_median, image, level, low** - as above, but sets data image > *level* to median of *image*

**function mask_circle, data, x, y, r, v, rev** - masks *data* (image or cube) with circle center [*x,y*] radius *r*, set masked-out value to *v* (default 0). If *rev*<>0, reverse mask.

**function mask_set_nan_min, data, minvalue** - set *data* values to *minvalue* if value = Nan. If minvalue is zero, use the current minimum value. Equivalent to **cblank** function is *minvalue*=0

**function mask_cone, data, xc1, yc1, xc2, yc2, pa, beta, maskflag** - mask cone area, equator [*xc1, yc1*], [*xc2, yc2*] (can be same coordinate), centerline angle *pa*, internal full-angle *beta*. If *maskflag*=0, return the mask, if *maskflag*=1, return the masked input data

**function mask_line, data, x1, y1, x2, y2, side** - creates a mask of *data* dimensions on one side of a line [*x1, y1*], [*x2, y2*]. *side* =0 for left, =1 for right side of line

### lib_velmap
Velocity map (velmap) extension functions

**function velmap_std_to_ext, velmpstd, r, cmin, vmethod, vzero, vx, vy** - convert standard QFitsView velmap *velmpstd* to extended form, *r*=instrumental resolution, *cmin*= minimum continuum value - output is extended velmap format - see below. Velocity zero is set by *vmethod* =

- 0 - median   
- 1 - average
- 2 - flux-weighted average
- 3 - manual (*vzero* value)
- 4 - pixel ([*vx,vy*] is set to zero)

**function velmap_vel_center, velmap, vmethod, vcenter, vx, vy** - returns the wavelength value from the velmap cube, using the methods as above

**function velmap_vel, velmap, vmethod, vcenter, vx, vy** - returns the velocity map from the velmap cube, using the methods as above

**function velmap_vel_set, velmap, vmethod, vcenter, vx, vy** - Fix extended velmap velocity as per velmap_std_to_ext (re-do extended velmap cube)

**function velmap_rescale, velmapext, scale** - rescales extended velmap flux data (e.g. flux calib change)

**function velmap_fix, velmap, contlo, conthi, flo, fhi, vlo, vhi, wlo, whi, setvalue** - clean up velmap (either standard or extended form), setting values out of range to setvalue. Value ranges 
- contlo, conthi - continuum
- flo, fhi - flux
- vlo, vhi - wavelength
- wlo, whi - fwhm
- setvalue - value to set where spaxel is out of range (usually 0/0)

**function velmap_extcorr, velmp, av, lambda** - extinction correct velocity map *velmp* at wavelength *lambda* (in nm), *av*=extinction A_V

**function velmap_extcorr_map, velmp, extmap, lambda** - as above, but *extmap* is a map of extinction values

**function velmap_fix_interp, velmp, npix** - interpolate velmap *velmp* missing values, missing is Nan in continuum (usually after **velmap_fix**). *npix* is interpolation width maximum

**function velmap_clean_map_wvt, velmp, map, nregion** - Clean up velmap *velmp* based on WVT *map* region number, setting region *nregion* pixels to NaN

**function velmap_mask, velmp** - set *velmp* to Nan where continuum=0

**procedure velmap_comps, velmp, prefix, hmax** - Output velmap components from *velmp*, to the current working directory. *prefix* (string) sets file names, terminated with \_Flux/\_Flux\_Norm/\_Vel/\_EW/\_Sig/\_VelHist/\_SigHist - Histograms (VelHist and SigHist) are generated if *hmax*>0, with velocity range -*hmax* -> *hmax*, dispersion 0->*hmax* with 50 bins

**function profit_to_velmap, profit_data, velmpstd** - convert PROFIT cube format (see Riffel, R. A. 2010, Astrophys Space Sci, 327, 239, http://arxiv.org/abs/1002.1585) to standard velmap format.

### lib_chmap 
Channel and position/velocity map functions

**function chmap_create, cube, lambda_cent, lambda_width, cutoff, width_factor, smooth** - make a channel map from the cube
- *lambda_cent* - estimate of central wavelength
- *lambda_width* - estimate of FWHM
- *threshold* - % of maximum for cutoff
- *width_factor* - wavelength widow (multiple of lambda_width)
- *smooth* - integer=boxcar, non-integer=gauss, 0=no smoothing 

Returns cube, axis 3 in velocity difference from median. Spaxel values are FLUX (not flux density) in that channel

**function chmap_rebin, cube, lnew, velwidth, sm, minval**- rebin channel maps in *cube* into *lnew* bins between velocities *v1* and *v2* (usually symmetric about 0, but not necessarily), with *sm* smoothing value, integer=boxcar, non-integer=gauss, 0=no smoothing, set output to NaN where < *minval*

**procedure chmap_comps, cube, dirout, fnameout** - splits channel map into components and writes images to dirout, named fnameout plus velocity (e.g. if *fnameout* ="pa_beta-450”, then output file name will be e.g.  "pa_beta-100.fits" etc.

### lib_pv
Position Velocity Diagram functions

**function pv_array, cube, ystart, wslit, nslit, lcent, lwidth** - create pv diagram from *cube* parallel to x axis, *ystart* - y pixel to start, *wslit* - slit width, *nslit* - number of slits, extract over range *lcent*-*lwidth* to *lcenter+lwidth*

**function pv_single, cube,  xc, yc, angle, width, lcent, vwidth, npix, contflag** - extract single PV plot at *xc*/*yc*/*angle*/*width* - centerered on *lcent*. *vwidth* - velocity width around *lcent*, rebinned in velocity to *npix* channels. *contflag* =1 subtract continuum (flux) =2 divide continuum (effectively the same as equivalent width) =0 don’t remove continuum

**function pv_ratio, cube,  xc, yc, angle, width, lcent1, lcent2, vwidth, npix** - create PV diagram as above for ratio of 2 lines *lcent1*, *lcent2*

**function pv_meddev, image** - divide *image* by median along x axis (useful for EW for PV diagrams)

### lib_wvt
Weighted Voronoi Tesselation functions

**function wvt_cube, cube, sn_target** - make wvt *cube* using noise in each spaxel, to *sn_target*. Bad pixels where S/N is > 10x brightest pixel S/N

**function wvt_cube_mask, cube, l1, l2, mask, cutoff, sn1, sn2** -  make WVT cube using 2 S/N ratios, inside and outside *mask*. Returns WVT applied to *cube*

- *l1, l2* - wavelength range to use for signal and noise determination (“quiet” part of spectrum with no emission lines)
- *mask* - if 2D mask, use this. If *mask*=0, use *cutoff* to determine mask
- *cutoff* - percentage of peak maximum for mask level
- *sn1*, *sn2* - S/N ratios for inside/outside mask. If *sn2*=0, just use *sn1* over whole cube

**function wvt_sn_mask, cube, l1, l2, mask, cutoff, sn1, sn2** - as for **wvt_cube_mask**, except returns WVT image data - layers:

1. Signal
2. Noise
3.  S/N
4.  Mask
5.  Signal binned
6.  Signal bin map
7.  Bin density (1=maximum - smallest bins , 0=minimum - biggest bins)

**function wvt_build_from_map_cube, cube, wvtmap, prntflag** - make WVT cube from *cube* and *wvtmap*. If *prntflag* =1 print diagnostic every 100 regions

**function wvt_build_from_map_image, image, wvtmap** - make WVT image from *image* and *wvtmap*.

**function wvt_velmap, velmp, layer, sn** - make WVT velmap from standard or extended velmap *velmp*, *layer* is either 0=continuum, 1=flux, *sn*=S/N target

**function wvt_density, wvtmap** - make map of region density, i.e. 1/# of pixels in region. *wvtmap* is WVT with /map flag.

**function wvt_cube_to_specarray, cube, wvtmap, normflag, prntflag** - convert *cube* inbuff to spectrum array, using *wvtmap* regions. If *normflag* = 1, divide each spectrum in array by the first one. If *prntflag* = 1, print running diagnostics

**function wvt_specarray_to_cube, image, wvtmap** - reverse of **wvt_cube_to_specarray**

### lib_astronomy

General astronomy functions, implemented from IDL.

**function G** - gravitational constant (MKS)

**function Msun** - Mass of the sun in kg

**function Pc** - 1 Parsec in meters

**function airtovac, wave** - Convert air wavelengths to vacuum wavelengths, *wave* in Å

**function planck, wave, temp** - Calculates the Planck function in units of ergs/cm2/s/Å. *wave* in Å, *temp* in degrees K.

**function coordstring, ra, dec, rad** - create a nice string of celestial coordinates. *ra* and *dec* should be given in radians; if *rad*>0, convert to degrees

### lib_astro_general

General astrophysics functions. All the "lib\_astro\_*.dpuser" functions are executed from the "lib\_astro.dpuser" script.

**function redshift_data, data, z, rdflag** - redshift data by *z*, assuming last axis is wavelength. WCS values set. If *rdflag* = 1, redisperse shifted data to same wavelength range as input

**function bb_make,t, l1, l2, npix, wlflag** - make black-body function at temparture *t*, wavelength range *l1* to *l2*, number of pixels *n*, *wlflag*=0, wavelength in A, =1=> nm, =2 => um

**function bb_make_log, t, l1, l2, npix, scale, cutof**f - make bb at temp *t* over log wavelength [*l1,l2*] (in log meters), creating spectrum length *npix*, multiply wavelengths by *scale* (to convert to e.g. nm), set result to Nan where below *cutoff*. 

**function bb_div, spectrum, temp** - divide *spectrum* by black-body at temperature *t*

**function extinction_calc, f1, f2, l1, l2, rat, galext, s, flmin1, flmin2**- create an extinction map from 2 emission line maps - *f1*, *f2*  are flux maps, *l1*, *l2*=wavelengths, *rat*=expected flux ratio, *galext*=galactic extinction, *smth*= smooth pixels, *flmin1*, *flmin2*=minimum flux value for each map - calculates the extinction constant (CCM laws for IR and optical)

**function extinction_correct, cube, av** - correct *cube* for extinction (*av* =single value for extinction) - wavelength from axis 3

**function extinction_correct_map, cube, av** - correct *cube* for extinction (*av* =extinction map) - wavelength from axis 3.

**function extinction_correct_lambda, data, av, lambda** - correct value/image for extinction *av* at wavelength *lambda*: can be used on value or image

**function flux_to_mag, image, zpm, ssize, zpflag** - convert flux *image*  to mag, *zpm* is either zero-point magnitude (*zpflag*=0) or zero-magnitude flux (*zpflag*=1). *ssize* = pixel size in arcsec to convert to mag/arcsec^2

### lib_astro_mapping
Astronomy functions (mapping)

**function map_compare_diagram, image1, image2, min1, max1, min2, max2, nbin, lgaxesflag** - Map diagram density plot. *image1*, *image2* - value maps, x and y axes. *min1/2*, *max1/2* - min and maximum values for axes 1/2. *nbin* - no of bins on each axis. *lgaxesflag* - 1=plot in log space (min,max must be in log values)

**function map_compare_pos,image1, image2,image3,image4, x, y, boxsize**- get 2 sets of map ratios (*image1*/*image2*, *image3*/*image4*)at position [*x,y*], averaged over *boxsize* x *boxsize* pixels (e.g. excitation ratios at feature position)

**function map_basis_distance, basex0, basey0, basex100, basey100, x1, x2, y1, y2, size** - creates an image (dimensions *size*, limits (*x1, y1*), (*x2, y2*)) of distance from basis points [*basex0, basey0*] to [*basex100, basey100*] - for use in AGN mixing ratios for contour values.

**function map_compare_basis,image1, image2, basex0, basey0, basex100,basey100,lgaxesflag** - plots basis distance (AGN mixing ratio) from basis points [*basex0, basey0*] to [*basex100, basey100*] . *lgaxesflag* - 1=take log of *image1*, *image2* before calculation

**function map_regime_ir, image1, image2, a1, a2, a3, b1, b2** - create position excitation map. If a1=0, use the standard Riffel 2013 excitation regimes. inbuff1 is H_2/Br_gamma, inbuff2 is [Fe II]/Pa_beta. Both in log values. Output values are SF=1, AGN=2, LINER=3, TO1=4, TO2=5

**function map_regime_optical, image1, image2, typeflag**- create position excitation map for optical line ratios (*image1* and *image2*). *typeflag* = 1 ([N II]/H_alpha diagram), =2 ([S II]/H_alpha diagram), =3 ([O I]/H_alpha diagram). Returns 1=SF, 2=Seyfert, 3=LINER, 4=Composite

### lib_astro_spectrum
Astronomy functions (spectrum)

**spec_fluxdens, spectrum, l1, l2, prflag** - flux density (counts/nm) for *spectrum* between *l1* and *l2* <u>wavelength</u>; returns a single value. If *prflag*<>0, print results as well.

### lib_astro_image
Astronomy functions (image) 

**function img_aphot_annular, image, xcen, ycen, r, ib, ob** - aperture photometry on *image*,centered on [*xcen,ycen*]; aperture *r*, background annulus from *ib* to *ob* (inner to outer boundary). If *ib* and *ob* are zero, set to r and 2*r

**function img_apphot_simple, image, xcen, ycen, r, pixsize, scale** - simple aperture photometry on image,centered on  [*xcen,ycen*]; aperture *r*.

### lib_astro_cube
Astronomy functions (cube) 

**function cube_apphot, cube, xcen, ycen, r1, r2, mx, my, mr** - aperture photometry, centered on xcen,yceb; aperture r1, background annulus r2, mask out circle mx/my/mr (mx>0) - result is spectrum

**function cube_apspec, cube, ox, oy, or, bx, by, br, br2, mask** - Get star spectrum from *cube* withusing circular aperture and background, plus a mask circle. 

- *ox, oy, or* - center and radius of aperture
- *bx, by, br* - center and radius of mask, if not required then bx = 0
- *br2* - inner radius of mask annulus 
- *mask* - any other mask required (2D fits)

Output is spectrum of aperture less average of background (with mask) - values <0 set to Nan.

**function cube_fluxdens, cube, l1, l2, prflag/function spec_fluxdens, spectrum, l1, l2, prflag** - flux density (counts/nm) between *l1* and *l2* wavelength; returns image (cube\_) or single (spec\_) value. If *prflag*<>0, print results as well

**function cube_sky_rem, cube, bckgnd_lvl** - removes skylines from *cube*. Takes background pixels as those with median value below *bkgnd_lvl*

**function cube_sl_clean, cube, skyline_list, width** - removes skylines from *cube* using skyline_linelist (array of wavelengths) - interpolated over wavelength ± *width*

**function clean_cube_bp_fix, cube, bp_cube** - cleans *cube* based on bad pixel cube *bp_cube* using **dpixapply** over x image slices

**function clean_cube_bp, cube, threshold** - create bad pixel cube using threshold scanning over wavelength slices.

**function clean_cube_bp_limits, cube, ll, ul** - create bad pixel cube from *cube* for input to **clean_cube_bp_fix**, flagging pixels below *ll* and above *ul* values

# Standard Procedures
## Velocity maps
* Create QFitsView **velmap** with wavelength, fwhm estimate
* Examine velmap for continuum, height, wavelength and fwhm “sensible” ranges
* Use **velmap_fix** to clean up velmap, entering "good" ranges for each fit component. This sets out-of-range spaxels to Nan.
* Use **velmap_fix_interp** to interpolate over NaN values (if required)
* Use **velmap_std_to_ext** to create extended velmap format
### Standard VELMAP Format

1. Continuum
2. Peak height above continuum
3. Wavelength
4. FWHM
5. e_Continuum
6. e_Peak
7. e_Wavelength
8. e_FWHM
9. Chi-squared
### Extended VELMAP Format
1. Continuum
2. Peak height above continuum
3. Wavelength
4. FWHM
5. e_Continuum
6. e_Peak
7. e_Wavelength
8. e_FWHM
9. Chi-squared
10. Velocity (zero-point calculated/set by *method* parameter in the **velmap_std_to_ext** function)
11. Dispersion (sigma) velocity, corrected for spectral resolution
12. Flux (Peak\*FWHM*1.0699)
13. Equivalent width (flux/continuum)
14. Total support (order + turbulence) (√V^2 +σ^2)
15. Order vs turbulence (|V/σ|)

## Channel maps

- Create basic channel map using “**chmap_create**” (usually do not smooth)
- Rebin to required # of channels (e.g. 9 or 16) using “**chmap_rebin**” (smoothing if required)
- Output individual channel maps using “**chmap_comps**

### 