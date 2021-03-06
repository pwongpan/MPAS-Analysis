# Eddy Kinetic Energy Climatology Mapping

<h2>
Kevin Rosa <br>
date: 2018/06/18 <br>
</h2>

## Summary

The document describes a new feature which will be added to the MPAS-Analysis
tools package: visualization of surface Eddy Kinetic Energy (EKE).
The EKE climatology map will function very similarly to other climatological fields (e.g. SSH, SST, etc.).
The output file will contain three images: the modeled EKE climatology, the observed EKE climatology, and the difference.
Plotting EKE is particularly important for MPAS-O because one can configure meshes with eddy-permitting regions and would then want to compare the EKE in these regions against observations.

## Requirements

1. Model output must contain the meridional and zonal components of both `timeMonthly_avg_velocity*` and `timeMonthly_avg_velocity*Squared`.
1. User can download the EKE observations data, via 1 of 2 methods:
  - Run `./download_analysis_data.py -o /path/to/output/directory` if they wish to download all observations data.
  *or*
  - Download only the EKE dataset at [https://web.lcrc.anl.gov/public/e3sm/diagnostics/observations/Ocean/EKE/drifter_variance.nc](https://web.lcrc.anl.gov/public/e3sm/diagnostics/observations/Ocean/EKE/drifter_variance.nc)
1. In config file...
  1. Specify `ekeSubdirectory` with location of EKE observations file.
  1. Under `[climatologyMapEKE]`, leave `seasons =  ['ANN']`.  *Only annual observations are available currently.*
  1. When setting `generate`, task `climatologyMapEKE` has tags: `climatology, horizontalMap, eke`


## Physics

In the ocean, it is convenient to separate the the horizontal current, *u*,
into its mean and eddy components:
(1) <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?u=\overline{u}+u'" title="u = ubar + uprime" /></a>

This approach separates the total kinetic energy into mean kinetic energy
(MKE) and eddy kinetic energy (EKE).

The EKE over much of the ocean is at least an order of magnitude greater than
the MKE (Wytrki, 1976).
This eddy energy is important for transporting momentum, heat, mass, and chemical
constituents of seawater (Robinson, 1983).

## Algorithms

Time mean of equation 1: <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\overline{u^2}=\overline{u}^2+\overline{u'^2}" title="u = ubar + uprime" /></a>

The model outputs <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\overline{u}" title="ubar" /></a>
and <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\overline{u^2}" title="u-squared bar" /></a>
while the observational dataset provides <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\overline{u'^2}" title="u-prime-squared bar" /></a>
so two different EKE equations must be used:

(2) <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\overline{EKE_{model}}=(\overline{u^2}-\overline{u}^2+\overline{v^2}-\overline{v}^2)/2" title="EKE model" /></a>

(3) <a href="" target="_blank"><img src="https://latex.codecogs.com/gif.latex?\overline{EKE_{obs}}=(\overline{u'^2}+\overline{v'^2})/2" title="EKE observations" /></a>



## Design and Implementation
The primary design consideration for this feature is that it integrate
seamlessly with the rest of the analysis tools.
To this end, the sea surface temperature (SST) plotting tools will be used as a
template.

Files to create:
- `mpas_analysis/ocean/climatology_map_eke.py`
- `docs/tasks/climatologyMapEKE.rst`
- `README.md` for `drifter_variance.nc` dataset

Files to edit:
- `mpas_analysis/ocean/__init__.py`
- `docs/analysis_tasks.rst`
- `docs/api.rst`
- `mpas_analysis/config.default`
- `mpas_analysis/obs/analysis_input_files`

The main challenge for plotting EKE is that EKE is a function of several model variables and is not itself a variable that is directly written by the model.
Because of this, the climatology mapping functions for SSH, SST, SSS, and MLD will not serve as a direct template for the EKE formulation in `mpas_analysis/ocean/climatology_map_eke.py`.
I will try to follow the structure of `mpas_analysis/ocean/compute_transects_with_vel_mag.py` as much as possible.

It appears that there is a method for plotting velocity magnitudes on the antarctic grid.  Look into 'climatology_map_sose.py'...

## Testing

I will test runs of varying durations and resolutions to make sure the EKE plotting is working.  I will also ensure that the following jobs fail:
1. Input model results files missing at least one of the 4 necessary velocity variables.
1. Request seasonal plots.
1. Test that `./download_analysis_data.py` downloads EKE data.


## Bibliography

- https://latex.codecogs.com/eqneditor/editor.php
- Chelton, D. B., Schlax, M. G., Samelson, R. M. & Szoeke, R. A. de. Global observations of large oceanic eddies. Geophysical Research Letters 34, (2007).
- Laurindo, L. C., Mariano, A. J. & Lumpkin, R. An improved near-surface velocity climatology for the global ocean from drifter observations. Deep Sea Research Part I: Oceanographic Research Papers 124, 73–92 (2017).
- Wyrtki, K., Magaard, L. & Hager, James. Eddy energy in the oceans. Journal of Geophysical Research 81, 2641–2646



