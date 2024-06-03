# UFO Dev Container

This repository documents the VS Code Dev Container setup for UFO software stack. The associated Dockerfile in the
`.devcontainer` directory would be built, which would setup the dev container. We need to figure out, how to bring the
stuff for reconstruction into the workspace.

Following are some example codes, which could be used for a quick check on the functionaility of the UFO software inside
the container.

```bash
# Cropping with ufo-launch query
$ ufo-launch -q read path="radios" ! crop width=[width] height=[height] x=[horiz_coord] y=[vert_coord] ! write filename="radios/cradios.tif" bytes-per-file=1000000000000 tiff-bigtiff=True

# Flat field correction - can be used with --absorptivity flag if no special treatments such as spit removal is required
$ tofu preprocess --projections radios/radios.tif --darks darks --flats flats --fix-nan-and-inf --output fc.tif --output-bytes-per-file 1000000000000 --dark-scale 1.0 --flat-scale 1.0 --projection-filter none

# Finding Scintillator Spots ###
$ tofu find-large-spots --output-bytes-per-file 0 --number 1 --output spot_mask.tif --method median --median-width 20 --dilation-disk-radius 3 --image flats --gauss-sigma 0. --spot-threshold 3500 --spot-threshold-mode absolute --grow-threshold 100 --find-large-spots-padding-mode mirrored_repeat

# Spot Removal
$ ufo-launch -q [read path="fc.tif", read path="./spot_mask.tif"] ! horizontal-interpolate ! write filename="interp.tif" bytes-per-file=1000000000000 tiff-bigtiff=True

# Transmission Image -> Absorption Image
$ ufo-launch -q read path='interp.tif' ! calculate expression="'(isinf (v) || isnan (v) || (v <= 0)) ? 0.0f : -log(v)'" ! write filename='abs.tif' bytes-per-file=1000000000000 tiff-bigtiff=True

# Generating Sinogram
$ tofu sinos --number 3000 --projections abs.tif --output sinos.tif --output-bytes-per-file 1000000000000

# Stripe filter - if required
$ ufo-launch -q read path="sinos.tif" ! pad width=4096 height=8192 x=1040 y=2596 addressing-mode=mirrored_repeat ! fft dimensions=2 ! filter-stripes horizontal-sigma=100.0 vertical-sigma=1.0 ! ifft dimensions=2 ! crop width=[crop_width] height=3000 x=1040 y=2596 ! write filename="sl_sinos_out.tif" bytes-per-file=1000000000000 tiff-bigtiff=True

# Optimize rotation axis
# Normally z-value = 1008
$ tofu reco --projections interp.tif --number 3000 --center-position-z [z-value] --z-parameter center-position-x --region=[min, max, delta] --output alignment.tif --overall-angle 180

# Reconstruct
$ tofu tomo --sinograms sinos.tif --output reco-abs.tif --output-bytes-per-file 1000000000000 --axis [optimized_rotation_axis]

# Phase Retrieval
# Input to the phase retrieval is always transmission image. Algorithm automatically computes the logarithm for the absorption
$ tofu preprocess --projections fc.tif --output fc_pr.tif --retrieval-method=tie --energy=13.5 --propagation-distance=0.1 --pixel-size=6.11e-6 --regularization-rate=1 --delta=1e-6  --retrieval-padded-width=4096 --retrieval-padded-height=4096 --retrieval-padding-mode=clamp_to_edge --thresholding-rate=0.01 --frequency-cutoff=1e+30 --projection-filter none

$ tofu sinos --number 3000 --projections fc_pr.tif --output sino_pr.tif --output-bytes-per-file 1000000000000

$ tofu tomo --sinograms sino_pr.tif --output reco_pr.tif --output-bytes-per-file 1000000000000 --axis [optimized_rotation_axis]
```
