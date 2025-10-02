# Using 1kg load cell with hx711 24 bits ADC, channel A+A-

# Load cell Probe
Current config is for BTT-EBB42 v1.2 using I2C port of EBB42 board.

dout_pin: EBB42:PB3

sclk_pin: EBB42:PB4
<img width="1153" height="948" alt="image" src="https://github.com/user-attachments/assets/ae3c9dec-a9da-458b-92ee-21412048698e" />
        
        [load_cell_probe]
        sensor_type: hx711
        dout_pin: EBB42:PB3
        sclk_pin: EBB42:PB4
        gain: A-128
        sample_rate: 80
        counts_per_gram = 8945.79000
        reference_tare_counts = -481936
        #reference_tare_counts:
        #sensor_orientation:
        #   These parameters must be configured before the probe will operate.
        #   See the [load_cell] section for further details.
        force_safety_limit: 700
        #   The safe limit for probing force relative to the reference_tare_counts on
        #   the load_cell. The default is +/-2Kg.
        trigger_force: 75
        #   The force that the probe will trigger at. 75g is the default.
        #drift_filter_cutoff_frequency: 0.8
        #   Enable optional continuous taring while homing & probing to reject drift.
        #   The value is a frequency, in Hz, below which drift will be ignored. This
        #   option requires the SciPy library. Default: None
        #drift_filter_delay: 2
        #   The delay, or 'order', of the drift filter. This controls the number of
        #   samples required to make a trigger detection. Can be 1 or 2, the default
        #   is 2.
        #buzz_filter_cutoff_frequency: 100.0
        #   The value is a frequency, in Hz, above which high frequency noise in the
        #   load cell will be igfiltered outnored. This option requires the SciPy
        #   library. Default: None
        #buzz_filter_delay: 2
        #   The delay, or 'order', of the buzz filter. This controls the number of
        #   samples required to make a trigger detection. Can be 1 or 2, the default
        #   is 2.
        #notch_filter_frequencies: 50
        #   1 or 2 frequencies, in Hz, to filter out of the load cell data. This is
        #   intended to reject power line noise. This option requires the SciPy
        #   library.  Default: None
        #notch_filter_quality: 2.0
        #   Controls how narrow the range of frequencies are that the notch filter
        #   removes. Larger numbers produce a narrower filter. Minimum value is 0.5 and
        #   maximum is 3.0. Default: 2.0
        tare_time: 0.08
        #   The rime in seconds used for taring the load_cell before each probe. The
        #   default value is: 4 / 60 = 0.066. This collects samples from 4 cycles of
        #   60Hz mains power to cancel power line noise.
        z_offset: 0
        speed: 50
        samples: 1
        sample_retract_dist: 20
        lift_speed: 10
        #samples_result:
        #samples_tolerance:
        #samples_tolerance_retries:
        #activate_gcode:
        #deactivate_gcode:
        #   See the "[probe]" section for a description of the above parameters.

# Force move and homing override
    [force_move]
    enable_force_move: True

    [homing_override]
    gcode:
        _HOME_Z
        G28 XY
    #   A list of G-Code commands to execute in place of G28 commands
    #   found in the normal g-code input. See docs/Command_Templates.md
    #   for G-Code format. If a G28 is contained in this list of commands
    #   then it will invoke the normal homing procedure for the printer.
    #   The commands listed here must home all axes. This parameter must
    #   be provided.
    #axes: xyz
    #   The axes to override. For example, if this is set to "z" then the
    #   override script will only be run when the z axis is homed (eg, via
    #   a "G28" or "G28 Z0" command). Note, the override script should
    #   still home all axes. The default is "xyz" which causes the
    #   override script to be run in place of all G28 commands.
    #set_position_x:
    #set_position_y:
    #set_position_z:
    #   If specified, the printer will assume the axis is at the specified
    #   position prior to running the above g-code commands. Setting this
    #   disables homing checks for that axis. This may be useful if the
    #   head must move prior to invoking the normal G28 mechanism for an
    #   axis. The default is to not force a position for an axis.
    
    [gcode_macro _HOME_Z_FROM_LAST_PROBE]
    gcode:
        {% set z_probed = printer.probe.last_z_result %}
        {% set z_position = printer.toolhead.position[2] %}
        {% set z_actual = z_position - z_probed %}
        SET_KINEMATIC_POSITION Z={z_actual}
    
    [gcode_macro _HOME_Z]
    gcode:
        SET_GCODE_OFFSET Z=0  # load cell probes dont need a Z offset
        # position toolhead for homing Z, edit for your printers size
        #G90  # absolute move
        #G1 Y50 X50 F{5 * 60}  # move to X/Y position for homing
    
        # soft home the z axis to its limit so it can be moved:
        SET_KINEMATIC_POSITION Z={printer.toolhead.axis_maximum[2]}
    
        # Fast approach and tap
        PROBE PROBE_SPEED={5 * 60}  # override the speed for faster homing
        _HOME_Z_FROM_LAST_PROBE
    
        # lift z to 2mm
        G91  # relative move
        G1 Z2 F{5 * 60}
    
        # probe at standard speed
        PROBE
        _HOME_Z_FROM_LAST_PROBE
    
        # lift z to 10mm for clearance
        G91  # relative move
    G1 Z10 F{5 * 60}
    
