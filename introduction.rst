Introduction
============

A vocabulary of technical terms used in this report is found in the appendices.

This report describes a part of the process of digitalizing the heating infrastructure at Christiania, Copenhagen.
The heating infrastructure goes under the name "Nabovarme" and is organized as a collection of wooden pellet stoves heating
groups of households or apartments through heated water and radiators.
Nabovarme currently uses pen and paper to document each customers heating usage by checking the heating meters in each
household once a year.
It was decided that digitalizing the collection of usage data would give better data and data with a much higher frequency.
This higher frequency would first of all make the billing much easier since all data was already existing in the database
when billing time came around. The higher frequency however would also open up a list of opportunities like giving customers
the ability to follow, and react upon, their usage and maybe lead to a lower pellet usage for whole Christiania.
I started this project as a prototype that includes a device that transmits household heating usage to a central
server that will visualize it in html.

Collaboration
-------------

The project has been a collaboration between Johannes Jørgensen and Kristoffer Ek from Christiania Nabovarme.
All source code, PCB layout, debugging and development in general has happened on site at Christiania Nabovarme.
All sources are released through the github accounts belonging to Nabovarme.
See **https://github.com/nabovarme/kamstrup-wifi** for details


Problem formulation
-------------------
Nabovarme wants a device that is able to communicate with the class of heating meters found in all households in Christiania.
The device has to be embedded inside the heating meters and use the same builtin power supply if possible.
The device has to sample at a predefined frequency and sleep between samples if possible, all samples must have been sent to a
central server or at least attempted to be sent once. The device has to use WiFi to transmit samples and an NTP (network time protocol)
client must exist on the device to timestamp all samples.
The device has to be easily configured, with regards to what WiFi hot-spot it will use for transmission, and it has to strive for
end to end encrypted packages so interception is not an issue.
The device has to be cheap and relatively low powered and available in large quantities. A PCB will have to be made for
connecting the device to the heating meter.

Requirements
------------


+-------+------------------------------------------------------------------------------+
| Name  |Description                                                                   |
+=======+==============================================================================+
|R1     |The protocol for serial communication with "Kamstrup Multical 601" has to be  |
|       |reverse engineered and implemented in c.                                      |
+-------+------------------------------------------------------------------------------+
|R2     |The device must use the protocol MQTT to transmit samples to a central server |
+-------+------------------------------------------------------------------------------+
|R3     |The device must use WiFi for data transmission and feature a grace period for |
|       |configuring which access point to join                                         |
+-------+------------------------------------------------------------------------------+
|R4     |The device must use DNS for looking up the ip of the central server and use   |
|       |NTP for getting the unix timestamp it uses in its RTC.                        |
+-------+------------------------------------------------------------------------------+
|R5     |A central server has to receive and save the samples in an SQL database and    |
|       |the same server has to serve a demo web page where the data of a single       |
|       |device can be seen visualized as a graph                                      |
+-------+------------------------------------------------------------------------------+
|R6     |The device has to sleep between samples                                       |
+-------+------------------------------------------------------------------------------+
|R7     |The device has to to transmit all samples at least once if possible (QOS)     |
+-------+------------------------------------------------------------------------------+
|R8     |A PCB should be designed and manufactured for the device to be placed inside  |
|       |the Kamstrup meter                                                            |
+-------+------------------------------------------------------------------------------+