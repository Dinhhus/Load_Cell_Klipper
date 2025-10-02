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
    #counts_per_gram: 73039
    counts_per_gram = 8945.79000
    reference_tare_counts = -481936
    force_safety_limit: 700
    trigger_force: 75
    tare_time: 0.08
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
    
