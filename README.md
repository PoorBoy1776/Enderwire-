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
