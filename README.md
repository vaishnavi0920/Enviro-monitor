# Nature-monitor
This project utilizes a Raspberry Pi Zero W, to monitor, showcase, and report various aspects of the air, including particles, gases, temperature, humidity, air pressure, light levels, and noise levels. It offers the option to include an SGP30 sensor to monitor eCO2 and TVOC levels. The code is primarily based on Python examples and libraries provided by Pimoroni, with several modifications and improvements.

Additionally, a basic weather forecast feature has been incorporated, relying on air pressure levels and their fluctuations.

The original light level display in the Weather and Light program has been altered to represent air quality levels instead. The background color now reflects the air quality level, and a visible sun icon indicates the sun's position. The program also offers weather forecast details, optional noise level monitoring, and minor adjustments to the humidity indicator.

The Combined function has undergone modifications to enhance the clarity of each graph. Graph colors are now determined by predefined thresholds for each parameter, and only the measured parameters are displayed. The 'display_everything' method has also been adjusted to focus solely on air quality parameters, improving the legibility of the display.

The All in One function has been altered to enable cycling through all of the Nature Monitor's functions.

Extensive testing and regression analysis have been conducted to improve the accuracy of temperature and humidity measurements. This involved developing more effective compensation algorithms within the temperature range of 0 to 40 degrees Celsius. 

In addition to improving temperature and humidity measurements, testing and regression analysis was undertaken to provide time-based drift, temperature, humidity and air pressure compensation for the Enviro+ gas sensors. Algorithms and clean-air calibration are also used to provide gas sensor readings in ppm. A data logging function is provided to support the regression analysis. The log file for that analysis needs to be enabled and converted to a valid json format before undertaking further regression analysis.

Accuracy of the air pressure readings is delivered through altitude compensation. The altitude is set by the ‘altitude’ parameter in the config.json file.

# Note: Even though the accuracy of the Nature Monitor has been improved, the readings are still not thoroughly and accurately calibrated. The calibration exercise has been undertaken for temperatures between 0 and 40 degrees Celsius, so it's unlikely to provide effective readings outside of that range. They readings should therefore not be relied upon for critical applications.
The case is not water resistant and needs to be sheltered from the elements. The base is only required if the unit is not mounted on a vertical surface. There is a variant of the case and cover for the Indoor Plus model that monitors eCO2 and TVOC levels. This variant of the case provides additional space and airflow for the SGP30 sensor.

The case also has the option of adding a weather cover to provide additional protection from the elements. When using this cover, it is necessary to set "enable_display" in the config.json file to "false". That limits the display functionality to just air quality-based hue and serial number, as well as changing the temperature and humidity compensation variables to mitigate the effect of the cover on the temperature and humidity sensor.

Approximate noise levels measurements have been added to Version 6, based on this repository. This feature has not been calibrated and is not to be used for accurate sound level measurements. Version 6.7 has improved frequency compensation of the noise level measurement function, using this, but still further work and calibration is required. This noise level measurement function requires additional setup (described below) and after setup, needs to be enabled in the configuration file.

mqtt support is provided to enable the use of external temperature and humidity sensors (for data logging and regression analysis), interworking between the nature Monitor and a home automation system and to support interworking between outdoor and indoor Nature Monitors. That interworking allows the display of an indoor Enviro Monitor to cycle between indoor and outdoor readings.

An alternative to using mqtt-linked indoor and outdoor Nature Monitors to get outdoor readings on an indoor Enviro Monitor, is to configure the indoor Nature Monitor to capture Luftdaten readings or Adafruit IO feeds from another Nature Monitor.

Luftdaten interworking has been modified to support the addition of minimum, maximum and mean noise level readings. Noise level readings can be sent to Luftdaten by setting "enable_luftdaten_noise" to true in the config.json file. Note that Luftdaten can't currently be configured with three sensors per node, so noise level readings can therefore only be sent to Luftdaten if either PM or climate readings are disabled. That can be done by setting "disable_luftdaten_sensor_upload" in the config.json file to either "Climate" or "PM".

The same Enviro+ setup is used to set up the Nature Monitor and the config.json file parameters are used to customise its functionality. A description of the config.json file's parameters is here.

Setting up of the noise level measurements requires the following additional steps:

# Additional Noise Measurement Setup
Successful execution of this setup is necessary before enabling noise measurement in the config file.

sudo apt-get update

sudo apt-get upgrade

curl -sSL https://get.pimoroni.com/enviroplus | bash

sudo python -m pip uninstall sounddevice

sudo pip3 install sounddevice==0.3.15

For Versions 6.7 and later, also do:

sudo apt-get install python3-scipy

sudo pip3 install git+https://github.com/endolith/waveform_analysis.git@master

Then follow the instructions at: https://learn.adafruit.com/adafruit-i2s-mems-microphone-breakout/raspberry-pi-wiring-test including “Adding Volume Control”

Use the following instead of the documented text for ~/.asoundrc:

#This section makes a reference to your I2S hardware, adjust the card name
#to what is shown in arecord -l after card x: before the name in []
#You may have to adjust channel count also but stick with default first
pcm.dmic_hw {
type hw
card adau7002
channels 2
format S32_LE
}
#This is the software volume control, it links to the hardware above and after
#saving the .asoundrc file you can type alsamixer, press F6 to select
#your I2S mic then F4 to set the recording volume and arrow up and down
#to adjust the volume
#After adjusting the volume - go for 50 percent at first, you can do
#something like
#arecord -D dmic_sv -c2 -r 48000 -f S32_LE -t wav -V mono -v myfile.wav
pcm.dmic_sv {
type softvol
slave.pcm dmic_hw
control {
name "Master Capture Volume"
card adau7002
}
min_dB -3.0
max_dB 30.0
}
For versions prior to Version 6.7:

Use alsamixer to set adau7002 capture level to 50

For Version 6.7 and later:

Use alsamixer to set adau7002 capture level to 10

A User Guide provides guidance on the use of the Enviro Monitor.

# Adafruit IO Support
Support is provided for streaming weather forecast, air quality, temperature, humidity, air pressure, PM concentration, gas concentration, light levels, noise levels and, with the optional SGP30 sensor, eCO2 and TVOC data to Adafruit IO. This can be enabled and set up as follows:

# Config File Settings to Support Adafruit IO
The following fields in the Enviro Monitor’s config.json file need to be populated to supply data to the Adafruit IO feeds.

"enable_adafruit_io": Set to true to enable and false to disable Adafruit IO feeds,

"aio_user_name": "Your Adafruit IO User Name",

"aio_key": "Your Adafruit IO Key",

"aio_feed_window": Value between 0 and 9. Sets the start time for the one minute feed window (see Adafruit Throttling Control). Set to 0 if you only have one Enviro Monitor,

"aio_feed_sequence": Value between 0 and 3. Sets the feed update start time within the one minute feed update window (see Adafruit Throttling Control). Set to 0 if you only have one Enviro Monitor,

"aio_household_prefix": "The Adafruit IO Key Prefix for the household you’re monitoring (see Adafruit IO Naming Convention)",

"aio_location_prefix": "The Adafruit IO Key Prefix for the location of this particular Enviro Monitor. Use ‘indoor’ for an indoor monitor or ‘outdoor’ for an outdoor monitor. (see Adafruit IO Naming Convention)",

"aio_package": Set to "Premium Plus" or "Premium Plus Noise" or "Premium" or "Premium Noise" or "Basic Air" or "Basic Combo"

You will need an Adafruit IO+ account in order to use ‘Premium Plus’, 'Premium Plus Noise', ‘Premium’ or 'Premium Noise' packages and an Enviro Monitor Indoor Plus (equipped with an SGP30 eCO2/TVOC sensor) for the ‘Premium Plus’ or 'Premium Plus Noise' packages (see Adafruit IO Packages)",

# Adafruit IO Feed, Dashboard and Block Setup
The script sets up the Enviro Monitor’s Adafruit IO feeds, dashboards and blocks like this example

The script can set up multiple households and locations in one run, by populating the aio_feed_prefix dictionary with the required data. The format for aio_feed_prefix is:

aio_feed_prefix = {'Household 1 Name': {'key': 'household1key', 'package': 'aio_package', 'locations': {'Location1Name': 'location1key', 'Location2Name': 'location2key'}, 'visibility': 'public' or 'private'}, 'Household 2 Name': {'key': 'household2key', 'package': 'aio_package', 'locations': {'Location1Name': 'location1key'}, 'visibility': 'public' or 'private'}}

The Household Names and Household Keys need to be consistent with those defined in the relevant Enviro Monitors’ config.json files.

For example, if you only have one Enviro Monitor for your household, and if you’ve set "aio_household_prefix" to “home”, "aio_location_prefix" to “outdoor” and "aio_package" to “Premium” in your config.json file for that Enviro Monitor, and if you want the feeds, dashboards and blocks set with private visibility:

aio_feed_prefix = {‘Home’: {'key': 'home', 'package': Premium', 'locations': {‘Outdoor': 'outdoor’}, 'visibility': 'private'}}

If you have two Enviro Monitors for your household, and if you’ve set the config.json files as "aio_household_prefix" to “home” for both Enviro Monitors, "aio_location_prefix" to “outdoor” for the outdoor monitor and “indoor” for your indoor monitor, "aio_package" to “Premium” for your outdoor monitor and “Premium Plus” for your indoor monitor, and if you want the feeds, dashboards and blocks set with public visibility:

aio_feed_prefix = {‘Home’: {'key': 'home', 'package': Premium Plus', 'locations': {‘Outdoor': 'outdoor’, ‘Indoor’: ‘indoor’}, 'visibility': 'public'}}

The two other user-defined dictionaries are aio_user_name and aio_key. These need to be populated with the same user name and key that you used in your Enviro Monitor’s config.json file.

aio_user_name = "Your Adafruit IO User Name"

aio_key = "Your Adafruit IO Key"

# Adafruit IO Throttling Control
If enabled, Adafruit IO feed updates are generated every 10 minutes. The config file's aio_feed_window and aio_feed_sequence variables are used to minimise Adafruit IO throttling errors when collecting feeds from multiple Enviro Monitors. The aio_feed_window variable can be a value between 0 and 9 to set the start time for a one minute feed update window. 0 opens the window at 0, 10, 20, 30, 40 and 50 minutes past the hour, 1 opens the window at 1, 11, 21, 31, 41, and 51 minutes past the hour, 2 opens the window at 2, 12, 22, 32, 42 and 52 minutes past the hour, and so on. The aio_feed_sequence variable can be a value between 0 and 3 to set the feed update start time within the one minute feed update window. 0 starts the feed update immediately after the window opens, 1 delays the start by 15 seconds, 2 by 30 seconds and 3 by 45 seconds.

# Adafruit IO Naming Convention
The naming convention for each Enviro Monitor’s Adafruit IO feeds, dashboards or blocks, is to use the name of the household, followed by the location of the relevant Enviro Monitor’s location within that household, as a prefix for each feed, dashboard or block. You choose a suitable name for "aio_household_prefix", and "aio_location_prefix" can either be “indoor” or “outdoor”. For example, setting “aio_household_prefix" to “home” and "aio_location_prefix" to “outdoor” will set the prefix of each feed’s name as “Home Outdoor “ and the prefix of each feed’s key as “home-outdoor-“. So, the Temperature Feed will have the name “Home Outdoor Temperature” and the key “home-outdoor-temperature”. The dashboard will have the name “Home” and key “home” and the temperature gauge block within that dashboard will have the name “Outdoor Temperature Gauge” and the key “outdoor-temperature-gauge”.

# Adafruit IO Packages
Six Adafruit IO package options are available: "Premium" with 14 data feeds per Enviro, "Premium Noise" with 17 data feeds per Enviro, "Premium Plus" with 16 data feeds per Enviro (i.e. the addition of eCO2 and TVOC through the optional SGP30 sensor), "Premium Plus Noise" with 19 data feeds per Enviro which all need an Adafruit IO+ account; "Basic Air" with 5 air quality data streams (Air Quality Level, Air Quality Text, PM1, PM2.5 and PM10) and "Basic Combo" with 5 air quality/climate streams (Air Quality Level, Weather Forecast Icon, Temperature, Humidity and Air Pressure).

# Use of Adafruit IO Noise Packages
Using the "Premium Noise" and "Premium Plus Noise" Adafruit IO packages requires configuring and enabling Noise measurements in the Enviro, using the relevant setup instructions. Version 6.5 changes the noise feeds and dashboards to show Max, Min and Mean noise levels between feed updates, whereas prior versions only showed Max noise levels between feed updates.

# License
This project is licensed under the MIT License - see the LICENSE.md file for details

# Acknowledgements
Weather Forecast based on www.worldstormcentral.co/law_of_storms/secret_law_of_storms.html by R. J. Ellis
