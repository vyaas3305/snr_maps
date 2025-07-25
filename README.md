# SNR Maps from the SPOTLIGHT Pipeline
This repository contains beamwise Signal-to-Noise Ratio (SNR) Map plotting code that validates the true candidates and their SNR distribution over sky coordinates by visuallizing the SNR across multiple beams. The SNR maps can be plotted at various stages of the pipeline (i.e. from the raw filterbank data, after AstroAccelerate, after clustering, after classification). This code is a part of the SPOTLIGHT FRB detection pipeline.

## Usage
To run the code, use the following commands:

* For running the pulsar folding script - `folding.sh`:

Modify the paths for the pulsar parameter file and the directory for beam-wise filterbank files. Run the shell script as `./folding.sh`

* For plotting the folded SNR map - `process_pfd_plot.py`:

Fill all the observational parameters correctly in the configuration file `config.yaml` and then execute the code as `python process_pfd_plot.py`. Make sure to source the correct Python environment that includes all the required libraries.

* For plotting the raw data SNR map - `make_raw_snr.py`:

Fill in all the observation-specific paths in the `config.yaml` file (including the path of the .h5 file of the verified burst candidate and output directory) and execute the code as `python make_raw_snr.py`. Make sure to run this on a GPU node (since `candies` is run), along with sourcing the required environment

* For plotting the stage-wise SNR maps - `make_snr_post*.py`:

Fill all the observation-specific paths in the `config.yaml` file. Ensure that the pipeline has run successfully for the input observation/scan and execute the code as `python make_snr_post*.py`. Make sure to run this code on a free compute node (CPU nodes are preferred, as code doesn't involve GPU usage), after sourcing the correct environment.

* For plotting the beam pattern maps - `make_beam_pattern.py`:

Fill the observation-specific paths in the `config.yaml` file and execute the code as `python make_beam_pattern.py`. Ensure that the correct environment is sourced before executing the script.

## Description

###  `process_pfd_plot.py`
This script extracts Signal-to-Noise Ratio (SNR) information from pulsar folding results across all observed beams and visualizes their distribution across the sky. It is tailored for `.pfd` files generated by _prepfold_, a tool from the _PRESTO_ ([Pulsar REsearch Software Toolkit](http://www.cv.nrao.edu/~sransom/presto/)) software package, widely used in pulsar data analysis. Since `.pfd` files do not include sky coordinates, the script retrieves RA and DEC values from associated header files and creates a unified pandas DataFrame containing Beam Index, RA, DEC, and SNR for plotting. It also reads observation details and plotting settings from a user-provided config.yaml file, which must be placed in the same directory as the script.

Additionally, the script supports a comparison between observed and simulated SNR distributions. It accepts a simulated SNR map in .txt format—generated externally by a beam simulation tool—and overlays it with the observed results. This enables detailed visual comparison through a set of plots: the Folded Profile SNR Map, optional Folded Profile Data, Simulated SNR Map, and Residual SNR Map.

### `make_beam_pattern.py`
This script provides a simple tool to visualize the beam layout from header files generated by the `SPOTLIGHT` pipeline. It plots the spatial arrangement of telescope beams on the sky using right ascension and declination data extracted from the observation metadata. The central beam (phase center) is also highlighted to assist with interpretation.

All necessary parameters — including the scan ID, observation path, and desired output directory — are read from the `config.yaml` file. The resulting plot is saved to the specified output location.

This utility is useful for verifying the beam tiling used during observations, and for quickly checking if the metadata appears correct before further analysis.

### `make_raw_snr.py`
This script generates a Signal-to-Noise Ratio (SNR) map for a verified burst by tracing candidates from pipeline outputs on raw filterbank data. It uses `candies` for feature extraction and plots beam-wise SNR maps highlighting the beam with maximum SNR. This script is specifically designed to work on the output file structure of the SPOTLIGHT pipeline.

The script extracts the required candidate metadata, generates the appropriate input `.csv` for the `candies` feature extractor, and runs a shell script to produce `.h5` files for each beam. These are then loaded and processed to compute the SNR for each beam. The final output is a PNG file (`rSNR.png`) that plots these values over their corresponding sky positions, highlighting the beam with the maximum SNR and the telescope's phase center.

Please make sure that the `make_h5_candies.sh` script is executable before running the main script. If a `candydates.csv` already exists in the output directory, the program will skip generating a new one and reuse the existing file. Also ensure that the directory structure of your data (such as FilData/, FRBPipeData/) matches the expected format. Failure to maintain these conventions may result in path resolution errors or incorrect file handling.

### `make_snr_post*.py`
These scripts provide utilities for generating beam-wise SNR scatter plots at various stages of the `SPOTLIGHT` pipeline. The scripts read candidate data files and observation metadata to highlight beams with strong detections of transient signals like FRBs. They focus on pinpointing and visualizing the spatial distribution of candidates, especially those temporally and spectrally close to a known burst event (the verified candidate burst)

The scripts assume that the observation files and candidate HDF5 data are correctly formatted and accessible at the specified paths. Be cautious with the configuration parameters, especially the DM and time thresholds, as overly narrow values may exclude relevant beams, while overly broad ones may include unrelated candidates. Always verify the content of `config.yaml` before execution.

**NOTE**: These scripts depend on helper functions defined in `snr_plt_utils.py`, which must be present in the same directory or accessible in the import path.

