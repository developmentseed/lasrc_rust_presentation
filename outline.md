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

- All changes are validated by fully automated testing against a suite of
  test granules to ensure that there has been no fidelity or performance
  regression associated with a code change.
- All code are validated against an automated unit testing suite to ensure
  structural and semantic correctness.
- A change to algorithmic code is isolated in a well structured pull request
  that we can selectively test, apply and if necessary rollback.
- That we can automatically compile and deploy any code updates with limited
  changes to our continuous integration pipelines.
- Our codebase is balancing resource utilization and compute performance with
  output fidelity.  Obviously a system that produces perfect surface reflectance
  values is optimal but impractical if it takes many hours to run.
- Our codebase is flexible enough to support data format changes from upstream data
  providers (USGS/NASA and ESA) with very simple modifications.
- The language and structure we employ are friendly to newer engineers who are
  onboarded to the project.
- Releases are tagged so that deployments in production are pinned to specific
  versions of the code and this is included in product metadata.


### Our vision - A Community Driven Atmospheric Correction Model
The goals described above reduce several of the HLS project's largest risk factors and friction points around LaSRC integration. But even with these enhancements we would still be reliant on a legacy codebase maintained principally by a very small team of developers. Ideally we would prefer a library with a large community of maintainers led by a consortium of organizations that provide continuous improvements to the codebase and can quickly react to upstream source product changes from ESA and NASA

To achieve this vision we proposed the following steps.

- Create a Rust port of the existing ESPA / USGS LaSRC C library. The adoption of Rust provides equal if not superior numerical performance while increasing language safety and maintainability. In addition, the increasing momentum around Rust in academic and engineering communities will ensure we have a larger pool of potential new maintainers who would otherwise be less interested in working on a large legacy C codebase.

- Create an archive of reference Landsat and Sentinel images to be used for reflectance target performance assessment and memory/CPU performance assessment to track algorithmic improvements and regressions.

- Create a repository of fully automated Jupyter notebooks used to compare and analyze LaSRC outputs so that we can automate inter-comparison testing of our Rust code and we continually test algorithmic updates to evaluate their performance and validity.


###  The arrival of a robot army 
My initial estimate for building and testing this Rust LaSRC port was 5-6 months
of 1 FTE effort.  Given the size of our production team and the continuous
demands of operating a production system while piloting other SNWG solutions it
was difficult to prioritize the time to tackle this.

But as we investigated the ESPA LaSRC code updates we would need to make to add
alternative auxiliary data support for the new low-latency solution
requirements, we realized that it might be more efficient to add this support in
new, simpler Rust codebase.

Luckily, this has coincided with a massive growth in the coding capabilities of
LLMs and the increased adoption of multi-agent harnesses to decompose
engineering tasks into work streams that can be executed in parallel. 

Using several Claude Code sessions we had agents attempt to replicate the ESPA
LaSRC codebase as closely as possible.  By continually comparing the agentic
code's output against a reference ESPA LaSRC granule output we were able to make
adjustments and refactor portions of the code the output was within the sensor's
noise floor threshold.

### A robust validation suite
Since the HLS project's start, the production team has been requesting a large
scale, automated validation suite from the science team.  This would allow us to
safely incorporate algorithmic code changes while assessing the effects of these
changes across a wide representative set of targets.

In addition, enabling resource utilization logging when generating the outputs
to use in the validation suite allows us to estimate and assess an performance regressions that algorithmic changes might have.


Rather than rely on external teams to manage our validation testing, Trang Vo and Madhu Sridhar have been continually building and refining this system.
For the LaSRC Rust port validation we are using https://github.com/NASA-IMPACT/hls-application/blob/lasrc_validation/notebooks/LaSRC_container_validation.ipynb

This notebook suite compares band outputs from both the ESPA LaSRC C codebase
and the LaSRC Rust port.  This generates a series of plots and tabular data
frames to demonstrate the statistical differences between the outputs in a
repeatable, objective way.

Based on feedback from the science team, we have selected 5 DN as the
inter-comparison threshold so that we have a well defined target we are working
to achieve for alignment between the ESPA and Rust LaSRC outputs.

### Humans in the loop
This initial round of AI assisted development provided a strong foundation.
The results on our initial test granules demonstrated good agreement.  But
thanks to the validation suite, we now have a automated approach to compare a
larger set of granules. When we executed the suite against the full test granules we
discovered a host of bugs.  Chris Holden started incrementally tackling these
with a combination of agent driven refactoring and manual review.  There were a
host of inconsistencies and bugs to address, but working in an agentic loop
drastically accelerated this process.  To highlight some of the problems that
were addressed

- The ESPA LaSRC C codebase was very inconsistent with type conversion in many
  areas.  Our port attempted to faithfully replicate many of these conversions
  which resulted in residual accumulation differences.
- The Rust port chose a different approach for invalid auxiliary retrievals.  
  The Rust port had set the transmittance to 0 instead of to the closest non-fill neighbor.
- The Rust port was arguably more accurate in defining the scene-center geometry by calculating it, while the C version grabbed the SZA from metadata and set VZA and relative azimuth to 0.
- The Rust port had bi-linearly interpolated to get the scene center pressure/O3/WV, while C did a nearest neighbor.

### Where are we now?
After working to iteratively address all of the issues we identified in the port
we've reached the following results

- 100% alignment between the Rust and ESPA C LaSRC outputs (within the 5 DN threshold for) for all of the granules in "golden granules" reference set.
- The Rust port runs 30% faster with 7% less memory.

In many cases during our initial LLM based porting work, the model chose
alternative implementations that caused alignment differences.  In many cases
these alternative implementations were selected because the ESPA LaSRC codebase
had bugs or inconsistencies.  Now that we've demonstrated inter-comparison
equivalence we'll be able to individually address these problems.

### What is next?
Now that we've demonstrated inter-comparison equivalence with the Rust and C
code we have created a plan on how to prioritize continuing work on the Rust
port.  In rough order of priority 

- The largest rationale for undertaking this port was to reduce the level of
  effort for switching to GDAS based auxiliary data to support the HLS
  low-latency solution.  We'll begin testing and evaluation of this code path in
  this PI.
- The structure of the code in the Rust port reflects the original ESPA C
  structure.  We'll incrementally refactor and restructure the Rust code to
  improve maintainability and make it easier for other contributors to work on
  the project. 
- Performance optimization.  Initially, we've made some targeted performance
  tweaks within the Rust port but there are likely a large number of additional
  performance change we can apply to reduce runtime, resource utilization and
  overall processing costs.

With our new Rust structure, the time for implementing and testing these new
features and performance improvements is drastically reduced from what would be
required with the ESPA LaSRC codebase.

### Collaboration
Going forward, the HLS production team will manage the Rust port.  We will
continue to work on structural and performance changes.  LaSRC is an evolving
algorithm and scientific and algorithmic changes will continue to be driven by
the GSFC team.  Our hope is to use this approach for incorporating new
algorithmic changes

- Dr. Vermote and the GSFC team can prototype and test these changes within the
  Fortran environment (or wherever is most convenient for them).
- They capture proposed changes in a formal specification document which
  describes the algorithm changes in detail.
- The HLS production team implements this specification document in the Rust
  code and uses the validation suite to evaluate the changes and their potential
  effects on processing outputs.

