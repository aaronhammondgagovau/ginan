
# Overview of the PEA

The software execution of the Parameter Estimation Algorithm (PEA), written in C++, will be largely sequential - using threads sparingly to limit the overhead of collision avoidance. Where possible tasks will be completed in parallel using parallelisation libraries to take advantage of all cpu cores in multi-processor systems while still retaining a linear flow through the execution.

Sections of the software that create and modify global objects, such as while reading ephemeris data, will be executed on a single core only. This will ensure that collisions are avoided and the debugging of these functions is deterministic.

For sections of the software that have clear delineation between objects, such as per-receiver calculations, these may be completed in parallel, provided they do not attempt to modify or create objects with more global scope. When globally accessible objects need to be created for individual receivers, they should be pre-initialised before the entry to parallel execution section.

## Data Input and Synchronisation

Before the processing of data from an epoch is initiated, all other relevant data is accumulated. As this code affects global objects that have effects in multiple places, this code is run in a single thread until data processing is ready to begin.

### Config

Configurations are defined in YAML files. At the beginning of each epoch the timestamp of the configuration file is read, and if there has been a modification, new parameters in the configuration will be loaded into memory.

### Product Input

Various external products may be required for operation of the software, as defined in the configuration file. At the beginning of each epoch, if any product input files have been added to the config, or if the inputs are detected to have been modified, they will be re-read into memory.

### Metadata Input

Metadata such as GNSS ephemerides are available from external sources to augment the capability of the software. This data is ingested at the beginning of each epoch before processing begins.

### Observation Data Input

Observation data forms the basis for operation of the software. Observations from various sources are synchronised and collated at the beginning of each epoch before processing begins. The software uses class inheritance and polymorphism such that all data type inputs are retrieved using a single common instruction, with backend functions performing any retrieval and parsing required.

Observation data is synchronised by timestamp - when the main function requests data of a specific timestamp all data until that point is parsed (but may be discarded), before the observation data corresponding to that timestamp being used in processing.

### Initialisation of Objects

During the following stages of processing many receiver-specific objects may be created within global objects. To prevent thread collision in the global objects, the receiver-specific objects are created here sequentially.

# Preprocessor

The preprocessor is run on input data to detect the anomalies and other metrics that are available before complete processing of the data is performed. This enables low-latency reporting of issues, and prepares data for more efficient processing in the later stages of operation.

* Cycle Slip Detection
* Low Latency Anomaly Detection
* Missing Data
* Output / Reporting

## Precise Point Positioning

The largest component of the software, the PPP module ingests all of the data available, and applies scientific models to estimate and predict current and future parameters of interest.

Version 1 of the GINAN toolkit satisfies many of the requirements for GNSS modelling, but has been achieved by incrementally adding features as they became available and as scientific models have been developed. Many of the components make assumptions about the outputs of previous computations performed in the software, and require care before adding or making changes to the code, or even setting configuration options.

It is intended that the software will be reorganised with the benefit of hindsight, to remove interactions between modules and explicitly execute each processing step in a manner similar to an algebraic formulation used by experts in the field.

It is tempting for researchers to apply heuristics or corrections that may have been historically used to assist in computation, but these must be limited to effects that can be modelled and applied through the Kalman filter, in order to maintain the efficiency and robustness that it provides.

As the models required for ‘user’, ‘network’, and even ‘ionosphere’ modes are equivalent, the only distinction between the modes is the extent of modelling to be applied, which can be reduced to a simple configuration change. As such the parallel streams within the software will be eliminated and reduced to a single unified model, with example configurations for common use-cases.

### Force and Dynamic Models

At the beginning of processing of an epoch, parameters with time-dependent models are updated to reflect the time increment since the previous epoch. Simple models will be well defined when initialised, but more complex models will require updating at every epoch.

Ultimate positioning performance largely depends on accurate dynamic models, with development of these models improving predictive capability, and reducing uncertainty and adjustments at every point in time.

* Gravity
* Solar Radiation Pressure
* Other

### Orbit and State Prediction

Before the available observations for an epoch are utilised, a prediction is made of the parameters of interest by utilising the previous estimates and applying dynamic models through the Kalman filter’s state transition.

### Phenomena Modelling + Estimation

In order to accurately estimate and predict parameters of interest, all phenomena that affect GNSS/SLR observations must be isolated and modelled, as being components comprising the available measurements.

Where the values of parameters are well known they may be used directly to extract other parameters of interest from the data - such as using published corrections to precisely determine a user’s position.

When data is unavailable, or when it is desired to compute these products for subsequent publication and use, estimates of the values are derived from the available data.

It is the sophistication of the models available and applied that determines the ultimate performance of the software.

The software will be developed to allow for all applicable phenomena to be modelled, estimated, such that user’s desired constraints may be applied and parameters of interest extracted.

### Initialisation of Parameters

Estimation parameters are initialised on the point of first use, automatically by the Kalman filter module. Their initial value may be selected to be user-defined, extracted from a model or input file, or established using a least-squares estimation.

### Robust Kalman Filter

It is well known that the Kalman filter is the optimal technique for estimating parameters of interest from sets of noisy data - provided the model is appropriate.

In addition, statistical techniques may be used to detect defects in models or the parameters used to characterise the data, providing opportunities to intervene and make corrections to the model according to the nature of the anomaly.

By incorporating these features into a single generic module, the robustness that was previously available only under certain circumstances may now be automatically applied to all systems to which it is applied. These benefits extend automatically to all related modules (such as RTS), and often perform better than modules designed specifically to address isolated issues.

For further details about the software's robust Kalman filter see the Ginan Science Manual.

### RTS Smoothing

The intermediate outputs of a Kalman filter are of use for other algorithms such as RTS smoothers. All intermediate values required for such algorithms are to be recorded in a consistent manner, suitable for later processing.

For further details about the software's RTS smoothing algorithm see the Ginan Science Manual.

### Integer Ambiguity Resolution

GNSS phase measurements allow for very precise measurements of biases but require extra processing steps to disambiguate between cycles. Techniques have been demonstrated that perform acceptably under certain conditions and measurement types, but require substantial bookkeeping and may not easily transfer to different measurement applications.

For further details about the software's ambiguity resolution algorithms see the Ginan Science Manual.

### Product calculation

In order for estimated and predicted values to be of use to end-users, they must be prepared and distributed in an appropriate format.

Some parameters of interest are not directly estimated by the filter, but may be derived from estimates by secondary operations, which are performed in this section of the code.

In this section, data is written to files or pushed to NTRIP casters and other data sinks.


## Post-processing


### Smoothing

The RTS Smoothing algorithm is capable of using intermediate states, covariances, and state transition matrices stored during the Kalman filter stage to calculate reverse smoothed estimates of parameters.

The intermediate data is stored in binary files with messages that contain tail blocks containing the length of the message. This allows for the file to be efficiently traversed in reverse; seeking to the beginning of each message as defined by the tail block.

For further details about the software's RTS Smoothing algorithm see the Ginan Science Manual.


### Minimum Constraints

The minimum constraints algorithm is capable of aligning a network of stations to a reference system without introducing any bias to the positions of the stations.

A subset of stations positions are selected and weighted to create pseudo-observations to determine the optimal rigid transformation between the coordinates and the reference frame. The transformation takes the same algebraic form as a Kalman filter stage and is implemented as such in the software.

For further details about the software's minimum constraints algorithm see the Ginan Science Manual.


### Unit Testing

The nature of GNSS processing means that well-defined unit tests are difficult to write from first-principles. The software however, is capable of comparing results between runs to determine if the results have changed unexpectedly.

Intermediate variables are tagged throughout the code, and auxiliary files specify which variables should be tested as they are obtained, and the expected values from previous runs.

### Logging

Details of processing are logged to trace files according to the processing mode in use.

Per-station files are created with intermediate processing values and information, while a single summary file is generated for the unified filter and combined processing.

Information produced during processing is output to the console, and may also be redirected to other logging sinks such as a database, or json formatted output.

In addition to processing information, inputs may be recorded to file for replaying later.

# Using PEA in network mode

PEA is designed to estimate the GNSS error parameters that cannot be precisely determined. The PEA will estimate the following parameters:

* Correction to satellite initial conditions estimated by the POD component
* Satellite clock offset and drift
* Satellite hardware bias for two signal carriers
* Satellite differential bias for two signal pseudoranges 
* Ionospheric propagation delay
* Tropospheric propagation delay
* Receiver/station position and velocity
* Receiver/station clock offset and drift
* Receiver/station hardware bias for signal carriers
* Receiver/station differential bias for two signal pseudoranges 
* Relative carrier phase ambiguities

In order to estimate the full range of parameters, the PEA will need to ingest GNSS observation data from a Global network of sufficient density. Receiver position, clock and Tropospheric delays can also be estimated from a single receiver and thus will be addressed on chapter \ref{ch:pea_user_mode}.\\

As is the case for the POD, the PEA uses YAML formatted configuration files to set the processing options, and is run using the command:

    ./pea --config <path_to_config_file>

Details on the configuration parameters included in YAML files can be found in chapter \ref{ch:pea_yaml_config}.

With one exception (Ionosphere delay modelling using smoothed pseudorange), processing of network data is activated by setting the `processing_options : process_modes : network` to `true`.
Configuration files corresponding to the examples in this section can be found in the `ginan/examples` directory. A few of these examples are explained bellow.

## Processing a Global Network to Adjust Satellite Positions

PEA is designed to use integrated/predicted satellite positions for its real-time network processing mode, thus all satellite position corrections can only be made in post process mode.
A basic example for PEA network processing is provided in  `ex17_pea_pp_netw_gnss_ar.yaml`. 

In order to activate estimation of satellite position corrections the entry /textit{default_filter_parameters : satellites : orb : estimated} needs to be set to `true`.

It will also require an ICF file, generated by POD in orbit fitting mode, as input. The path to such file should be given in `input_files : orbfiles` entry.

ANTEX files, with antenna information for both stations and satellites should be provided in  `input_files : atxfiles` 

SINEX files, with station antennas should be provided in  `input_files : snxfiles`

RINEX 3.XX navigation files, with broadcast clocks should be provided in  `input_files : navfiles`

BLQ formatted ocean tide loading parameters for each station should be provided in `input_files : blqfiles` if available to correct for OTL, otherwise `processing_options : tide_otl` should be set to `false`.

RINEX 3.XX observation files for the network stations should be provided in `station_data : rnxfiles` (* can be used as a wildcard).

The main output of this processing mode would be an ICF formatted file containing the corrected initial conditions for each of the processed satellites. 
The file will be created in the same path as the input ICF file with the `_pea` suffix attached. 
This file can be used by the POD to integrate/predict the satellite positions over the required time window.

Receiver/station positions can also be estimated as part of the network processing. 
In order to estimate station positions, the parameter `default_filter_parameters : stations : pos : estimated` and `output_files : output_sinex` need to be set to `true`
A SINEX formatted file with the estimated station position will be generated in the path specified as `root_output_directory`.

## Post process estimation of Satellite clocks and biases

The example configuration file `ex17_pea_pp_netw_gnss_ar.yaml` corresponds to a post-mission network processing mode for GPS satellite clocks.

The required input files are similar to the satellite position estimation example.
The main difference will be that SP3 formatted files can be used as input for satellite position (POD generated ICF files can also be used).

The frequency in which the GNSS error parameters, including satellite and receiver clocks, are estimated should be set by the parameter `processing_options : epoch_interval`. 

In order to output the estimated clocks, the parameter  `output_files : output_clocks` needs to be set to `true`.

The satellite and receiver will then be output to a RINEX clock formatted file specified in `output_files : clock_filename`.
 
In order to estimate the satellite and receiver hardware biases, the ambiguities need to be separated from the biases and resolved to integer values. 

In order to perform ambiguity resolution, the proper parameters needs to be set on the `ambiguity_resolution_options` fields. 

The target constellations need to be selected by setting `GPS_amb_resol` and/or `GAL_amb_resol` to true (only GPS and GAL constellation are supported in the current version). 

Both `WL_mode` and `NL_mode` needs to be set to something different than `off`. 

It is advised that `round` or `iter_rnd` be used for network processing. 

In order to output the estimated biases the parameter `output_files : output_biasSINEX` needs to be to `true` and `ambiguity_resolution_options : bias_output_rate` set to a number (of seconds) different than zero.
The satellite biases will then be output to a bias SINEX formatted file specified in `output_files : biasSINEX_filename`.

Receiver biases are also estimated by the process, however they are not reflected on the output files.

## Real-time estimation of Satellite clocks and biases

An example of using PEA for real-time estimation of GPS satellite clocks is provided in `ex17_pea_rt_netw_gnss_ar.yaml`.

The configuration file is similar to that used for post-mission processing.
The main difference is that the input data specified in the `station_data` field will correspond to RTCM formatted streams instead of files.

Currently the PEA can only get real-time data by connecting to an NTRIP caster.
The host name, user name and password corresponding to the NTRIP should be specified under `station_data : stream_root` using the format `http(s)://user:password@hostname/`.

The mountpoint corresponding to station observables need to be listed under `station_data : obs_streams`.

Ephemeris streams (broadcast ephemeris and SSR corrections) should be listed under `station_data : nav_streams`.

Alternatively the mountpoints can be specified using the full path `http(s)://user:password@hostname/mountpoint`, leaving the  `station_data : stream_root` field empty,  this allows to use streams from multiple NTRIP casters.

Satellite positions needs to be provided using (predicted) SP3 files or using real-time streams (broadcast + SSR corrections).

In the `processing_options` field, the `ppp_ephemeris` parameter needs to be set to `precise` is using SP3 files and `ssr_apc` or `ssr_com` if using SSR correction streams.

In the same field, the `epoch_interval` is used to set the update interval of the network solutions, and the `wait_next_epoch` and `wait_all_stations` to help synchronise station streams.

The PEA will wait for `wait_next_epoch` seconds from the start of the previous epoch for the first observation to arrive (and skip the current epoch if no observations arrive).

The PEA will wait for `wait_all_stations` seconds from the first observation for data from other stations before processing.

The estimated clock and bias can be output to an NTRIP caster (as well as to local files). 

The output NTRIP caster streams need to be specified using the `output_streams` field.
The host name, user name and password can be set in the `output_streams : stream_root` parameter.
The names for the output streams should be listed under `output_streams : stream_label`
Once the label is created, the mountpoint and RTCM messages to encode can be specified in the `output_streams : label` field.

Currently the PEA supports output for GPS and Galileo orbits and clock messages (1060 and 1243), code bias (1059 and 1243) and phase bias (1265 and 1267).

## Post process estimation of atmospheric delays

The PEA is capable of estimation both Tropospheric and Ionospheric delays on GNSS signals.
Tropospheric delays can be estimated both in network (`processing_options : process_modes : network = true`) and end-user (`processing_options : process_modes : user = true`) modes. (End-user mode and will be addressed in the next chapter)

Ionosphere delay estimation and mapping is activated by setting `processing_options : process_modes : ionosphere` to `true`.

Two types of Ionosphere delay estimates are supported by PEA. 
If all other parameters in the `processing_options : process_modes` field are set to `false` then the Ionospheric delay are estimated based on carrier smoothed pseudoranges.
If in addition to `processing_options : process_modes : ionosphere`, `processing_options : process_modes : network` is set to true, the PEA will attempt to calculate Ionospheric delay measurements from ambiguity resolved carrier phase measurements. For this mode to work, the parameters in the `ambiguity_resolution_options` field needs to be set properly.
Ionospheric delay estimate are available for GPS signals only.

The Ionosphere slant delay measurements delay measurements can be outputted into STEC files if `output_files : output_ionstec` parameter is set to `true` (the output file name can be set using `ionstec_filename`).

The format of STEC files have one of two forms. If the vector in `ionosphere_filter_parameters : layer_heights` is not empty, each line on the STEC file will contain the slant delay measurements and the piercing points at each layer height:

    #IONO_MEA, 2102,171000.000, AGGO, G05, -1.6030, 1.9699e-04, 2, 1, 350, -33.662, -61.955, 1.443

the fields representing, from left to right:

1. "IONO_MEA" label
1. GPS week
1. GPS TOW in seconds
1. Receiver/station name
1. Satellite ID
1. Slant delay in meters
1. Slant delay variance in meters$^2$
1. Number of Ionosphere layers N
1. N fields containing: 
    1. Height of layer in Km
    1. Latitude of piercing point (in degrees)
    1. Longitude of piercing point (in degrees)
    1. Slant to vertical mapping function

If the layer heights field is empty the "Number of Ionosphere layers" field will be 0 and followed by the receiver position in ECEF, and the satellite position in ECEF.

Vertical TEC (VTEC) maps can be estimated from the slant delay measurements and output as IONEX formatted maps ( `output_files : output_ionex = true`) and its corresponding DCB ( `output_files : output_biasSINEX = true`).

Ionosphere mapping and output are controlled by parameters in the `ionosphere_filter_parameters` field.
Currently only spherical harmonics based mapping is supported by the PEA `model = spherical_harmonic`, setting  `ionosphere_filter_parameters : model` to `meas_out` will output the ionosphere measurements but will not perform mapping. 

If spherical harmonics is selected as mapping method, the ionospheric delays will be mapped into multiple thin layer shells.
The height of the shells can be set in the `ionosphere_filter_parameters : layer_heights` vector.

The VTEC at each layer will be fit to spherical harmonic components, with a maximum order and degree of  `ionosphere_filter_parameters : func_order`.
If  `output_files : output_ionex` is set to `true` the Ionosphere map will be outputted in IONEX 1.11 format.

The area of the IONEX map can be set using the `lat_center`, `lon_center`,  `lat_width`, `lon_width` parameters.
The horizontal resolution of the IONEX map can be set using the `lat_res`, `lon_res` parameters.
The temporal resolution of the IONEX file is defined by the `time_res` parameter.

A configuration file, `ex16_pea_pp_ionosphere.yaml` can be used to generate a single layer IONEX map and accompanying biasSINEX file from smoothed pseudorange observations.

# Using PEA in user mode

When set to end user mode, the PEA component of Ginan will process each station separately. This mode will allow the estimation of parameters available to users with single receivers. 

* Receiver position
* Receiver clock offset
* Tropospheric delay at receiver location
* Ionospheric delay at the receiver location (not yet available)
* Carrier phase ambiguities


In order to use the PEA in end-user mode, the ` processing_options : process_modes : user` parameter needs to be set to `true`.

The results of PEA run in end user mode are printed in the trace files.
Trace file outputs can be activated by setting the ` output_files : output_trace` parameter to `true`.
The most commonly used outputs from the PEA used in end-user mode are expected to be: the receiver position, receiver velocity, receiver clocks and tropospheric delays.

## Receiver position 

Receiver position results are preceded by the "\$POS" label and thus, in Linux, can be extracted using the command:

    grep "$POS" <path_to_trace_file>

the output line for the for receiver position will have 10 comma separated fields with the following format:

    $POS, 2166, 278015.000, 6, -4052053.0060, 4212836.8682, -2545105.0796, 0.0245227, 0.0231919, 0.0163678

the fields represent, from left to right:

 * `$POS` label
 * GPS week
 * GPS TOW in seconds
 * Solution type (6 for float PPP, 1 for ambiguity fixed PPP)
 * Receiver ECEF X position in meters
 * Receiver ECEF Y position in meters
 * Receiver ECEF Z position in meters
 * Standard deviation of ECEF X positions in meters
 * Standard deviation of ECEF X positions in meters
 * Standard deviation of ECEF X positions in meters


## Receiver clock

Receiver clock offset results are preceded by the `$CLK` label and thus, in Linux, can be extracted using the command:

    grep "$CLK" <path_to_trace_file>

the output line for the for receiver position will have 13 comma separated fields with the following format:

    $CLK, 2166, 278015.000, 6, 14, 3.1902, 0.0000, 1.1924, 0.0000, 0.0860, 0.0000, 0.0953, 0.0000

the fields represent, from left to right:

1. `$CLK` label
1. GPS week
1. GPS TOW in seconds
1. Solution type (6 for float PPP, 1 for ambiguity fixed PPP)
1. Number of satellites used in the solution
1. Receiver clock offset for with respect to GPS clock, in nanoseconds
1. Receiver clock offset for with respect to GLONASS clock, in nanoseconds
1. Receiver clock offset for with respect to Galileo clock, in nanoseconds
1. Receiver clock offset for with respect to Beidou clock, in nanoseconds   
1. Standard deviation of clock offset wrt. GPS, in nanoseconds
1. Standard deviation of clock offset wrt. GLONASS, in nanoseconds
1. Standard deviation of clock offset wrt. Galileo, in nanoseconds
1. Standard deviation of clock offset wrt. Beidou, in nanoseconds

If clock offsets for a particular constellation are not available both the offset and its variance will be set to 0.

## Tropospheric delays 

Tropospheric delays at the receiver position are preceded by the `$TROP` label and thus, in Linux, can be extracted using the command:

    grep "$TROP" <path_to_trace_file>

the tropospheric delay solutions will be represented to either a single line, with the `$TROP` or three lines, as follows:

    $TROP, 2166, 278015.000, 6, 14 ,2.294950, 0.0030977
    $TROP_N, 2166, 278015.000, 6, 14, -0.174797, 0.0181385
    $TROP_E, 2166, 278015.000, 6, 14, -0.223868, 0.0250276

each of the troposphere output line will contain 7 comma separated field, of which the first five are:

* Label, `$TROP`, `$TROP_N` or `$TROP_E`
* GPS week
* GPS TOW in seconds
* Solution type (6 for float PPP, 1 for ambiguity fixed PPP)
* Number of satellites used in the solution

The line starting with `$TROP` contain the Zenith Tropospheric Delay (ZTD) and its standards deviation, both in meters, as their last two fields.  The line starting with `$TROP_N` contains the tropospheric delay gradient in north-south direction, and  the line starting with `$TROP_E` contains the tropospheric delay gradient in east-west direction.

Configuration files for specific examples have been added to the `examples` folder in the repository. Examples corresponding to end user processing are explained bellow. In order to 

## Dual frequency PPP with floating ambiguities

As the end-user processing mode cannot calculate satellite states, the satellite position and clock offset needs to be provided externally.

The PEA supports SP3 formatted satellite position inputs, specified in `input_files : sp3files`, and RINEX clock files, `input_files : clkiles`, as satellite clock inputs. 

ANTEX files, with antenna information for both stations and satellites should be provided in  `input_files : atxfiles` 

SINEX files, with station antennas should be provided in  `input_files : snxfiles`

RINEX 3.XX navigation files, with broadcast clocks should be provided in  `input_files : navfiles`

BLQ formatted ocean tide  loading parameters for each station should be provided in `input_files : blqfiles` if available to correct for OTL, otherwise `processing_options : tide_otl` should be set to `false`.

RINEX 3.XX observation files for the network stations should be provided in `station_data : rnxfiles`

The configuration files named `examples/ex11_pea_pp_user_gps.yaml` and `examples/ex12_pea_pp_user_gnss.yaml` set the PEA to calculate a post-process end user solution for a static receiver. 

The constellations to be used in processing can be specified in the `processing_options : process_sys` field.

The tracking of a moving receiver can be done by setting the `default_filter_parameters : stations : pos : proc_noise` parameter to the maximum expected velocity.

Receiver velocity can also be estimated by setting  `default_filter_parameters : stations : pos_rate : estimate` to `true`.

Tropospheric delays are estimated as a combination of hydrostatic and wet components, each component is in turn estimated as the products of the zenith delay and a mapping function.
If `default_filter_parameters : stations : trop : estimate` is set to `true`, the PEA estimates the zenith wet delay.
If `default_filter_parameters : stations : trop_grad : estimate` is set to `true`, the PEA also estimates azimuthal components of tropospheric mapping functions.
 
The hydrostatic zenith delays and elevation dependent component of mapping functions are calculated based on pre-defined models.
Available models, which can be selected using the `processing_options : troposphere : model` parameter, are the GPT2 and VMF3 models.

If using the GPT2 model the path to the necessary grid file needs to be specified in  `processing_options : troposphere : gpt2grid`

If using the VMF3 model, the tropospheric parameters corresponding to the observation times need to be provided in a directory specified by `processing_options : troposphere : vmf3dir`, and the orography file for atmospheric circulation models need to be specified in `processing_options : troposphere : orography`

## Single frequency PPP

It is possible to perform end user PPP processing using single frequency data (although at reduced accuracy) by providing external Ionospheric delay data.

The configuration files named `examples/ex13_pea_pp_user_gps_sf.yaml` set an example to process single frequency observations.

The PEA currently uses IONEX formatted VTEC maps as Ionosphere delay data. 
The path to the IONEX file needs to be specified  in  `input_files : ionfiles`

In order for the PEA to use the VTEC maps, the `processing_options : ionosphere : corr_mode` parameter should to be set to `total_electron_content`

If provided separately, files containing the satellite DCB (either RINEX DCB or bias SINEX) should be specified in `input_files :dcbfiles` 

## Dual frequency PPP with ambiguity resolution

The PEA (in both network and user processing modes) can be specified to perform ambiguity resolution in an attempt to improve accuracy and convergence times.

In aside from the requirements for floating PPP ambiguities, information on satellite hardware biases needs to be provided order to allow correct ambiguity resolution in end user PEA processing.

For post-process, the PEA use bias SINEX formatted files as input channels for satellite hardware biases.
The bias SINEX file can be specified in `input_files :bsxfiles`.

The ambiguity resolution process is controlled by the `ambiguity_resolution_options` field.
Currently ambiguity resolution is only supported for GPS and Galileo satellites. 

Ambiguity resolution for GPS satellites can be activated by setting the `GPS_amb_resol` parameter to `true`

Ambiguity resolution for Galileo satellites can be activated by setting the `GAL_amb_resol` parameter to `true`

In addition the ambiguity resolution algorithm needs to be specified for both the wide-lane ambiguity (`WL_mode`) and narrow-lane ambiguities (`NL_mode`)

For best results, `round` or `iter_rnd` are recommended for Wide-lane ambiguities and  `lambda_alt` or `lambda_bie` is recommended for narrow-lane ambiguities.

## Real-time PPP

The PEA can also be used to process GNSS data in real-time. 
Real-time processing will make use of RTCM formatted streams for receiver observables and satellite data.

Currently the PEA can only get real-time data by connecting to an NTRIP caster.

An example of such a caster, can be accessed by registering at https://www.auscors.ga.gov.au/
The host name, user name and password corresponding to the NTRIP can shold be specified under `station_data : stream_root` using the format `http(s)://user:password@hostname/`
The mountpoint corresponding to station observables need to be listed under `station_data : obs_streams`

Ephemeris streams (broadcast ephemeris and SSR corrections) shold be listed under `station_data : nav_streams`

The PEA support MSM4, MSM5, MSM6 and MSM7 messages for observations, and orbit and clock messages, code bias messages and phase bias messages for GPS and Galileo.

Real-time outputs are not yet defined for PEA, the processed receiver solution are printed in real time on the TRACE files.