---
layout: post
title: "A Ghost, an Arduino and Go"
description: "Controlling an LED lamp with an arduino and a Go web service"
modified: 2013-09-17
category: articles
tags: [arduino, Go]
---

### Introduction

Going through Newcastle the other month, I out of habit went to visit Forbidden Planet, for comics but mainly for the toys and stuff. 

![Pacman Ghost]({{ site.url }}/images/IMG_20130917_233156.jpg)
{: .image-pull-right}

I came across what is possible one of the best lamps I have seen, I am of course talking about the LED Pacman ghost lamp that can be seen on the right. Now what makes it particularly interesting is that it comes with an infra-red (IR) remote control that can be used to change the colour as well as set a number of effects such as a fading and strobe effects. 

### Checking a few things
After playing around with it a for a little while I thought it would be interesting to see whether the system uses the standard IR frequency. What do I mean by standard, well most TV, music systems and a lot of other devices that can be controlled by IR devices send data over the IR LEDs at a specific frequency, 38kHz to be exact. After purchasing an IR receiver that was able to receive data at this frequency it was time to dig out the Arduino. 

Without knowing the exact format the IR remote control was using to send the data I couldn't be sure of the individual codes or even if it was using the 38kHz frequency. After a bit of googling I found Ken Shirriff's multi-protocol infrared remote library for the arduino [^1]. Using the example code on his website that I've given below, I was able to see that the IR was using a known format and could view the individual hex codes for each of the different buttons on the remote. There are a few different formats that are used to send/receive IR codes, by default the code below will not give the format that's being used. To find this edit the *IRemote.h* file that can found in the IRremote folder where the Arduino library was installed and add the line *#define DEBUG*, just above the line *class decode_results {*.
	
{% highlight c %}	
#include <IRremote.h>

int RECV_PIN = 11;
IRrecv irrecv(RECV_PIN);
decode_results results;

void setup()
{
  Serial.begin(9600);
  irrecv.enableIRIn(); // Start the receiver
}

void loop() {
  if (irrecv.decode(&results)) {
    Serial.println(results.value, HEX);
    irrecv.resume(); // Receive the next value
  }
}
{% endhighlight %}

From running through the code with the DEBUG flag it became apparent that it was using the NEC protocol. Now as a test I copied down one of the codes, 0xF720DF, in this case the HEX value that was sent out when the red colour was chosen and used the code below to test that it would work.

{% highlight c %}
#include <IRremote.h>
IRsend irsend;

void setup()
{
  Serial.begin(9600);
}

void loop() {
  irsend.sendNEC(0xF720DF, 32);
} 
{% endhighlight %}

Lo and behold the lamp turned red.

### Software Design
Having proven that it was possible to control the lamp via IR using the arduino, it was time to design the software to allow the lamp to be controlled via REST web services. I decided to use two main pieces of software, the first running on the Arduino that takes commands via serial regarding the colour and effect to use and then proceed to send these in the format the lamp expects. The second was a set a of RESTful web services written in go that receives the commands, interprets them and then sends them over serial to the arduino. 

#### Arduino
The arduino code is relatively simple, there's just a continuous loop that reads a byte from the serial. This byte is used as a command byte that controls the type of effect that we want to show. A switch case then just parses through the command bytes,
at the moment there are four types of commands that can be used. *Constant*, *flash*, *alt* and finally *off*. 

For the constant command the code waits for a second byte that defines the colour to use. This is then displayed on the lamp until another command is received. The flash again just takes a single byte as the colour, and alternates between showing the colour given and blank. In order to produce accurate timing I'm using this [timer](https://github.com/JChristensen/Timer) library. The alt command produces an effect similar to the alt command except that it takes two colours, and then alternatively flashes between two two colours. Off does what you might expect and switches the lamp off. The code for the arduino side can be found in its entirety [here](https://gist.github.com/biomood/7493332).

#### RESTful web services
The go side is split into two parts, the first part handles the web services and the second part handles communication with the arduino via serial.

In order to create the RESTful web services I'm using the [Go-Json-Rest](https://github.com/ant0ine/go-json-rest) library. An example of setting one of the handlers is below:
{% highlight go %}
handler := rest.ResourceHandler{}
handler.SetRoutes(
	rest.Route{"POST", "/ghost", PostCmd},
)

http.ListenAndServe(":8080", &handler)
{% endhighlight %}
The code for the *PostCmd* is then below, in a nutshell the function decodes the REST payload into JSON and then parses through the command contained in the JSON. The appropriate function is then called in the underlying *ghostlib* library. 
{% highlight go %}
func PostCmd(w *rest.ResponseWriter, r *rest.Request) {
	cmd := Command{}
	err := r.DecodeJsonPayload(&cmd)
	if err != nil {
		rest.Error(w, err.Error(), http.StatusBadRequest)
		return
	}
	err = nil

	switch cmd.Cmd {
	case "CONST":
		if cmd.Colour == "" {
			rest.Error(w, "missing parameter", 400)
		} else {
			err = ghostlib.ConstantColour(cmd.Colour)
		}
	case "FLASH":
		if cmd.Colour == "" {
			rest.Error(w, "missing parameter", 400)
		} else {
			err = ghostlib.FlashColour(cmd.Colour)
		}
	case "ALT":
		if cmd.Colour == "" || cmd.ColourAlt == "" {
			rest.Error(w, "missing parameters", 400)
		} else {
			err = ghostlib.AlternateColour(cmd.Colour, cmd.ColourAlt)
		}
	case "ON":
		err = ghostlib.On()
	case "OFF":
		err = ghostlib.Off()
	default:
		rest.Error(w, "unknown command", 400)
	}

	if err != nil {
		rest.Error(w, "Server Error", 500)
	}
}
{% endhighlight %}

Data is sent to the arduino via serial and for this I'm using the goserial library available [here](https://github.com/tarm/goserial). To open a connection to the arduino via serial using this library first we must set the string of the connection to make and speed, open the port and then when we're ready to write to the port just call Write with the bytes to send.
{% highlight go %}
c := &serial.Config{Name: "/dev/tty.usbserial-A600agDn", Baud: 9600}
port, err := serial.OpenPort(c)
if err != nil {
	log.Fatal(error)
}

vals := make([]byte, 1)
vals[0] = cmds["OFF"]
port.Write(vals)
{% endhighlight %}

That pretty much covers how to send data via serial to the arduino, the full code for go RESTful web services can be found [here](https://dl.dropboxusercontent.com/u/1995939/ArduinoIR.zip).

This covers part 1 of using web services to control the colour of a lamp with an arduino. In a next post I'll go over the android application I've developed to remotely control the lamp as well as flash when a text message or phone call is made.

[^1]:
	Available at: http://www.righto.com/2009/08/multi-protocol-infrared-remote-library.html

