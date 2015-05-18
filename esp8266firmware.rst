ESP8266 firmware
................

The firmware has the following main objectives:

* When booted it should enter a grace period and let the user config which access point to join
* After a grace period it should lookup the current time from an NTP server and save it to its rtc and then enter sample mode
* Everything in the firmware is timer based.

Config mode
,,,,,,,,,,,

.. figure::
   diagrams/config_mode.png
   :figwidth: 80%
   :scale: 200%

   Diagram shows the successful configuration of an ESP8266 using a smartphone within the graceperiod


.. comment
	.. seqdiag::
	   seqdiag {
	    ESP8266 -> ESP8266 [label = "set wifimode to \nAP (access point)\nrun tcp server\nstart graceperiod "];
	    ESP8266 <- smartphone [label = "smartphone joins AP"];
	    ESP8266 <- smartphone [label = "http GET index"];
	    ESP8266 -> smartphone [label = "serve html with list of AP's"];
	    ESP8266 <- smartphone [label = "smartphone configs new\ntarget AP"];
	    ESP8266 -> ESP8266 [label = "saves target AP\ncredentials.\nGrace period times out"];
	    ESP8266 -> ESP8266 [label = "set wifimode to \nSTA (station)"];
	    ESP8266 -> target-AP [label = "join with saved credentials"]
	   }

This mode features a grace period where the esp will act as an access point
and serve a html site where the user can edit which access point the esp will join later on.
The config mode access point is protected by wpa encryption and credentials are generated
when flashing a new esp.

If the user changes the prefered access point of the esp it will be saved using sdk functions for saving and loading configs.
These api calls go to the binary blobs and are not documented further.
The configs are permanently saved.

After the graceperiod ends the esp will set its wifi mode to station and try to connect to saved ssid with saved credentials.

Ntp transition
,,,,,,,,,,,,,,

.. figure::
   diagrams/ntp_transition.png
   :figwidth: 80%
   :scale: 200%

   Diagram shows how we use the builtin DNS to lookup NTP servers, request a timestamp and store it

.. comment
	.. seqdiag::
	  
	   seqdiag {
	   ESP8266 -> "google dns" [label = "espconn_gethostbyname\ndk.pool.ntp.org"];
	   ESP8266 <- "google dns" [label = "83.151.158.44"];
	   ESP8266 -> "dk.pool.ntp.org" [label = "timestamp request"];
	   ESP8266 <- "dk.pool.ntp.org" [label = "receive timestamp"];
	   ESP8266 -> ESP8266 [label = "unpack and\nsave timestamp\n\nstart 1-second timer"];
	   ESP8266 <- "ESP8266\n1-second timer" [label = "timer interupt"];
	   ESP8266 -> ESP8266 [label = "add 1 second to\nstored timestamp"];
	   }

When grace period ends, the esp will transition into sample mode.
This transition features a continous try at getting a timestamp from a predefined NTP server.
The esp will stay in this transition until it successfully receives the timestamp which it then will save in ram.

The timestamp is saved in a resolution of seconds and a timer based on the rtc increments the stored timestamp every second.
In this way we have a guaranteed unix timestamp that holds its time down to the rated precision of the RTC.

Of course we never reach the precision threshold where seconds are dropped since we update the stored
timestamp on a frequency of an hour plus random minutes between 0 and 60. The random minutes guarantee that all meters not ask for NTP time at the same time.

Both the RTC and the integer we store the unix timestamp in are 32 bit.
This means that the RTC can only hold about 1 hour and eleven minutes which affects max sleep time. And the timestamp we increment every second will overflow in 2038, this is not a problem since something better propably has replaced the system by then.

Sample mode
,,,,,,,,,,,

	.. figure::
	   diagrams/sample_mode.png
	   :figwidth: 80%
	   :scale: 200%
	   
.. comment
	.. seqdiag::
	   
	   seqdiag{
	   ESP8266; "kamstrup multical"; "mqtt broker";"mqtt subscriber";
	   ESP8266 -> ESP8266 [label = "create timer for\nsampling with 1\nminute intervals"];
	   ESP8266 -> ESP8266 [label = "initialize uart\nreceiving callback"];
	   === after 1 minute ===
	   ESP8266 -> ESP8266 [label = "craft uart frame"];
	   ESP8266 -> ESP8266 [label = "stop sampling timer"];
	   "mqtt subscriber" -> "mqtt broker" [label = "subscribe"];
	   ESP8266 -> "kamstrup multical" [label = "send frame", failed];
	   ESP8266 -> "kamstrup multical" [label = "send frame"];
	   ESP8266 <- "kamstrup multical" [label = "kmp response"];
	   ESP8266 -> ESP8266 [label = "crc check failed"];
	   ESP8266 -> "kamstrup multical" [label = "send frame"];
	   ESP8266 <- "kamstrup multical" [label = "kmp response"];
	   ESP8266 -> ESP8266 [label = "crc check passed"];
	   ESP8266 -> ESP8266 [label = "encode response\ninto mqtt message\ninclude timestamp\n\nadd message to\nbuffer"];
	   ESP8266 -> "mqtt broker" [label = "transmit buffer"];
	   "mqtt broker" -> "mqtt subscriber" [label = "send all new messages"];
	   }


Sample mode is about sampling from the Kamstrup meter and reporting the samples using MQTT.
A timer runs every minute and sends a kmp request to the Kamstrup meter. This breaks the idea of a request-response protocol
but we have not experienced any Kamstrup meters sending the same sample twice, so the idea of a malformed response is experienced, therefore it was chosen to
sample at a fixed frequency and asynchronously listen for the responses.

When ever a response is received, it is formatted into a string for transmission by MQTT.
It is then stored in the MQTT queue (circular buffer) and MQTT will try to transmit the sample until it either is successfully transmitted or is pushed out the queue
by new samples filling the queue.
It could be tempting to store the raw samples in the queue and decode them when transmitting because they already are compressed but
we need to compute crc on them, and maybe retransmit kmp request, before storing them in the queue.

It is a part of the plan to downscale the sample frequency and introduce a deep sleep break between samples.
This deep sleep period would heavily reduce the power usage of the esp.
The esp sdk comes with a standard way of doing this that implies the use of free user memory in the rtc.
This memory can be used to store a timestamp. If you know beforehand how long you sleep then when you wake you can just append that to the stored timestamp and you have the current time again without ntp.
The esp rtc overflows at 32 bit microseconds which means that you can max sleep ca. 71.58 seconds before having to wake again.
If you want to sleep longer you would have to wake up append sleep period to stored timestamp and then sleep again.
The esp has the ability to define which mode to wake up in so you actually tell it to wake up without wifi which makes power usage even lower with extended sleep.





