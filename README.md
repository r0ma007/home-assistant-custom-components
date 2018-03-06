# Home-Assistant Custom Components

Custom components I made for use with [home-assistant hass.io](http://www.home-assistant.io).

## Shabbat Times Custom Sensor

This component acts as a new platform called *shabbat_times* for the *sensor* domain.
The component works in a simillar manner as the *rest* type sensor, it send an api request towards [Hebcal's Shabbat Times API](https://www.hebcal.com/home/197/shabbat-times-rest-api) and retrievs the **next** or **current** shabbat start and end date and time, and sets them as attributes within a created sensor.</br>
The component can create multiple sensors for multiple cities around the world, the selected city is identified by its geoname which can selected [here](https://github.com/hebcal/dotcom/blob/master/hebcal.com/dist/cities2.txt).

### Installation

- Copy file [`custom_components/sensor/shabbat_times.py`](custom_components/sensor/shabbat_times.py) to your `ha_config_dir/custom_components/sensor` directory.
- Configure with config below.
- Restart Home-Assistant.

## Usage
To use this component in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml

sensor:
  - platform: shabbat_times
    geonames: VALID_GEONAME
```

Configuration variables:

- **geonames** (*Required*): A valid geoname selected [here](https://github.com/hebcal/dotcom/blob/master/hebcal.com/dist/cities2.txt), multiple geonames seperated by a comma is allowed.
- **candle_lighting_minutes_before_sunset** (*Optional*): Minutes to subtract from the sunset time for calculation of the candle lighting time. (default = 30)
- **havdalah_minutes_after_sundown** (*Optional*): Minutes to add to the sundown time for calculation of the shabbat end time. (default = 42)
- **scan_interval** (*Optional*): Seconds between updates. (default = 60)

Working Configuration Example:

```yaml
# Example configuration.yaml

sensor:
  - platform: shabbat_times
    geonames: "IL-Haifa,IL-Rishon LeZion"
    candle_lighting_minutes_before_sunset: 0
    havdalah_minutes_after_sundown: 40
```
This configuration will create two sensors:
- *sensor.shabbat_times_il_haifa*
- *sensor.shabbat_times_il_rishon_lezion*

Each sensor will have its own set of attributes:
- *shabbat_start*
- *shabbat_end*

Which will be calulated based on configration optional values **candle_lighting_minutes_before_sunset** and **havdalah_minutes_after_sundown**.
These attributes are available for use within templates like so:
- *{{ states.sensor.shabbat_times_il_haifa.attributes.shabbat_start }}* will show the shabbat start date and time in Haifa.
- *{{ states.sensor.shabbat_times_il_rishon_lezion.attributes.shabbat_end }}* will show the shabbat end date and time in Rishon Lezion.

Sensor States:

The created sensors has 4 possible states:
- *Awaiting Update*: the sensor hasn't been updated yet.
- *Working*: the sensor is being updated at this moment.
- *Error...*: the api has encountered an error.
- *Updated*: the sensor has finished updating.

Any state besides *Updated* is rarely used and will probably add an error message in home assistant's logs. If so, please check the log and try to correct the error if its source is in the configration parameters. If a code modification is required please create a new issue.

**Special Note**: The sensors will allways show the date and time for the next shabbat, unless the shabbat is now, and therefore the sensors will show the current shabbat date and time.

## Date Notifier Custom Component

This component is called *date_notifier* and it is dependent on the **notify component**, it's used for creating reminders based on dates and times.</br>
Before using this component, please configure notifications using the [**notify component**](https://home-assistant.io/components/notify/) instructions.</br>
The **Date Notifier Component** supports four types of reminders:
- Yearly recurring reminder
- Monthly recurring reminder
- Daily recurring reminder
- One Time non-recurring reminder

### Installation

- Copy file [`custom_components/date_notifier.py`](custom_components/date_notifier.py) to your `ha_config_dir/custom_components` directory.
- Configure with config below.
- Restart Home-Assistant.

## Usage
There are four type of reminders, yearly, monthly, daily and one time reminders.</br>
The type of reminder is decided based on the configuration variables.</br>

**The following configuration variables are required for any reminder, use only the following for configuring a daily reminder:**
- **name** (*Required*): Any string representing the name of the reminder.
- **hour** (*Required*): A **positive integer between 0 and 23** represnting the hour of the notification arrival.
- **minute** (*Optional*): A **positive integer between 0 and 59** represnting the minute of the notification arrival. (default = 0)
- **message** (*Required*): Any string representing the message for the notification, the message will concatenated with a predefined text represnting the number of days to the event.
- **notifier** (*Required*): A valid *notifier name* to be used as the recipient of the notification.

**The following configuration variable is optional, add the following to all of previous for configuring a monthly reminder:**
- **day** (*Optional*): A **positive integer between 1 and 31** represnting the day for a monthly reminder. (default = None)

**The following configuration variable is optional, add the following to all of previous for configuring a yearly reminder:**
- **month** (*Optional*): A **positive integer between 1 and 12** represnting the month for a yearly reminder. (default = None)

**The following configuration variable is optional, add the following to all of previous for configuring a one time reminder:**
- **year** (*Optional*):  A **positive 4 digits integer** represnting the year for a one time reminder. (default = None)

**The following configuration variable is optional and eligible when configuring a monthly, yearly or one time reminders:**
- **days_notice** (*Optional*): A **postive integer** represnting the number of days before the date in which the notification will be send. (default = 0)

Working Configuration Example:

```yaml
# Example configuration.yaml

date_notifier:
# One Time Reminder will be send 1 day before the event date, on date 2017-11-19 at 21:25
  one_time_reminder:
    name: "one-time test"
    hour: 21
    minute: 25
    day: 20
    month: 11
    year: 2017
    message: "one-time test"
    days_notice: 1
    notifier: "ios_tomers_iphone6s"

# Yearly Reminder will be send 2 days before the event date every year, on November 19th at 21:26
  yearly_reminder:
    name: "yearly test"
    hour: 21
    minute: 26
    day: 21
    month: 11
    message: "yearly test"
    days_notice: 2
    notifier: "ios_tomers_iphone6s"

# Monthly Reminder will be send on the 19th of every month at 21:27
  monthly_reminder:
    name: "montly test"
    hour: 21
    minute: 27
    day: 19
    message: "montly test2"
    notifier: "ios_tomers_iphone6s"
  
# Daily Reminder will be send every day at 21:28
  daily_reminder:
    name: "daily test"
    hour: 21
    minute: 28
    message: "daily test"
    notifier: "ios_tomers_iphone6s"
```
Based on this configuration, I've received four notifications withing four minutes, you can see the received notifications [here](sample_pics/date_notifier_notifications.jpg) (the picture was taken on date 2017-11-19 at 21:28).

**Entity States**
Each reminder will create it's own entity with the configuration variables as state attributes, there are five potential states:
- *daily*: for daily reminders.
- *monthly*: for monthly reminders.
- *yearly*: for yearly reminders.
- *on_date*: for future one time reminders.
- *past_due*: for past one time reminders.

**Special Notes**:
- This component is dependent on the **notify component**, before using this component, please configure notification using the [**notify component**](https://home-assistant.io/components/notify/) instructions.
- In future releases I plan on adding another configure variable of boolean type called *countdown*, when true reminders with a *days_notice* variable bigger then 0, will launch a "countdown" everyday starting with the *days_notice* limit and ending at the day of the event.
- In future releases I plan on adding a service for relaoding the configuration, for now, when editing any active reminders or adding new ones, a Home Assistant restart is required.

## Switcher V2 Bolier

This custom component is based on the awesome script made available by **NightRang3r**, you can find the original script and the instruction on how to retrieve your device's information in Shai's repository [here](https://github.com/NightRang3r/Switcher-V2-Python).</br>
This component acts as a new platform called *switcher_heater* for the *switch* domain, the end result should look something like [this](/sample_pics/switcher.jpg).

### Requirements
- **Home Assistant version 0.62 or higher** (tested with Hassio 0.63.2 and Hassabian 0.63.3).
- Your switcher device needs to have a **Static IP Address** reserved by your router.
- Please follow [Shai's instructions](https://github.com/NightRang3r/Switcher-V2-Python#requirements) and **gather the following information**:
  - phone_id
  - device_id
  - device_pass

### Installation

- Copy file [`custom_components/switch/switcher_heater.py`](custom_components/switch/switcher_heater.py) to your `ha_config_dir/custom_components/switch` directory.
- Configure like instructed in the Usage section below.
- Restart Home-Assistant.

## Usage
To use this component in your installation, add the following to your `configuration.yaml` file, supports multiple devices:

```yaml
# Example configuration.yaml

switch:
  - platform: switcher_heater
    switches:
      device1:
        friendly_name: "heater_switch"
        local_ip_addr: 'XXX.XXX.XXX.XXX'
        phone_id: 'XXXX'
        device_id: 'XXXXXX'
        device_password: 'XXXXXXXX'
```

Configuration variables:

- **friendly_name** (*Required*): A string representing the friendly name of your device.
- **local_ip_addr** (*Required*): The IP Address assigned to your device by your router. A static address is preferable.</br>

The following was retrieved based on NightRang3r original instructions:
- **phone_id** (*Required*): Your phone id.
- **device_id** (*Required*): Your device id.
- **device_password** (*Required*): Your device password.</br>

The end result should look something like [this](/sample_pics/switcher.jpg).

## Broadlink S1C Alarm kit

This custom component is based on the script made available by [**NightRang3r**](https://community.home-assistant.io/t/broadlink-s1c-kit-sensors-in-ha-using-python-and-mqtt/19886), and the [python-broadlink repository](https://github.com/mjg59/python-broadlink).</br>
This component acts as a new platform called *broadlink_s1c* for the *sensor* domain.

### Requirements
- **Home Assistant version 0.62 or higher** (tested with Hassio 0.63.3).
- Your S1C Hub needs to have a **Static IP Address** reserved by your router.

### Installation

- Copy the file [`custom_components/sensor/broadlink_s1c.py`](custom_components/sensor/broadlink_s1c.py) to your `ha_config_dir/custom_components/sensor` directory.
- Configure like instructed in the Usage section below.
- Restart Home-Assistant.

## Usage
To use this component in your installation, add the following to your `configuration.yaml` file:

```yaml
# Example configuration.yaml

sensor:
  - platform: broadlink_s1c
    ip_address: "xxx.xxx.xxx.xxx"
    mac: "xx:xx:xx:xx:xx:xx"
```

Configuration variables:

- **ip_address** (*Required*): The IP Address assigned to your device by your router. A static address is preferable.</br>
- **mac** (*Required*): The MAC Address of your S1C Hub.</br>

## Sensor Available States
### All Sensors
- `tampered`
- `unknown` - Inherited from *homeassistant.const.STATE_UNKNOWN*
### Door Sensor
- `open` - Inherited from *homeassistant.const.STATE_OPEN*
- `closed` - Inherited from *homeassistant.const.STATE_CLOSED*
### Motion Detection Sensor
- `no_motion`
- `motion_detected`
### Key Fob Sensor
- `disarmed` - Inherited from *homeassistant.const.STATE_ALARM_DISARMED*
- `armed_away` - Inherited from *homeassistant.const.STATE_ALARM_ARMED_AWAY*
- `armed_home` - Inherited from *homeassistant.const.STATE_ALARM_ARMED_HOME*
- `sos`

## Special Notes
- Initial configuration of the sensor in the Broadlink App is required.
- The platform discovers the sensors upon loading, therefore if you add another sensor, restart Home Assistant and the new sensors will be added to ha.
- The entity name of each sensor is constructed from the original sensor name from the Broadlink App concatenated with the platform name. Spaces and dashes will be replaced with underscores.</br>
  For instance, if you sensor is name *Bedroom Door* the entity name will be *broadlink_s1c_bedroom_door*, and to reference  it you will call *sensor.broadlink_s1c_bedroom_door*
