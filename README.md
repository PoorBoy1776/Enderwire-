# Enderwire Manta E3 ez board
printer.cfg
[include mainsail.cfg]

[include KAMP/*cfg]

[exclude_object]

## *** THINGS TO CHANGE/CHECK: ***
## MCU paths                         	[mcu] section
## Thermistor types                      [extruder] and [heater_bed] sections - See 'sensor types' list at end of file
## PID tune                              [extruder] and [heater_bed] sections
## Fine tune E steps                     [extruder] section

## For wiring directions please see https://docs.vorondesign.com/build/electrical/sw_miniE3_v20_wiring.html

## Webclient config files. Uncomment one depending on UI being used.
#[include mainsail.cfg]
#[include fluidd.cfg] 

[printer]
kinematics: corexz
max_velocity: 200
max_accel: 1500
max_z_velocity: 50
max_z_accel: 1500
square_corner_velocity: 5.0

[mcu]
###Change to device found by "ls -l /dev/serial/by-id/" with just one this MCU connected to Pi
serial: /dev/serial/by-id/

# [board_pins]
# aliases:
#     # EXP1 header
#     EXP1_1=PC1, EXP1_3=PC3, EXP1_5=PC0, EXP1_7=PA2, EXP1_9=<GND>,
#     EXP1_2=PC2,  EXP1_4=<RST>, EXP1_6=PA0, EXP1_8=PA1, EXP1_10=<5V>

[static_digital_output usb_pullup_enable]
pins: !PC13

#####################################################################
# 	X Stepper Settings
#####################################################################

######
# Motor -XM
# Endstop - X-STOP
###############
[stepper_x]
step_pin: PA14
dir_pin: PA10
enable_pin: !PA13
microsteps: 16
rotation_distance: 40
full_steps_per_rotation: 200
endstop_pin: ^PC4
position_endstop: 220
position_min: 0
position_max: 220
homing_speed: 50
homing_positive_dir: true

[tmc2209 stepper_x]
uart_pin: PB8
##diag_pin: PC4
run_current: 0.650
stealthchop_threshold: 999999

#####################################################################
#   Y Stepper Settings
#####################################################################

######
# Motor -YM
# Endstop - Y-STOP
###############
[stepper_y]
step_pin: PC8
dir_pin: PA15
enable_pin: !PC14
microsteps: 16
rotation_distance: 40
full_steps_per_rotation: 200
endstop_pin: ^PB0
position_endstop: 0
position_min: 0
position_max: 220
homing_speed: 50
homing_positive_dir: false

[tmc2209 stepper_y]
uart_pin: PC9
##diag_pin: PB0
run_current: 0.650
stealthchop_threshold: 999999
## Uncomment if using sensorless Y homing.
#driver_SGTHRS: 120 # tune this once it's working.

#####################################################################
# 	Z Stepper Settings
#####################################################################

######
# Motor -ZAM
# Endstop - Z-STOP
###############

[stepper_z]
step_pin: PD2
dir_pin: !PD4
enable_pin: !PD3
microsteps: 16
rotation_distance: 40
full_steps_per_rotation: 200
endstop_pin: probe:z_virtual_endstop
#position_endstop: 0.0
position_max: 220
homing_speed: 35
position_min: -2.0

[tmc2209 stepper_z]
uart_pin: PD0
##diag_pin: PC6
run_current: 0.650
stealthchop_threshold: 999999

#####################################################################
#   Extruder Settings
#####################################################################

######
#Motor - EM
###############
[extruder]
step_pin: PD5
dir_pin: !PD6
enable_pin: !PB3
# Tune per individual printer
# Default for Bondtech 5mm Bore Drive Gears
rotation_distance: 13.262
# Tune for extruder 
gear_ratio: 50:17
microsteps: 16
full_steps_per_rotation: 200
nozzle_diameter: 0.400
filament_diameter: 1.75
heater_pin: PB11
##  Validate the following thermistor type to make sure it is correct
##  See https://www.klipper3d.org/Config_Reference.html#common-thermistors for additional options
sensor_type: NTC 100K MGB18-104F39050L32
sensor_pin: PA4
min_temp: 10
max_temp: 270
max_power: 0.650
min_extrude_temp: 170
max_extrude_cross_section: 5
#control = pid
#pid_kp = 26.213
#pid_ki = 1.304
#pid_kd = 131.721
#Set appropriate once tuning your printer
pressure_advance: .05
##	Default is 0.040, leave stock
pressure_advance_smooth_time: 0.040
max_extrude_only_distance: 50.0

[tmc2209 extruder]
uart_pin: PD1
run_current: 0.600
stealthchop_threshold: 999999

#####################################################################
# 	Bed Heater
#####################################################################

######
# BED Connector
###############
[heater_bed]
heater_pin: PB2
##  Validate the following thermistor type to make sure it is correct
##  See https://www.klipper3d.org/Config_Reference.html#common-thermistors for additional options
sensor_type: NTC 100K MGB18-104F39050L32
sensor_pin: PA3
min_temp: 0
max_temp: 130
#control: pid
#pid_kp: 58.437
#pid_ki: 2.347
#pid_kd: 363.769

#####################################################################
# 	Probe
#####################################################################

######
#Z Max Connector on Z(main) Board
#Inductive Probe
###############
[probe]
##      If your probe is NO instead of NC, add change pin to !z:P1.24
pin: ^PA6 
x_offset: 0
y_offset: 20
#z_offset: 0
samples: 3
samples_result: median
sample_retract_dist: 3
samples_tolerance: 0.05
samples_tolerance_retries: 3

activate_gcode:
    {% set PROBE_TEMP = 150 %}
    {% set MAX_TEMP = PROBE_TEMP + 5 %}
    {% set ACTUAL_TEMP = printer.extruder.temperature %}
    {% set TARGET_TEMP = printer.extruder.target %}

    {% if TARGET_TEMP > PROBE_TEMP %}
        { action_respond_info('Extruder temperature target of %.1fC is too high, lowering to %.1fC' % (TARGET_TEMP, PROBE_TEMP)) }
        M109 S{ PROBE_TEMP }
    {% else %}
        # Temperature target is already low enough, but nozzle may still be too hot.
        {% if ACTUAL_TEMP > MAX_TEMP %}
            { action_respond_info('Extruder temperature %.1fC is still too high, waiting until below %.1fC' % (ACTUAL_TEMP, MAX_TEMP)) }
            TEMPERATURE_WAIT SENSOR=extruder MAXIMUM={ MAX_TEMP }
        {% endif %}
    {% endif %}

#####################################################################
# 	Fan Control
#####################################################################

######
# Electronics Fan
# FAN1 Connector
###############
[controller_fan my_controller_fan]
pin: PB14 
max_power: .50
kick_start_time: 0.200
heater: heater_bed

######
# Hot End Fan
# FAN2 Connector
###############
[heater_fan extruder_fan]
pin: PA8
heater: extruder
heater_temp: 50.0
##  If you are experiencing back flow, you can reduce fan_speed
#fan_speed: 1.0

######
# Part Cooling Fan
# FAN0 Connector
###############
[fan]
pin: PB15  # "FAN0"
cycle_time: .08
##	Depending on your fan, you may need to increase this value
##	if your fan will not start. Can change cycle_time (increase)
##	if your fan is not able to slow down effectively
kick_start_time: .25

#####################################################################
#   Homing and Bed Mesh
#####################################################################
[homing_override]
axes: z
set_position_z: 0
gcode:
    G90
    G0 Z5 F500
    G28 X0 Y0
    G0 X110 Y110 F9000
    G28 Z0
    G0 Z5 F500

[bed_mesh]
speed: 150
horizontal_move_z: 5
mesh_min: 25,35.0
mesh_max: 210.0,210
probe_count: 5,5
algorithm: bicubic
fade_start: 1
fade_end: 10
fade_target: 0

#[safe_z_home]
#home_xy_position: 117.5, 117.5
#speed: 50
#z_hop: 15
#z_hop_speed: 5
#####################################################################
# 	Displays
#####################################################################
## 	For the mini12864 Display, the [display] and [neopixel] must be uncommented
# mini12864 LCD Display
# connected to exp1/2
# 
# [display]
# lcd_type: st7920
# cs_pin: EXP1_7
# sclk_pin: EXP1_6
# sid_pin: EXP1_8
# encoder_pins: ^EXP1_5, ^EXP1_3
# click_pin: ^!EXP1_2

# [display]
# #    mini12864 LCD Display
# lcd_type: uc1701
# cs_pin: EXP1_7
# a0_pin: PB8
# rst_pin: PB15
# encoder_pins: ^EXP1_5, ^EXP1_3
# click_pin: ^!EXP1_2
# contrast: 63

# spi_software_sclk_pin: PC10
# spi_software_mosi_pin: PC12
# spi_software_miso_pin: PC11

# [neopixel btt_mini12864]
# # # ##  To control Neopixel RGB in mini12864 display
# pin: EXP1_1
# chain_count: 3
# initial_RED: 0.1
# initial_GREEN: 0.5
# initial_BLUE: 0.0
# color_order: RGB

# # ##  Set RGB values on boot up for each Neopixel. 
# # ##  Index 1 = display, Index 2 and 3 = Knob
# [delayed_gcode setdisplayneopixel]
# initial_duration: 1
# gcode:
#          SET_LED LED=btt_mini12864 RED=0 GREEN=1 BLUE=0 INDEX=1 TRANSMIT=0
#          SET_LED LED=btt_mini12864 RED=0 GREEN=1 BLUE=0 INDEX=2 TRANSMIT=0
#          SET_LED LED=btt_mini12864 RED=0 GREEN=1 BLUE=0 INDEX=3

#####################################################################
#   Case Lights
#####################################################################
#[output_pin LIGHTS]
#pin: PA8
#value: 0
#shutdown_value: 0

#[gcode_macro lights_on]
#gcode:
    #SET_PIN PIN=LIGHTS VALUE=1.0

#[gcode_macro lights_off]
#gcode:
    #SET_PIN PIN=LIGHTS VALUE=0.0

#####################################################################
#  input shaper  definition
#####################################################################
[input_shaper]
shaper_freq_x: 41.7  # frequency for the X mark of the test model
shaper_freq_y: 58.0  # frequency for the Y mark of the test model
shaper_type_x: mzv
shaper_type_y: mzv


#####################################################################
# 	Macros
#####################################################################

[gcode_macro PRINT_START]
#   Use PRINT_START for the slicer starting script - PLEASE CUSTOMISE THE SCRIPT
gcode:
    {% set bedtemp = params.BED|int %}
    {% set extrudertemp = params.EXTRUDER|int %}
    {% set chambertemp = params.CHAMBER|default(0)|int %}

    SETUP_KAMP_MESHING DISPLAY_PARAMETERS=1 LED_ENABLE=1 FUZZ_ENABLE=1
    SETUP_LINE_PURGE DISPLAY_PARAMETERS=1 ADAPTIVE_ENABLE=1
     
    M117 Homing...                 ; display message
    M104 S150
    M190 S{bedtemp}
    BED_MESH_CLEAR
    G28 Y0 X0 Z0
    BED_MESH_CALIBRATE
    
    ##Purge Line Gcode
    #G92 E0;
    #G90
    #G0 X5 Y5 F6000
    #G0 Z0.4
    #G91
    #G1 X120 E30 F1200;
    #G1 Y1
    #G1 X-120 E30 F1200;
    #G92 E0;
    #G90
    
    G1 Z15.0 F600 ;move the platform down 15mm
    G1 X110 Y110 F3000
    M109 S{extrudertemp} 
    LINE_PURGE
    M117 Printing...


[gcode_macro PRINT_END]
#   Use PRINT_END for the slicer ending script
gcode:
    #   Get Boundaries
    {% set max_x = printer.configfile.config["stepper_x"]["position_max"]|float %}
    {% set max_y = printer.configfile.config["stepper_y"]["position_max"]|float %}
    {% set max_z = printer.configfile.config["stepper_z"]["position_max"]|float %}
    
    #   Check end position to determine safe directions to move
    {% if printer.toolhead.position.x < (max_x - 20) %}
        {% set x_safe = 20.0 %}
    {% else %}
        {% set x_safe = -20.0 %}
    {% endif %}

    {% if printer.toolhead.position.y < (max_y - 20) %}
        {% set y_safe = 20.0 %}
    {% else %}
        {% set y_safe = -20.0 %}
    {% endif %}

    {% if printer.toolhead.position.z < (max_z - 2) %}
        {% set z_safe = 2.0 %}
    {% else %}
        {% set z_safe = max_z - printer.toolhead.position.z %}
    {% endif %}
    
    #  Commence PRINT_END
    M400                             ; wait for buffer to clear
    G92 E0                           ; zero the extruder
    G1 E-4.0 F3600                   ; retract
    G91                              ; relative positioning
    G0 Z{z_safe} F3600               ; move nozzle up
    G0 X{x_safe} Y{y_safe} F20000    ; move nozzle to remove stringing
    
    M104 S0                          ; turn off hotend
    M140 S0                          ; turn off bed
    M106 S0                          ; turn off fan
    G90                              ; absolute positioning
    G0 X{max_x / 2} Y{max_y} F3600   ; park nozzle at rear
    M117 Finished...
