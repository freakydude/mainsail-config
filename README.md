# Mainsail klipper macros and settings

This is the new location of the macros and settings provided by the Mainsail team. The intent is to enable all users to setup important macros and settings needed by the mainsail UI.

### Important features

- Park position depending on printer kinematics or user preference
  - Round beds 0:Ymax
  - Square beds Xmax:Ymax
  - User Position
  - User can specify an differnt position for PAUSE and CANCEL_PRINT
- PAUSE now supports option input parameters [X,Y,Z_MIN]
  - That is helpful to direct the use of the PAUSE macro in your M600 (see the mainsail.cfg for an example)
- Customization via a single macro that contains all allowed variables
- Additional custom variables for stuff like extra retract at CANCEL_PRINT.
- "Pause at next Layer" and "Pause at Layer #"

### Why have we decided to use a dedicated repo?

There are 2 main reasons for that decision

1) The current moonraker changes have shown that we need to react very fast on changes.
2) We regularly improve the macros and that is the simplest way that the installed base can benefit from it.

### How to install

We currently do not provide an installer and we might never do. But doing it is simple copy and paste a few commands in an ssh terminal.
The instructions assume that you have an up to data and working moonraker installation. If not start with updating your moonraker first.

```sh
cd ~
git clone https://github.com/mainsail-crew/mainsail-config.git
ln -sf ~/mainsail-config/mainsail.cfg ~/printer_data/config/mainsail.cfg
```

### Add updater section

You need to update your moonraker.conf to alway get the latest version.

Either open your moonraker.conf and add

```ini
[update_manager mainsail-config]
type: git_repo
primary_branch: master
path: ~/mainsail-config
origin: https://github.com/mainsail-crew/mainsail-config.git
managed_services: klipper
```

below the mainsail updater section.

You can also link and include the prepared file to your moonraker.conf. ssh in your PI and

```sh
ln -sf ~/mainsail-config/mainsail-moonraker-update.conf ~/printer_data/config/mainsail-moonraker-update.conf
```

then open your moonraker.conf and add

```ini
[include mainsail-moonraker-update.conf]
```

below the mainsail updater section.

### How to setup

The setup process is still the same, simply add

```ini
[include mainsail.cfg]
```

to your printer.cfg file. Be aware the file will show up as read only. This was intended by use.

### How to customize your settings

##### Different virtual sd card location or a different on_error_gcode

for that simply copy the [virtual_sdcard] block from the mainsail.cfg and simple place it below your include. The result should look similar to:

```ini
[include mainsail.cfg]

[virtual_sdcard]
path: "my new location"
on_error_gcode: CANCEL_PRINT
```

##### Customize macros

We provided a variable setup that let you customize the provided PAUSE, RESUME and CANCEL_PRINT macros. If you need e.g., your own park position simple copy the complete _CLIENT_VARIABLE macro from the mainsail.cfg and place it below your mainsail include.
After that uncomment the needed variable and fill them with your values.

The result for a custom park position would look like:

```ini
[include mainsail.cfg]

[gcode_macro _CLIENT_VARIABLE]
variable_use_custom_pos  : True  ; use custom park coordinates for x,y [True/False] 
variable_custom_park_x   : 150.0 ; custom x position; value must be within your defined min and max of X
variable_custom_park_y   : 10.0  ; custom y position; value must be within your defined min and max of Y
#variable_custom_park_dz  : 2.0   ; custom dz value; the value in mm to lift the nozzle when move to park position 
#variable_retract         : 1.0   ; the value to retract while PAUSE
#variable_cancel_retract  : 5.0   ; the value to retract while CANCEL_PRINT
#variable_speed_retract   : 35.0  ; retract speed in mm/s
#variable_unretract       : 1.0   ; the value to unretract while RESUME
#variable_speed_unretract : 35.0  ; unretract speed in mm/s
#variable_speed_hop       : 15.0  ; z move speed in mm/s
#variable_speed_move      : 100.0 ; move speed in mm/s
#variable_park_at_cancel  : False ; allow to move the toolhead to park while execute CANCEL_PRINT [True,False]
## !!! Caution [firmware_retraction] must be defined in the printer.cfg if you set use_fw_retract: True !!!
#variable_use_fw_retract  : False ; use fw_retraction instead of the manual version [True/False] 
gcode:
```

You can also uncomment all variables if you like. The values are the same as the user default.

##### Different park position

The following descrips how to setup a custom position for CANCEL_PRINT and PAUSE. As an example we asume the bed has asize of (300x300mm) and we want:

- Position for CANCEL_PRINT: back right (295x295 mm)
- Position for PAUSE       : front left (10x10 mm)

First copy the complete _CLIENT_VARIABLE macro from the mainsail.cfg and place it below your mainsail include. After that uncomment the needed variables or all. The values are the same as the default.
After that we need to enter the needed values. See the result below:

```ini
[include mainsail.cfg]

[gcode_macro _CLIENT_VARIABLE]
variable_use_custom_pos   : True  ; use custom park coordinates for x,y [True/False]
variable_custom_park_x    : 10.0  ; custom x position; value must be within your defined min and max of X
variable_custom_park_y    : 10.0  ; custom y position; value must be within your defined min and max of Y
variable_custom_park_dz   : 2.0   ; custom dz value; the value in mm to lift the nozzle when move to park position
variable_retract          : 1.0   ; the value to retract while PAUSE
variable_cancel_retract   : 5.0   ; the value to retract while CANCEL_PRINT
variable_speed_retract    : 35.0  ; retract speed in mm/s
variable_unretract        : 1.0   ; the value to unretract while RESUME
variable_speed_unretract  : 35.0  ; unretract speed in mm/s
variable_speed_hop        : 15.0  ; z move speed in mm/s
variable_speed_move       : 100.0 ; move speed in mm/s
variable_park_at_cancel   : True  ; allow to move the toolhead to park while execute CANCEL_PRINT [True/False]
variable_park_at_cancel_x : 295.0 ; different park position during CANCEL_PRINT [None/Position as Float]; park_at_cancel must be True
variable_park_at_cancel_y : 295.0 ; different park position during CANCEL_PRINT [None/Position as Float]; park_at_cancel must be True
# !!! Caution [firmware_retraction] must be defined in the printer.cfg if you set use_fw_retract: True !!!
variable_use_fw_retract  : False ; use fw_retraction instead of the manual version [True/False]
gcode:
```

This way insures that older configs still work as before.

### New Feature: "Pause at next Layer" and "Pause at Layer #"

This is based on a idea of Pedro Lamas. It let you add a Pause at the next layer change or if you reach a specific layer number.

First you need to prepare your slicer as described in https://github.com/Klipper3d/klipper/pull/5726

If you done that you can either use

```txt
SET_PAUSE_NEXT_LAYER [ENABLE=1] [MACRO=<name>]
```

to get execute the given GCODE macro at the next layer change. The MACRO is normally either PAUSE (default) or M600 (if you have specified it in your printer.cfg).

Or use

```txt
SET_PAUSE_AT_LAYER [ENABLE=1] [LAYER=<number>] [MACRO=<name>]
```

to get execute the given GCODE macro at the given LAYER number change. The MACRO is normally either PAUSE (default) or M600 (if you have specified it in your printer.cfg).

To remove the "Pause at next Layer" simple send

```txt
SET_PAUSE_AT_LAYER ENABLE=0
```

To remove the "Pause at Layer" simple send

```txt
SET_PAUSE_AT_LAYER [ENABLE=0]
```

Both will clear after execution.
