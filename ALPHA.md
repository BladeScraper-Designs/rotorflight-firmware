
# Alpha Development version of Rotorflight-2.0 (4.3.x)

This is a snapshot of the ongoing Rotorflight 4.3 development.
It is called _ALPHA_ because lots of things are broken.

# Configuration

## Hardware Config

Should be the same as in RF1.

All hardware related CLI commands (dump hardware) should work.


## Features

Currently there are no changes to feature flags.


## Servos

Servos have now geometry correction. This is for correcting the rotation
geometry. Not needed with linear servos.

The servo CLI command now has an extra argument for flags.
Geometry correction FLAG = 1 (bit0).

NOTE!!!

This will only work if the servo rate is set correctly, i.e. the servo is
actually rotating the way the FC is expecting it to move.

ALSO!!!

This will only work if servo _center_ and _trim_ are used correctly.

SERVO TRIM IS *NOT* FOR CENTERING THE SERVO!

USE SERVO CENTER for CENTERING THE SERVO!


## Modes / AUX

The CLI commands are the same, but the range (PWM) resolution
has been increased from 25 to 5us.


## Adjustment Functions

Adjustment functions have been totally rewritten.

There are now two new ranges in each adjustment, for setting the Rx ranges where
the increment/decrement step is triggered, respectively.

With the continuous adjustment, the first range sets the input range of the control channel.

In RF1, the decrement range was 1000..1300, and the increment range was 1700..2000.
The continuous input range was 1000..2000.

The CLI command format is:

```
adjfunc <index> <func> <enable channel> <start> <end>
        <value channel> <dec start> <dec end> <inc start> <inc end>
        <step size> <value min> <value max>
```

This allows one channel to inc/dec at least 8 different adjustment functions.


## Gyro Filterings

Most gyro filtering related CLI commands have been renamed in BF4.3.
There are further changes in Rotorflight.

CLI commands listed below.

```
set gyro_decimation_hz = 250
set gyro_lpf1_type = BIQUAD
set gyro_lpf1_static_hz = 150
set gyro_lpf1_dyn_min_hz = 25
set gyro_lpf1_dyn_max_hz = 200
set gyro_lpf2_type = PT1
set gyro_lpf2_static_hz = 0
set gyro_notch1_hz = 0
set gyro_notch1_cutoff = 0
set gyro_notch2_hz = 0
set gyro_notch2_cutoff = 0
set dterm_lpf1_type = PT1
set dterm_lpf1_static_hz = 15
set dterm_lpf1_dyn_min_hz = 0
set dterm_lpf1_dyn_max_hz = 0
set dterm_lpf2_type = PT1
set dterm_lpf2_static_hz = 0
set dterm_notch_hz = 0
set dterm_notch_cutoff = 0
```

## Blackbox

Blackbox has been refactored in BF4.3, and further in RF2.

#### `blackbox_mode = <OFF|BOX|ARM|ARMBOX>`

Refactored.

```
OFF     = No logging
NORMAL  = logging when both ARMED and BLACKBOX switch active
ARMED   = logging when ARMED
SWITCH  = logging when BLACKBOX switch active
```

In NORMAL mode, logging is paused if BLACKBOX switch is inactive.

#### `blackbox_sample_rate = <1/N>`

Blackbox sampling is now locked to the PID loop, and can be reduced
by a factor of 2,4,8,16,32,etc.


#### `blackbox_log_XYZ = <ON|OFF>`

Each blackbox field can be now separately enabled/disabled.

```
set blackbox_log_command = ON
set blackbox_log_setpoint = ON
set blackbox_log_mixer = ON
set blackbox_log_pid = ON
set blackbox_log_gyro_raw = ON
set blackbox_log_gyro = ON
set blackbox_log_acc = ON
set blackbox_log_mag = OFF
set blackbox_log_baro = ON
set blackbox_log_battery = ON
set blackbox_log_rssi = ON
set blackbox_log_rpm = ON
set blackbox_log_motors = ON
set blackbox_log_servos = ON
set blackbox_log_gps = OFF
```

## Debug

Debug has now eight 32bit values, instead of four 16bit values.

This will require changes in the Blackbox Explorer eventually.

Some debug modes are logging only one axis, which is selected
with the `debug_axis` parameter.

This parameter replaces the following BF parameters:

```
gyro_filter_debug_axis
acro_trainer_debug_axis
rc_smoothing_debug_axis
```


## Sensors

New settings for sensor update speed:

```
set vbat_update_hz = 100
set ibat_update_hz = 100
set esc_sensor_update_hz = 100
```


## Profile (PID Profile)

PID control has been totally rewritten.

There are now multiple PID controllers to choose from.

#### `pid_mode = 0`

This is a test mode with the FF term only. Useful for debugging the firmware.

#### `pid_mode = 1`

Rotorflight-1.0 compatibility mode. This is exactly the same as RF1.
Parameters have different names, unfortunately.

#### `pid_mode = 2`

New PID mode under development. This is the current idea what RF2 could be.

#### `pid_mode = 9`

Test PID mode with a ridiculous number of parameters. Used for testing new ideas.

This mode has separate PID gains for yaw CW & CCW rotation.

The "other" direction gains are called "way" gains (as in yaw moving backwards...)

```
set way_p_gain = 50
set way_i_gain = 60
set way_d_gain = 10
set way_f_gain = 0
```

#### `pid_dterm_mode = <0|1>`

Selects between D-term on gyro signal (0) vs. on error signal (1).

Available in modes 2 and 9.


### PID Gains

The PID gain parameters have been renamed. Should be obvious.

#### `yaw_[c]cw_stop_gain = <N>`

Stop gain for yaw. Works differently in each mode.

Mode 1: Same as RF1.0. P and D gains multiplied by stop gain.

Mode 2: Only P gain multiplied by stop gain.

Mode 9: Not used


### PID Filters

There are a few extra filters in the PID controller (for evaluation).

#### `pitch_d_cutoff = <Hz>`

D-term bandwidth limit, separate for each axis.

Available in modes 2 and 9. Equivalent to RF1 D-term PT1 filter.
Must be set to a reasonable value for D-term to work.
Typical values 15..30.

#### `pitch_f_cutoff = <Hz>`

FF-term highpass filter cutoff. 0 = disabled.

A new idea about how to do FF. Needs evaluation.

Available in modes 2 and 9.

#### `pitch_error_cutoff = <Hz>`

Error term bandwidth limit. 0 = disabled.

Available in modes 1, 2 and 9.

#### `pitch_gyro_cutoff = <Hz>`

Gyro signal bandwidth limit, separate for each axis. 0 = disabled.
This is an extra PT1 filter for each gyro axis.

Available in modes 2 and 9.

#### `iterm_relax_type = <OFF|RP|RPY>`

Selects the I-term relax type, i.e. which axis to apply to.

#### `iterm_relax_level = <N>`

Sets the setpoint limit/level for I-term relax. This is fixed to 40 in BF.

#### `iterm_relax_cutoff = <R,P,Y>`

I-term relax filter cutoffs for roll, pitch, and yaw.


## Rates

The rates system has been rewritten for RF2.0.

There is now a curve also for collective. It is mostly useful for setting
the collective min/max, but also expo.

In addition to the rate curves, it is now possible to set
_setpoint limits_, _setpoint acceleration_, and _setpoint smoothness_.

#### `yaw_rate_accel = <N>`

Set waw setpoint max acceleration.

#### `yaw_rate_limit = <N>`

Set absolute maximum for yaw setpoint. The curve will usually control what to value is at the stick extreme, so this is an extra limit.

#### `rates_smoothness = <N>`

Set overall smoothness for setpoint changes. This is equivalent to "Style" or "Smoothness" in some FBLs.
Higher is smoother. Typical values 25-200.


## ESC Telemetry

#### `esc_sensor_protocol = <NONE|KISS|HOBBYWINGV4>`

#### `esc_sensor_hw4_current_offset = <N>`

An offset value for calibrating the zero current.

#### `esc_sensor_hw4_voltage_gain = <N>`

Voltage gain - different for each HW model.

```
 *
 * Voltage Gain:
 *   3-6S  (LV):    gain = 110
 *   3-8S  (LVv2):  gain = 154
 *   5-12s (HV):    gain = 210
 *
```

#### `esc_sensor_hw4_current_gain = <N>`

Current gain - different for each HW model.

```
 *
 * Current Gain:
 *   60A:           gain = 60
 *   80A:           gain = 78
 *   100A:          gain = 90
 *   120A:          gain = 100
 *   130A:          gain = 113
 *   150A:          gain = 129
 *   160A:          gain = 137
 *   200A:          gain = 169
 *
```

## Rescue

#### `rescue_mode = <OFF|CLIMB>`

At the moment, CLIMB is the only rescue mode available.

#### `rescue_flip = <OFF|ON>`

Enable flipping the heli upright, if rescue started inverted.

#### `rescue_flip_gain = <N>`

Gain for flipping the heli upright.

#### `rescue_level_fgain = <N>`

Gain for leveling the heli.

#### `rescue_pull_up_time = <N>`

Time for the initial pull-up. (1/10s steps)

#### `rescue_climb_time = <N>`

Time for climbing. (1/10s steps)

#### `rescue_flip_time = <N>`

Time(out) for flipping. (1/10s steps)
If flip did not complete withint this time, climb state is entered regardless.

#### `rescue_exit_time = <N>`

Time for slowly exiting rescue. (1/10s steps)

#### `rescue_pull_up_collective = <N>`

Collective value for pull-up. (1/1000 steps)

#### `rescue_climb_collective = <N>`

Collective value for climb.

#### `rescue_hover_collective = <N>`

Collective value for hover.

#### `rescue_max_rate = <N>`

Maximum cyclic rate for any rescue action.

#### `rescue_max_accel = <N>`

Maximum cyclic rate acceleration for any rescue action.
