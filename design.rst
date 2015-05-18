Design
======

.. figure::
   images/kamstrup_multical.png 
   :figwidth: 90%

   The Kamstrup Multical heating meter and its internals, including space for expansion boards.

The heart of the system is the Kamstrup Multical 601 meter. This meter is found in households in Denmark and other countries as well.
The meter is normally the property of the company providing the heating infrastructure. Therefore they are mostly
locked to keep end users from mangling and manipulating the device.
This is, however, not the case at Christiania where we have bought the meters and installed them ourself.
This gives us the opportunity to open the meter and install new electronics inside it where Kamsturp left room
for expansion boards like shown underneath:

.. figure::
   images/pcb_comparison.png 
   :figwidth: 90%

   Our ESP8266 PCB, its UART daughter board and in comparison the proprietary Kamstrup WiFi module.
   
Connected to the Kamstrup meter is a microcontroller using a UART connection to talk to the Kamstrup meter.
The microcontroller has onboard WiFi and and external antenna is connected to the device with an extension cable.
Both the microcontroller and the Kamstrup meter gets power from a builtin power supply.

The "meter unit" (Kamstrup meter and WiFi microcontroller) has an established connection to a nearby WiFi hot-spot.
The meter unit transmits samples through the WiFi hot-spot, over the internet to a central server.

The central server collects samples and serves a web page where the samples can be seen in a graph.