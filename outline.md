### Background
- When we initiated the HLS global processing effort in 2019, we wanted to
  select the simplest to maintain, compile and manage LaSRC implementation to
  use in our processing containers.
- This led us to USGS created and maintained ESPA LaSRC implementation. It
  included code infrastructure for building images for compilation, converting
  Landsat and Sentinel data to a common interchange format and utilities for
  obtaining and generating auxiliary water vapor and aerosol datasets.

### Reducing risk in the HLS project
One of the largest areas of risk and technical integration difficulty in the HLS project is the reliance on external teams for core portions of the scientific processing code. This is risk is most evident with our use of the USGS ESPA maintained LaSRC library.

- A single engineer responsible for maintaining the entire codebase.
- Python wrappers for C functions that use older patterns and cannot be used with Python > 3.8
- A reliance on several deprecated dependencies for compilation.
- No open license or public repository where issues can filed and tracked and community contributions can be added.
- Significant compute performance issues with the heuristics used for failed aerosol retrieval.
- Significant hardcoding in the product formatter https://github.com/NASA-IMPACT/espa-product-formatter codebase which converts upstream USGS and ESA distribution formats into a common interop format and requires significant updates anytime the providers introduce new changes.
- Inability for a single version of the codebase to use different sources of aerosol data.  In this case, there are discrete code paths for processing data with either VIIRS or MODIS derived auxiliary data which do not interoperate.
- No regular, controlled release cycle.
- No regular, controlled performance regression testing process.
- No regular baseline testing against a set of defined reference images.
- The HLS algorithmic C code makes assumptions about the internal HDF file structure of the LaSRC results and needs to be refactored to work with any updates to the LaSRC output.

### Technical coordination with GSFC
SNWG has provided funding to Dr. Vermote and his team to
- Develop algorithmic improvements to the USGS / ESPA LaSRC codebase to address reflectance performance over problematic targets
- Asses the feasibility of utilizing alternative sources for auxiliary data
  (specifically the NOAA Global Data Assimilation System (GDAS)) and modify the
  ESPA codebase to support this.

### Science code != production code
The main problems around risk and friction for maintaining the ESPA LaSRC
codebase and incorporating contributions from GSFC really boils down to a gap in
the robustness of and rigor for engineering practices.  As a team maintaining a
production data system we need assurances and validation that 

- A change to algorithmic code is isolated in a well structured pull request
  that we can selectively test, apply and if necessary rollback.
- That we can automatically compile and deploy any code updates with limited
  changes to our continuous integration pipelines.
- 
- Our codebase is flexible enough to support data format changes from upstream data
  providers (USGS/NASA and ESA) with very simple modifications.
