ReadMe for NEON DICEE Dataset
@authors: Zach Butler & Marja Haagsma
Date Edited: 04/10/2022 

Associated data paper: [add reference here]

Abstract

This directory contains datasets of and scripts to reproduce the National Ecological Observation Network Daily Isotopic Composition of Environmental Exchanges (NEON-DICEE). 


Description of script folder 

Folder name: 'DICEE Code'
Description: Scripts 'Iso_Precip.R', and 'Estimate_daily_p_flux_iso.py' were used for modeling daily isotopic composition of precipitation found in folder 'P Files'. 

	Name: 'Iso_Precip.R'
	Description: R Script (version 4.1.3) to download raw NEON Isotopes in Precipitation ratios and Precipiation amounts (mm).
		input:
		- dpID addresses for Isotopes in Precipitation (dpID="DP1.00038.001", both d18O and d2H) and Precipitation (dpID="DP1.00006.001")
			- See url's here: https://data.neonscience.org/data-products/DP1.00006.001 and https://data.neonscience.org/data-products/DP1.00038.001 
		- 'siteNamexxx' (Iso or Precip) station ID for the two seperate name lists for Isotopes in Precipitation and Precipiation data. All Isotopes in Precipitation data has Precipiation data
		- 'Path_xxx' (Iso or Precip) where the user wants the files saved too
		process:
		- Loops through each siteName ID list to download raw NEON data
		- Merges raw data into one csv file per siteName ID
		output:
		- Output csv files coordinate with each siteName ID to organize the raw data

	Name: 'Estimate_daily_flux_iso.py':
	Description: Python Script (version 3.8) to calculate the daily downscaling of Isotopes in Precipitation (d2H and d18O)
		Dependencies: - 'precip_iso_downscaling_scripts.py'
		input:
		- Downscaling method from 'precip_iso_downscaling_scripts.py' to produce estimates of d2H and d18O isotopes in precipitation
		- 'site_list' for all sites with both Istopes in Precipitation and Precipitation data
		process:
		- Runs 100 ensembles of downscaling method for a defined period of interest (whole record of data used)
		- Take the mean and standard deviation (std) of 100 ensembles for d2H and d18O downscaled files
		output:
		- Four csv files (two mean and two std for d2H and d18) plus one metadata file of quality flags (see flags at bottom of this document)

Description: Scripts 'mixing_model_d2H.R', 'mixing_model_d18O.R', 'mixing_model_d13C.R', and 'et_nee_flux.py' were used to estimate daily flux of d2H, d18O, and d13C found in folders 'ET Files' and 'NEE Files', respectively.

	Name: 'mixing_model_xxx.R' (where 'xxx' is the stable isotope)
	Description: R-script (version 4.1.3) to calculate the flux for the given stable isotope
		input:
		- 'inpath' where the calibrated hdf5 files for each of the NEON sites are stored.
		- 'radfile' file with downwelling radiance to determine which time-window a datapoint belongs to (daytime or nighttime).
		- 'outpath' where the flux data will be stored
		- 'which_part' variable for the stable isotope (either 'H2', 'O18', or 'C13')
		process:
		- reads in calibrated stable isotope file and applies the Miller-Tans regression to estimate the slope and uncertainty for each time-window.
		output:
		- three csv files with flux and uncertainty for each of the time-windows.

	Name: 'et_nee_flux.py'
	Description: Python script (version 3.8) to create three files: data file, error estimate, and metadata with quality flags of the flux of the given stable isotope.
		input:
		- 'C13_iso_file',' H2_iso_file', and 'O18_iso_file' paths and names of the csv files generated in 'mixing_model_xxx.R' (where 'xxx' is the stable isotope).
		- 'Outpath' where the output will be stored.
		process:
		- reads in flux files and reshapes is to a matrix with dates for rows and sites for columns
		- extracts the error estimate to determine the uncertainty of the flux
		- assigns quality flags based on the criteria described in section on 'ET Files'
			- criteria for the quality flags can be altered within the script
		output:
		- per isotope, nine csv files will be generated (for the three time-windows there will be three files each: data, error, and metadata)		
	


Description of dataset folders

Folder name: 'ET Files'
Description: Contains 18 csv files for daily d2H and d18O flux for three time-windows: 1) alltime (this dataset uses all data points), 2) daytime (this dataset only uses data points where downwelling radiation >= 10 Wm-2), and 3) nighttime (this dataset only uses data points where downwelling radiation < 10 Wm-2).
	For each time-window we have three associated files; 1) the data file, 2) the estimated absolute error in ‰, and 3) the metadata file with the quality flags. 
	
	- data file ('daily_et_flux_xxx_yyy.csv', where 'xxx' is the stable isotope and 'yyy' is the time-window):
	Contains isotopic composition of the daily flux in ‰. Each row in these files represents one calendar date, indicated in the first column. The columns are labeled with the NEON site abbreviation codes. A value of -9999 indicates that no calibrated isotope ratios were available. 

	- error file ('daily_et_flux_xxx_yyy_error.csv', where 'xxx' is the stable isotope and 'yyy' is the time-window):
	Contains standard error of the Miller-Tans regression slope as the uncertainty of the composition of daily flux in ‰.

	- metadata file ('daily_et_flux_xxx_yyy_metadata.csv', where 'xxx' is the stable isotope and 'yyy' is the time-window):
	Contains quality flags for each of the daily flux data points. Interpretation of quality flags:
	
		1	no calibrated isotope ratios were available, and a value of -9999 was used in the data file
		2	low number (n ≤ 5) of calibrated isotope values used to generate the daily flux values
		3	Miller-Tans model r-squared (R2) was below 0.9
		4	inferred isotope value was beyond the 1.5 times IQR of the 25th percentile (Q1 - 1.5 IQR) and 75th percentile (Q3 + 1.5 IQR) of the observation data
		0	good quality, passed criteria described above
		

Folder name: 'NEE Files'
Description: Contains nine csv files in same fashion as folder 'ET Files' but for d13C daily flux to estimate the net ecosystem exchange.


Folder name: 'P Files'
Description: Contains five csv files and one folder for daily precipitation isotopic composition for d2H and d18O.
		
	- data file ('daily_p_xxx_mean.csv' and 'daily_p_xxx_std.csv', where 'xxx' is the stable isotope):
	Contains mean and standard deviation of the modeled time series of isotopic composition corresponding to observed precipitation in ‰. Each row in these files represents one calendar date and the columns are labeled with the NEON site abbreviation codes. Days with no precipitation contain values of -9999.

	- metadata file ('daily_p_metadata.csv')
	Contains quality flags for the data files. Interpretation of quality flags:

		0 	observed precipitation was larger than trace event (≥ 0.25 mm)
		1	no precipitation data on that day
		2	observed precipitation was less than trace event (< 0.25 mm)

	- P_Ensemble folder
	Contains 200 csv files of the the individual runs of the model. 100 runs for d2H and d18O each. These runs are used to create the mean and standard deviation files. 
