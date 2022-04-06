# PEA Configuration File - YAML
	
The PEA processing engine uses a single YAML file for configuration of all processing options. A subset of the most important options available will be described here.

## Default Values and parameter descriptions

Many processing options have default values associated with them. To prevent repetition, and to ensure that the values are reported correctly, these values may be printed to the screen by the `pea` application.

The default value and description of almost every config parameter can be printed by using:

    ./pea -Y

The parameters will be output while retaining the required structure within the yaml format.

## YAML Syntax
The YAML format allows for heirarchical, self descriptive configurations of parameters, and has a straightforward syntax.

White-space (indentation) is used to specify heirarchies, with each level typically indented with 4 space characters.

Colons (:) are used to separate configuration keys from their values.

Lists may be created by either appending multiple values on a single line, wrapped in square brackets and separated by commas, or, by adding each value on a separate indented line with a dash before the value.

Adding a hash symbol (\#) to a line will render the remainder of the line as a comment to be ignored by the parser.

Strings with special characters or spaces should be wrapped in quotation marks.

You will see all of these used in the example configuration files, but the files may be re-ordered, or re-formatted to suit your application.


## Globbing
Files may be specified individually, as lists, or by searching available files using a globbed filename using the star character (`*`)

## Wildcard Tags
Output filenames can include wildcards wrapped in `< >` brackets to allow more generic names to be used. While processing, these tags are replaced with details gathered from processing, and allows for automatic generation of, for example, hourly output files.

#### `<CONFIG>`
This is replaced with the 'config_description' value entered in the yaml file.
#### `<STATION>`
This is replaced with the 4 character station id of each station that generates a trace file.
#### `<LOGTIME>`
This is replaced with the (rounded) time of the epochs within the trace file.

If trace file rotation is configured for 1 hour, the `<LOGTIME>` wildcard will be rounded down to the closest hour, and subsequently change value once per hour and generate a separate output file for each hour of processing.

#### `<DDD>`,`<D>`, `<WWWW>`, `<YYYY>`, `<YY>`, `<MM>`, `<DD>`, `<HH>`

These are replaced with the components of time of the start epoch.

## input_files:

This section of the yaml file specifies the lists of files to be used for general metadata inputs, and inputs of external product data.

	# Example
	input_files:

		root_input_directory: /data/acs/pea/proc/exs/products/
		
		atxfiles:   [ igs14_2045_plus.atx                                ]
		snxfiles:   [ igs19P2062.snx                                     ]
		blqfiles:   [ OLOAD_GO.BLQ                                       ]  
		navfiles:   [ brdm1990.19p                                       ]  
		orbfiles:   [ orb_partials/gag20624_orbits_partials_new.out      ]  
		sp3files:   [ "*.sp3"                                            ]
		clkfiles:   [ jpl20624.clk                                       ]  
		erpfiles:   [ igs19P2062.erp                                     ]  
		dcbfiles:   [                                                    ] 
		bsxfiles:   [ CAS0MGXRAP_20191990000_01D_01D_DCB.BSX             ]
		ionfiles:   [                                                    ] 
<!-- 
#### root_input_directory:
This specifies a root directory to be prepended to all other file paths specified in this section. For file paths that are absolute, (ie. starting with a /), this parameter is not applied.

#### atxfiles:
A list of ANTEX files to be used in processing. These may supply the antenna parameters to be used by satellites and receivers.

#### snxfiles:
A list of SINEX files to be used in processing. These may supply the initial positions and other metadata for receivers.

#### blqfiles:
A list of BLQ files to be used in processing. These may supply the ocean tide loading data.

#### navfiles:
A list of NAV files to be used in processing. These may supply the basic broadcast ephemerides for satellites.

#### orbfiles:
A list of ORB files to be used in processing. These may supply the PEA with orbital and ratiation pressure data from the Ginan's POD module, allowing precise orbit data to be passed between the two pieces of software.

#### sp3files:
A list of SP3 files to be used in processing. These may supply the ephemerides for higher precision processing.

#### clkfiles:
A list of CLK files to be used in processing. These may supply the clock offsets for satellites and receivers for higher precision processing.

#### erpfiles:
A list of ERP files to be used in processing. These may supply the earth rotation parameter information.

#### dcbfiles:
A list of DCB files to be used in processing. These may supply the differential code biases to assist with ambiguity resolution.

#### bsxfiles:
A list of BSINEX files to be used in processing. These may supply biases to assist with ambiguity resolution.

#### ionfiles:
A list of ION files to be used in processing. These may supply the ionospheric modelling parameters for single frequency processing.
 -->

## station_data:
This section specifies the sources of observation data to be used in positioning.


There are numerous ways that the `pea` can access GNSS observations to process. 
You can specify individual files to process, set it up so that it will search a particular directory, or you can use a command line flag `--rnx <rnxfilename>` to add an additional file to process. 
The data should be uncompressed rinex (gunzipped, and not in hatanaka format), or RTCM3 formatted binary data.


It may consist of RINEX files, or RTCM streams or files, which are specified as follows:

### Post processing:

	# post processing example
	station_data:
		root_stations_directory: /data/acs/ginan/examples/data
		rnxfiles:
			- "ALIC*.rnx"
			- "BAKO*.rnx"
			
		#obs_rtcmfiles:
		#	- "*-OBS.rtcm3"
			
		#nav_rtcmfiles:
		#	- "*-NAV.rtcm3"


#### root_stations_directory:

This specifies a root directory to be prepended to all other file paths specified in this section. For file paths that are absolute, (ie. starting with a /), this parameter is not applied.

#### rnxfiles:

This is a list of RINEX files to be used for observation data. The first 4 characters of the filename are used as the receiver ID.

If multiple files are supplied with the same ID, they are all processed in sequence - according to the epoch times specified within the files. In this case, it is advisible to correctly specify the start_epoch for the filter, or the first epoch in the first file will likely be used.

#### obs_rtcmfiles:

This is a list of RTCM binary files to be used for observation data. The first 4 characters of the filename are used as the receiver ID.

This can be used to read data that has been saved from a stream for later testing.

#### nav_rtcmfiles:

This is a list of RTCM binary files to be used for navigation and correction data. No receiver is to be associated with these files.

#### Real-time processing:

To process data in real-time you will need to set up the location, username annd password for the caster that you will be obtaining the input data streams from in the configuration file.

The pea supports obtaining streams from casters that use NTRIP 2.0 over http and https.

	# realtime streaming example
	station_data:
		
		stream_root: "http://<username>:<password>@auscors.ga.gov.au:2101/"

		nav_streams:
			- BCEP00BKG0
			- SSRA00CNE0
			
		obs_streams:
			- STR100AUS0
			
		ssr_input_antenna_offset: APC

As shown in listing: \ref{lst:realtimeConfig}, the caster url, username and password are specified within double quotes with the `stream_root` tag. In this example the streams are being obtained from the auscors caster run by Geoscience Australia. 
The broadcast information is being obtained from the stream `BCEP00BKG0` being supplied by BKG, and corrections to the utlra-rapid predicted orbit are being obtained from the stream `SSRA00CNE0`. 
The real-time data being processed is for the continuous GNSS station located at Mount Stromlo obtained from the stream `STR100AUS0`.

You can test your username and password is working correctly by running the curl command:

	curl https://ntrip.data.gnss.ga.gov.au/ALIC00AUS0 -H "Ntrip-Version: NTRIP/2.0" -i  --output - -u <user>

#### stream_root:
This specifies a root url to be prepended to all other streams specified in this section. If the streams used have individually specified root urls, usernames, or passwords, this should not be used.

#### obs_streams:
This is a list of RTCM streams to read realtime data from. The first 4 characters of the filename are used as the receiver ID.

In combination with the stream_root parameter, they may require a username, password, port and mountpoint.

The streams in this section are processed for observations from receivers.

#### nav_streams:
This is a list of RTCM streams to read real-time data from.

In combination with the stream_root parameter, they may require a username, password, port and mountpoint.

The streams in this section are processed separately from observations, and will typically be used for receiving SSR messages or other navigational data from an external service.

#### ssr_input_antenna_offset:
This setting should match the ephemeris type that is provided in the listed SSR stream, i.e. satellite antenna-phase-centre (APC) or centre-of-mass (COM). This information is listed in the NTRIP Caster's sourcetable - in general, use `APC` for `SSRA*` streams, and `COM` for `SSRC*` streams.


## output_files:
This section of the yaml file specifies options to enable outputs and specify file locations.

An example of this section follows:

	output_files:

	root_output_directory:          /data/acs/ginan/examples/<CONFIG>/

	output_trace:                   true
	trace_level:                    3
	trace_directory:                ./
	trace_filename:                 <CONFIG>-<STATION><LOGTIME>.TRACE

	output_residuals:               false

	output_config:                  true

	output_summary:                 false
	summary_directory:              ./
	summary_filename:               <CONFIG>-<YYYY><DDD><HH>.SUM

	output_clocks:                  true
	clocks_directory:               ./
	clocks_filename:                <CONFIG>.clk


#### root_output_dir:
This specifies a root directory to be prepended to all other file paths specified in this section. For file paths that are absolute, (ie. starting with a /), this parameter is not applied.

#### X_directory:
Directory to output file [X] to, where [X] are the features below. May contain wildcard tags. May be relative to root_output_dir, or absolute. If the directory does not exist, it will be created.
<!-- 
* trace_directory
* summary_directory
* clocks_directory
* ionex_directory
* biasSinex_directory
* sinex_directory
* persistance_directory
* rtcm_directory
 -->
#### X_filename:
Filename to use for output of [X]. May contain wildcard tags. File will be created or overwritten if it already exists.
<!-- 
* trace_filename
* summary_filename
* clocks_filename
* ionex_filename
* biasSinex_filename
* sinex_filename
* persistance_filename
* obs_rtcm_filename
* nav_rtcm_filename
#### trace_level:
Integer from 0-5 to specify verbosity of trace outputs. (5 - print everything)

 -->
#### trace_rotate_period, trace_rotate_period_units:
Granularity of length of time used for `<LOGTIME>` tags. These parameters may be used such that the filename of an output will change intermittently, and thus distribute the output over multiple files.

The `<LOGTIME>` tag is updated according to the epoch time, not the current clock time.

trace_rotate_period must be a numeric value, and trace_rotate_period_units may be one of seconds (default), minutes, hours, days, weeks, years, (with or without plural s).
<!-- 
#### output_residuals:
Boolean to print the residuals from kalman filter operation to relevant trace files.

#### output_config:
Boolean to print a copy of the yaml file to the top of each trace file. This may assist with keeping a record of the parameters used to generate the particular results contained in the file.

#### output_trace:
Boolean to generate per-station trace files.

#### output_summary:
Boolean to generate a network summary file.

#### output_clocks:
Boolean to generate RINEX formatted clock files from processed data.

#### output_AR_clocks:
Boolean to specify that the ambiguity resolved version of clocks should be output if output_clocks is enabled.

#### output_ionex:
Boolean to generate an IONEX file from processed ionosphere data.

#### output_ionstec:
Boolean to generate an IONSTEC file from processed ionosphere data.

#### output_biasSINEX:
Boolean to generate a biasSINEX from processed network data.

#### output_sinex:
Boolean to generate a sinex file containing processed solutions, and the metadata used to generate them.

#### output_persistance:
Boolean to save the network filter state, and navigation and ephemerides structure to disk once per epoch. For realtime processing where ephemerides are sourced from a a stream over several minutes, this may enable quicker start-up if the processor is restarted.

#### input_persisance:
Boolean to try to load a saved filter and navigation structure from disk.

#### output_mongo_measurements:
Boolean to output kalman filter measurements and residuals to a mongo database.

#### output_mongo_states:
Boolean to output the results of kalman filter processing to a mongo database.

#### output_mongo_logs:
Boolean to output timestamped log data from the console to a mongo database.

#### output_mongo_metadata:
Boolean to output timestamped metadata from processing to a mongo database. (unimplemented)

#### delete_mongo_history:
Boolean to delete a previous database using the same `<CONFIG>` tag before processing, to prevent collisions.

#### mongo_uri:
The URL to the location of the mongo database server.
 -->

## processing_options:

This sections specifies the extent of processing that is performed by the engine.

#### epoch_interval:
Increment in nominal epoch time for each processing epoch. This parameter may be used to sub-sample datasets by using an epoch_interval that is a multiple of the dataset's internal interval between epochs.

#### start_epoch
Nominal time of the first epoch to process. Time is formatted as YYYY-MM-DD HH:MM:SS.
This parameter may be left undefined to use the first available data point.

#### end_epoch
Maximum nominal time of the last epoch to process. This parameter may be left undefined.

#### max_epochs:
Maximum epochs to process before completion. This parameter may be left undefined.

#### process_modes:

	process_modes:
	    user:                   true
	    network:                false
	    minimum_constraints:    false
	    rts:                    false
	    ionosphere:             false

#### user:
Boolean to process all stations individually. Typically used for determining position of individual receivers.

#### network:
Boolean to process all stations in a single filter. May be used for determination of orbits, clocks, etc.

#### minimum_constraints:
Boolean to apply a rigid transformation to the results of the network filter after completion.

#### ionosphere:
Boolean to compute an ionosphere model from observations.

#### unit_tests:
Boolean to run tests to compare intermediate values during processing to stored results.


#### process_sys:
	process_sys:
	    gps:            true
	    glo:            false
	    gal:            false
	    bds:            false

Booleans to enable the various GNSS satellite systems.
<!-- 
* gps
* glo
* gal
* bds
 -->
 
#### elevation_mask:
Minimum elevation required for observations to be used, measured in degrees.

#### ppp_ephemeris:
Option to specify source of satellite ephemeris used in PPP processing. Sources are:

* broadcast
* precise
* precise_com
* sbas
* ssr_apc
* ssr_com
<!-- 
#### tide_solid:
Boolean to apply solid tide model to station positions.

#### tide_otl:
Boolean to apply ocean tide loading model to station positions.

#### tide_pole:
Boolean to apply pole tide model to station positions.

#### phase_windup
Boolean to apply phase windup model to satellite phase measurements.

#### reject_eclipse
Boolean to exclude eclipsed satellites from processing.

#### raim
Boolean to perform 'Receiver autonomous integrity monitoring' to detect and exclude observations that result in SPP failures.

#### cycle_slip: thres_slip:
Threshold to apply to geometry free phase values to determine if an observation should be rejected due to a slip.

#### max_inno:
Maximum innovation in PPP measurement before both phase and code measurements are excluded.

#### deweight_factor:
Factor by which measurement variances are increased upon detection of a bad measurement.

#### max_gdop:
Maximum 'geometric dilution of precision' allowed for an SPP result to be valid.

#### antexacs:
Internal processing option. Bad things will likely happen if this is set to false.

#### sat_pcv:
Boolean to model satellite phase center variations.

#### pivot_station:
Station specified as origin for receiver clocks. Clocks for this station will be constrained to zero. May be set to <AUTO > or undefined to use first available station.

#### pivot_satellite:
Unused.
 -->
#### wait_next_epoch:
Expected time interval between successive epochs data arriving. For real-time this should be set equal to epoch_interval.

#### wait_all_stations:
Window of delay to allow observation data to be received for processing.
Processing will begin at the earliest of:

* Observations received for all stations
* wait_all_stations has elapsed since any station has received observations
* wait_all_stations has elapsed since wait_next_epoch expired.

#### code_priorities:
List of observation codes that may be used in processing, and the order of priority for use. (Currently only a single code is used per frequency)
<!-- 
#### joseph_stabilisation:
Boolean to apply additional calculations in filter to ensure numerical stability.

## troposphere:

    troposphere:
        model:      vmf3    #gpt2
        vmf3dir:    grid5/
        orography:  orography_ell_5x5
        # gpt2grid: EX03/general/gpt_25.grd

#### vmf3dir:

Location of vmf3 files.

#### orography:

Orography filename for vmf3 troposphere.

#### gpt2grid:

Name of gpt2 grid file. Will be used as fallback in case of errors with vmf3.




## ionosphere:

#### corr_mode:

Ionosphere correction/model mode. May be one of:

* broadcast - broadcast model
* sbas - SBAS model
* iono_free_linear_combo - L1/L2 or L1/L5 iono-free LC
* estimate -  estimation
* total_electron_content - IONEX TEC model
* qzs - QZSS broadcast model
* lex - QZSS LEX ionospehre
* stec - SLANT TEC model

#### iflc_freqs:

Frequency pairs to be used in ionosphere-free linear combinations. May be one of:

* any
* l1l2_only
* l1l5_only



## unit_test_options:

#### output_pass:

Boolean to print pass messages in output file when tests pass. This may produce very long test files for not much benefit if we are just looking for failures.

#### stop_on_fail:

Boolean to halt processing as soon as an error is located. This may allow testing and reporting to complete far sooner if there is a failure.

#### stop_on_done:

Boolean to halt further processing if all required tests have been completed.

#### output_errors:

Boolean to print debug information about the error to the output file.

#### absorb_errors:

Boolean to replace incorrect values found in processing with the correct test values and continue processing as if the test had passed.
This may be useful for preventing a single bad test from causing cascading test failures as the values diverge from the original result.

#### directory, filename:

File and directory to store and open test files.

## ionosphere_filter_parameters:

#### model:

Model to use in ionosphere routines. May be one of:

* meas_out
* bspline
* spherical_caps
* spherical_harmonics

#### model_noise:

Process noise to be applied to ionosphere kalman filter. (deprecated)

#### lat_center, lon_center:

Longitude and latitude of center of ionosphere map in degrees.

#### lat_width, lon_width:

Width of ionosphere maps in degrees.

#### lat_res, lon_res:

Resolution of ionosphere maps in degrees.

#### time_res:

Resolution of ionosphere maps in time.

#### func_order:

Order of Legendre function used in spherical caps ionosphere.

#### layer_heights:

List of heights of modelled ionosphere layers.
 -->

## output_options:

This section specifies values to be used in the generation of output files.

	output_options:

	    config_description:             ex11
	    analysis_agency:                GAA
	    analysis_center:                Geoscience Australia
	    analysis_program:               AUSACS
	    rinex_comment:                  AUSNETWORK1

#### config_description:

The value entered here is used to complete the \verb|<CONFIG>| wildcard.
This may enable a single change in the yaml file to make changes to many options, including output folders and filenames.

#### analysis_agency, analysis_center, analysis_program, rinex_comment:

String to be written within files during various output files' generation.

## user_filter_parameters, network_filter_parameters, ionosphere_filter_parameters:

	user_filter_parameters:

	    max_filter_iterations:      5
	    max_prefit_removals:        3

	    rts_lag:                    -1      #-ve for full reverse, +ve for limited epochs
	    rts_directory:              ./
	    rts_filename:               PPP-<CONFIG>-<STATION>.rts

	    inverter:                   LLT         #LLT LDLT INV

## Configuration

All elements within the Kalman filter are configured using the yaml configuration file, and use a consistent format.

#### default_filter_parameters

	default_filter_parameters:

	    stations:

	        error_model:        elevation_dependent         #uniform elevation_dependent
	        code_sigmas:        [0.15]
	        phase_sigmas:       [0.0015]

	        pos:
	            estimated:          true
	            sigma:              [0.1]
	            proc_noise:         [0] #0.57 mm/sqrt(s), Gipsy default value from slow-moving
	            proc_noise_dt:      second
	            #apriori:                                   # taken from other source, rinex file etc.

	        clk:
	            estimated:          true
	            sigma:              [0]
	            proc_noise:         [10]
	            proc_noise_dt:      second

	        clk_rate:
	            estimated:          false
	            sigma:              [500]
	            proc_noise:         [1e-4]
	            proc_noise_dt:      second
	            clamp_max:          [+500]
	            clamp_min:          [-500]
	            
	    satellites:

	        clk:
	            estimated:          true
	            sigma:              [1000]
	            proc_noise:         [1]
	            #proc_noise_dt:      min
	            #proc_noise_model:   RandomWalk

	        # clk_rate:
	        #     estimated:          true
	        #     sigma:              [10]
	        #     proc_noise:         [1e-5]

	        orb:
	            estimated:          false

	    eop:
	        estimated:  true
	        sigma:      [40]


	override_filter_parameters:

	    stations:
	        #ALIC:
	            pos:
	                sigma:              [0.001]
	                proc_noise:         [0]


The majority of estimated states are configured in this section. These configurations are applied to all estimates unless another configuration overrides these parameters in the override_filter_parameter section.

The parameters that are available for estimation include:

* stations: - per station parameters
	* `pos` - position of station
	* `pos_rate` - rate of change of position of station (velocity)
	* `clk` - clock offset of station
	* `clk_rate` - rate of change of clock offset of station (clock skew)
	* `amb` - carrier phase ambiguity
	* `trop` - vertical troposphere delay at station
	* `trop_grads` - gradients of troposphere in North and East components
* satellites: - per satellite parameters
	* `pos` - position of satellite (coming soon)
	* `pos_rate` - rate of change of position (coming soon) of satellite (velocity)
	* `clk` - clock offset of satellite
	* `clk_rate` - rate of change of clock offset of satellite (clock skew)
	* `orb` - orbital corrections to be used with POD module.
* `eop` - earth orientation parameters (polar motion, delta length of day)


#### estimated:

Boolean to add the state(s) to the Kalman filter for estimation.

#### sigma:

List of a-priori sigma values for each of the components of the state.

If the sigma value is left as zero (or not initialised), then the initial variance and value of the state will be estimated by using a least-squares approach.
In this case, the user must ensure that the solution is likely rank-sufficient, else the least-squares initialisation will fail.

For states with multiple elements (eg, X,Y,Z positions), multiple sigma values may be added to the list. However, if insufficient values are added to the list, the initialiser will use the last value in the list for any extra elements.
ie. Setting `sigma: [10]` is sufficient to set all x,y,z components of the apriori standard deviation to 10.

#### proc_noise:

List of process noises to be added to the state during state transitions. These are typically in m/sqrt(s), but different times may be assigned separately.
As for the sigma list, the last value will be used for any elements exceeding the list length.

#### proc_noise_dt:

Unit of measure for process noise. 
May be left undefined for seconds, or using sqrt_second, sqrt_seconds, sqrt_minutes, sqrt_hours, sqrt_days, sqrt_weeks, sqrt_years.

#### override_filter_parameters:

In the case that a specific station or satellite requires an alternate configuration, or to exclude estimates entirely, the override_filter_parameters section may be used to overwrite selected components of the configuration.


#### user_filter_parameters, network_filter_parameters:

The internal operation of the Kalman filter is specified in this section. It has a large impact on the robustness, and associated processing time that the filter will achieve.

	user_filter_parameters:

	    max_filter_iterations:      5 #5
	    max_prefit_removals:        3 #5

	    rts_lag:                    -1      #-ve for full reverse, +ve for limited epochs
	    rts_directory:              ./
	    rts_filename:               PPP-<CONFIG>-<STATION>.rts

	    inverter:                   LLT         #LLT LDLT INV


#### max_prefit_removals:

Maximum number of pre-fit residuals to reject from the filter.

After the vector of residuals has been generated and before the filter update stage is computed, the residuals are compared with the expected values given the existing states and design matrix.
If the values are deemed to be unreasonable - because the variances of the transformed states and measurements do not overlap to with a 4-sigma level of confidence - then these measurements are deweighted by deweight_factor, to prevent the bad values from contaminating the filter.

These measurements are recorded as being rejected, and may have additional consequences according to other configurations such as phase_reject_limit.

#### max_filter_iterations:

Maximum number of times to compute the full update stage due to rejections.

This is similar to the max_filter_rejections parameter, but the 4-sigma check is performed with post-fit residuals, which are much more precise.

Rejections that occur in this stage require the entire filter inversion to be repeated, and has an associated performance hit when used excessively.


#### inverter:

There are multiple inverters that may be used within the Kalman filter update stage, which may provide different performance outcomes in terms of processing time and accuracy and stability.

The inverter may be selected from:

* llt
* ldlt
* inv

#### outage_reset_limit:
Maximum number of epochs with missed phase measurements before the ambiguity associated with the measurement is reset.

#### phase_reject_limit:
Maximum number of phase measurements to reject before the ambiguity associated with the measurement is reset.

#### rts_X:

For details about rts configuration, see section \ref{ch:RTS}

### Process Noise Guidelines

Currently in the PEA we have random walk process noise models implemented.

The units are typically in meters, and they are given as $\sigma$ = $\sqrt{variance}$

For a random walk process noise, the process noise is incremented at each epoch as $\sigma^2\times dt$ where dt is the time step between filter updates.

If you want to allow kinematic processing, then you can increase the process noise e.g.

proc_noise [0.003]
proc_noise_dt: second

equates to $0.003\frac{1}{\sqrt{s}}$

Or if you wanted highway speeds 100km/hr = 28 m/s

	proc_noise [28]
	proc_noise_dt: second

A nice value for using VMF as an apriori value is 0.1mm /sqrt(s)

	trop:
	    estimated:          true
	    sigma:              [0.1]
	    proc_noise:         [0.01]
	    proc_noise_dt:      hour


## default_filter_parameters:

	default_filter_parameters:

	    stations:

	        error_model:        elevation_dependent         #uniform elevation_dependent
	        code_sigmas:        [0.15]
	        phase_sigmas:       [0.0015]

	        pos:
	            estimated:          true
	            sigma:              [0.1]
	            proc_noise:         [0.00057] #0.57 mm/sqrt(s), Gipsy default value from slow-moving
	            proc_noise_dt:      second


#### error_model:

The GNSS observations can be weighted in three different ways in the PEA:

* uniform - all observations are assigned the same variance
* elevation_dependent - an elevation dependent function is used to scale the observations, those at higher elevation are given more weight (a smaller standard deviation) than those observed at lower elevation
* SNR observations (coming soon) - the Carrier to Noise observations supplied by the receiver are used to determine the observation weight. Generally speaking this is very similar to elevation weighting, but is useful when use observations obtained from a non-geodetic grade receiver/antenna.

#### code_sigmas, phase_sigmas:

Lists of the default sigma values for GNSS measurements, measured in meters.
Separate values may be entered for L1, L2 frequencies if desired, or the last value will be used for any undefined values in the list.

#### pos, clk, amb, trop...:

For details on the configuration of estimated elements refer to \ref{KFConfig}
<!-- 
## minimum_constraints:

For details on the configuration of minimum constraints refer to \ref{MinConConfig}

## ambiguity_resolution_options:

#### Min_elev_for_AR:

Minimum elevation to perform ambiguity resolution (degrees)


#### Set_size_for_lambda:

Candidate set size for lambda.


#### Max_round_iterat:

Maximum number of iterations when performing integer rounding.


#### GPS_amb_resol, GLO_amb_resol, GAL_amb_resol, BDS_amb_resol, QZS_amb_resol:

Booleans to enable the resolution of ambiguities for system X.


#### WL_mode, NL_mode:

Mode of ambiguity resolution for widelanes and narrowlanes.

May be one of the following:

* off
* round
* iter_rnd
* bootst
* lambda
* lambda_alt
* lambda_alt2
* lambda_bie


#### WL_succ_rate_thres, NL_succ_rate_thres:

Threshold for success rate test in LAMBDA. Values between 0 and 1 are valid.


#### WL_sol_ratio_thres, NL_sol_ratio_thres:

Thresholds for ambiguity validation: Ratio is performance of best solution compared to next best performance set. (greater than 1)


#### WL_procs_noise_sat, WL_procs_noise_sta:

Process noise applied for stations or satellites in the ambiguity resolution computations.


#### NL_proc_start:

Time before starting to calculate (and output) NL ambiguities/biases (in seconds)


#### read_OSB, read_DSB, read_SSR, read_satellite_bias, read_station_bias, read_GLONASS_IFB:

Booleans to enable reading of bias types from file.


#### write_OSB, write_DSB, write_SSR_bias, write_satellite_bias, write_station_bias:

Booleans to enable writing of bias types to file.

 -->
## RTS

	user_filter_parameters:

	    max_filter_iterations:      5
	    max_prefit_removals:        3

	    rts_lag:                    -1      #-ve for full reverse, +ve for limited epochs
	    rts_directory:              ./
	    rts_filename:               PPP-<CONFIG>-<STATION>.rts

	    inverter:                   LLT         #LLT LDLT INV


#### rts_lag:

Number of future epochs to use in RTS smoothing. 
A larger lag will give more optimal smoothing results, at the expense of a longer lag before they are calculated, and requiring more processing time per epoch.

A negative value indicates that the entire solution should be smoothed at the conclusion of processing. 
This will obtain optimal results, with lowest processing time, but is not suitable for real-time applications.

#### rts_directory:

Directory to output RTS files.

#### rts_filename:

Filename for RTS files. Multiple intermediate files are generated by RTS smoothing, as well as an additional output files for Kalman filter states and clocks.

During real-time processing with finite RTS applied, the intermediate values stored in the file will be deleted after last use, to prevent excessively large files from being generated.
This results in a rolling history of the filter being stored in the file, unlike other output files used in the software, which are rotated to new files at discrete points in time.