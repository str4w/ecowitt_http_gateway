# ecowitt_http_gateway
<i>Simple HTTP gateway that receives data from <b>GW1000 with Ecowitt protocol</b> and resend data to Meteotemplate or csv, json, ecc.</i><br>
Example on: http://www.kwos.org/poggiocorese_ecowitt
<br><br>
<b><i>Install this gateway if you have a web server at home, like a Raspberry or something where you want to store weather data</b></i>
  
The GW1000 allows sending data both to Ecowitt.net and Wunderground, even to an external site as long as you select one
of the two previous protocols. <br>
We know the Wunderground protocol and we know that it doesn't send UV and PM2.5 data, nor ground temperature or
other additional sensors, so you need to select the Ecowitt protocol.<br>

Now to permit this script to work, we need a web server to which the GW1000 need to send data.

<b>REQUIREMENTS</B>
<br>
The web server must have these possibilities:
- create a directory named <b>/data/report</b> (es. /var/www/html/data/report )
- in this directory will be put the <b>index.php</b> file 

So, the web site will look like: http://192.168.2.185/data/report/index.php<br>
In the GW1000 configuration it will be necessary writing only the IP address, es. 192.168.1.4 and specify the update rate.<br>
<br>
I recommend having this web server on a raspberry, in the same network of the GW1000, so the script can also be used to store data without losing them in case of Internet connection failure<br> 
When the GW1000 will contact the web site, the <i>index.php</i> will do these functions:

1) creates a .JSON file in /var/log/ecowitt ( overwrited every update, contains only last data )<br>
2) creates a .CSV file in /var/log/ecowitt ( appended every update, contains all data )<br>
3) converts in metric all data a resend to a Meteotemplate web site on Internet<br>
4) can create and send via FTP a file for the weather station registered on Meteonetwork.it 
5) creates a weewx.txt file in /var/log/ecowitt for the [ecowitt.py](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/ecowitt.py) driver for WeeWX

<b>HOW TO INSTALL</b>:
- Install Apache
- Install PHP
- Install jq ( for JSON query )
- Create directory Es. /var/www/html/data/report/
- Create /var/log/ecowitt with chmod 777
- Put file: index.php in /var/www/html/data/report/
- Configure index.php
- Configure GW1000 to send data to your server

<i>Note:</i><br>
<i>Look in /var/log/ecowitt to read fields using 'jq'<br>
jq -r '.tempc' weather_XXXXXXXXXXXXXXXX.json</i>

#
#
# ecowitt WeeWX driver

Called <b>ecowitt.py</b> <br>
This driver works in two modes:

- <b>normal</b>: reads data from a file generated by the [ecowitt_http_gateway](https://github.com/iz0qwm/ecowitt_http_gateway/blob/master/README.md#ecowitt_http_gateway) - Usually /var/log/ecowitt/weewx.txt<br>
name=value pair, for example:<br>

outTemp=79.3<br>
barometer=29.719<br>
pressure=29.719<br>
outHumidity=70<br>

- <b>server</b>: open a socket server that receives directly the GW1000. Configure it in the weewx.conf<br>
with the IP address where the GW1000 will connect to. Configure the GW1000 with the IP and the port, put / on the path.

## Automatic extension installation

Run the extension installer:

<b>sudo wee_extension --install weewx-ecowitt-x.x.tgz</b>
<br><br>
Modify weewx.conf:
<pre>
<b>[Station]
     station_type = ecowitt
</b>
</pre>
Check the automatic addition at the end of weewx.conf:<br>
<pre>
<b>
[ecowitt]
     poll_interval = 65                    # number of seconds, just a little more than the GW1000 update time
     path = /var/log/ecowitt/weewx.txt     # location of data file generated by ecowitt_http_gateway used in mode=normal
     driver = weewx.drivers.ecowitt
     mode = normal                         # normal = use with ecowitt_http_gateway - server = directly connected to GW1000
     address = 192.168.2.185               # IP address of the PC where weewx is running in mode=server, to which GW1000 will connect to                
     port = 9999                           # port to which GW1000 will connect to in mode=server
</b>
</pre>

Restart weewx<br>
<pre>
<b>
sudo /etc/init.d/weewx stop
sudo /etc/init.d/weewx start
</b>
</pre>
## Manual installation
To use this driver, put the ecowitt.py file in the weewx drivers directory (i.e. /usr/share/weewx/weewx/drivers ), then make<br>
the following changes to weewx.conf:<br>
<br>
<pre>
<b>[Station]
     station_type = ecowitt
[ecowitt]
     poll_interval = 65                    # number of seconds, just a little more than the GW1000 update time
     path = /var/log/ecowitt/weewx.txt     # location of data file generated by ecowitt_http_gateway
     driver = weewx.drivers.ecowitt
     mode = normal                         # normal = use with ecowitt_http_gateway - server = directly connected to GW1000
     address = 192.168.2.185               # IP address of the PC where weewx is running in mode=server, to which GW1000 will connect to                
     port = 9999                           # port to which GW1000 will connect to in mode=server
</b>
</pre>

In the weewx.conf or skinf.conf use the following Labels
<pre>
[Labels]
                [[Generic]]
                # Sensor status indicators
                txBatteryStatus      = WH51-Soil
                windBatteryStatus    = WS80-Temp/Wind
                rainBatteryStatus    = WH40-Rain
                outTempBatteryStatus = WH31_1-Temp/Hum
                inTempBatteryStatus  = Inside Temperature
                consBatteryVoltage   = WS80-Temp/Wind
                heatingVoltage       = WH51-Soil
                supplyVoltage        = WH40-Rain
</pre>
<b>NOTE</b><br>
If you don't want to use the ecowitt_http_gateway, you can use the [Interceptor driver](https://github.com/matthewwall/weewx-interceptor) to sniff data of your GW1000.

#
#
# ecowitt Meteotemplate plugin
<i>Simple plugin for [Meteotemplate](http://www.meteotemplate.com/), wonderful template developed by Jachym.</i><br><br>
<b><i>Install this plugin if you don't want to install the previous gateway and weewx, or if you only need to update your Meteotemplate web site. You will not store any data locally in your network</b></i>

- Download it from the repository
- Install it in the plugin directory of your template website, just like another plugin. (put in your plugins directory only the directory called <i>ecowitt</i> without the version number: <del>Meteotemplate/ecowitt_x.x/</del>ecowitt) 
- Go in the Plugin setup page, via Admin Panel of Meteotemplate
- Configure it
- Configure the GW1000 with the setup you read in the Plugin page. The web server port ususally is <b>80</b>

![Example of plugin page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_plugin1.png)
![Example of plugin admin page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_plugin2.png)
![Example of plugin admin page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_plugin3.png)

#
#
# ecowitt Meteotemplate blocks
<b> Remember to update the ecowitt Meteotemplate plugin to the latest version (or the index.php if you have your own webserver)</b>
![Example of airquality block page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_block_airquality1.png)
![Example of airquality block page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_block_airquality2.png)

![Example of modules block page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_block_modules1.png)
![Example of modules block page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_block_modules2.png)
![Example of modules block page](https://raw.githubusercontent.com/iz0qwm/ecowitt_http_gateway/master/images/ecowitt_block_modules3.png)


#
#
# NOTE from the issue forum

Googling: https://fhem.de/
Is a server for home automation

If you already have a Meteotemplate webserver you can use 3 mode of uploading mode:

1. **use the Meteotemplate plugin and configure GW1000 to send data to directly.**<br>
Follow only the section with title: _[ecowitt Meteotemplate plugin](https://github.com/iz0qwm/ecowitt_http_gateway/#ecowitt-meteotemplate-plugin)_<br>
example: http://www.kwos.org/meteotemplateweb/api.php<br>
Note: You have all extra sensors like soilmoisture, PM25

2. **have a raspberry at home, in the internal network, with a webserver where there is the index.php**<br> 
Follow only the section with the title: _[ecowitt_http_gateway](https://github.com/iz0qwm/ecowitt_http_gateway/#ecowitt_http_gateway)_<br>
In this case you don't need the Meteotemplate plugin but you have to configure the index.php to send data via api to Meteotemplate.<br>
 example: http//192.168.2.185/data/report/<br>
Note1: You have all extra sensors like soilmoisture, PM25 sent to Meteotemplate<br>
Note2: You can also send data to a weewx without extra sensors

4. **have a raspberry at home with weewx and weewx driver for GW1000.**<br>
Follow both the sections: _[ecowitt_http_gateway](https://github.com/iz0qwm/ecowitt_http_gateway/#ecowitt_http_gateway)_ and _[ecowitt WeeWX driver](https://github.com/iz0qwm/ecowitt_http_gateway/#ecowitt-weewx-driver)_<br>
Then configure weewx with the Meteotemplate plugin released here:<br>
https://github.com/matthewwall/weewx-meteotemplate<br>
Note: you don't have the extra sensors 
