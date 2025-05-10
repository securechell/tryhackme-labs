# Day 24: You can’t hurt SOC-mas, Mayor Malware!

**Learning Objectives**

In this task, you will learn about:
- The basics of the MQTT protocol
- How to use Wireshark to analyze MQTT traffic
- Reverse engineering a simple network protocol

### How Smart is Smart
Smart devices make our lives very easy. We no longer physically need to move and turn on or off a switch to control them. With smart HVAC (heating, ventilation, and air conditioning) systems, we can maintain the temperature of our homes and ensure they are not too cold or too hot when we come home from outside. Smart vacuum cleaners can clean our house while we work on other things or go out for dinner. Many smart devices come with apps that allow us to control them using our mobile phones. Even better, since these devices can be controlled remotely through apps and interfaces connected to the Internet, we can make their designs more minimalistic and aesthetically independent, and the need for adding switches or controls on the device itself is minimised.

### Is It Smart
While they make our lives easier, most smart devices need a network connection to provide control to the users. Many smart devices are connected over the Internet ("Internet of Things"), which means that anyone can potentially take control of them. We can limit the exposure of these devices by adding security controls such as network isolation and authentication mechanisms. However, the most secure system is a system that is shut down, but that does not deter us from using different systems to help us out in our daily lives, and the same should be the case with smart devices. Instead, we can ensure that we understand how our smart devices work and have adequate security set up for them.

### The Language of IoT
Although different IoT and smart devices use various programming languages, depending on the platform and vendor, they often need to speak the same language to be able to communicate with each other. For example, while IoT devices might use C++ or Java to talk to the compiler and the underlying hardware, they will need a language like HTTP or MQTT to talk with your system or mobile device.

### How to Speak MQTT
**Message Queuing Telemetry Transport (MQTT)** is a language very commonly used in IoT devices for communication purposes. It works on a publish/subscribe model, where any client device can publish messages, and other client devices can subscribe to the messages if they are related to a topic of interest. An MQTT broker connects the different clients, publishing and subscribing to messages.

### Demonstration
Let’s see how MQTT works in practice. The files related to this task are in the `~/Desktop/MQTTSIM/` directory.

As we discussed, different IoT devices communicate with each other using MQTT, a network protocol. Therefore, we must monitor network communication to see what is going on. We will use Wireshark to monitor network communication. On the left side, we can click the Wireshark logo to start Wireshark. On the home screen, we can select ‘any’ for the network interface to see traffic from all the network interfaces.

![image](https://github.com/user-attachments/assets/12dedd92-8a0a-41ae-976f-1af56e3d967a)

Because we've selected 'any' for the network interface, we're going to see the traffic from all the interfaces on the network. Since we only want to see traffic from the MQTT protocol, we can filter for this protocol only. To do that, we can type `mqtt` in the filter box and press enter.

![image](https://github.com/user-attachments/assets/f2d4c42b-6f07-41fd-877f-168e5c841c7a)

There's no traffic for now as no MQTT broker or client is running. Let’s start the MQTT broker and client to see the traffic. For that, there is a shortcut for a directory on the Desktop named **Link to MQTT**. The directory has different scripts in it. We will start the `walkthrough.sh` script to learn how the MQTT protocol works. 

Let’s run the `walkthrough.sh` script.

![image](https://github.com/user-attachments/assets/8c8688a2-7456-4733-98f8-c8be618c2f0b)

Once we run it, three windows will pop up. The window with the red text is the MQTT broker, the one with the blue text is the MQTT client, and the third is the application UI we might see as the end user. We can look at the logs on each of these windows if we like, but as an end user, we will only interact with the application UI.

![image](https://github.com/user-attachments/assets/30f686f2-0720-443d-a893-5a48ebc09b91)

Currently, the application is in automatic mode, the heater is off, the target temperature is 22 degrees, and the current temperature is 22.3 degrees. Let’s head over to Wireshark to see what kind of communication and see how the communication started.

![image](https://github.com/user-attachments/assets/d285ee72-2f7d-4399-a3e4-567349ef7f30)

The first two events show the connection establishment, showing the `Connect Command` and `Connect Ack` text in the info. The next two events show that one of the clients wants to subscribe to the `home/temperature` topic, shown in the square brackets. This must be the HVAC controller, which needs to know the temperature to turn the heater on or off. We can see the messages published by different clients, with their topics in the square brackets. The events from the `home/temperature` topic are from the temperature sensor, and the events from the `home/heater` are published by the HVAC controller. The screenshot below shows the details of the `home/temperature` topic message.

![image](https://github.com/user-attachments/assets/a8c1697d-f41f-4677-964f-4cd4fae3224f)

In the lower pane, we can see that it is publishing a temperature of -9.6877475608946. That is very cold. Therefore, we see a message from the heater right after the temperature is broadcast. This message shows that the heater was turned on, as seen in the highlighted part in the lower pane.

![image](https://github.com/user-attachments/assets/2979bf03-9e52-4ba6-9a1a-2ea6f8d2a923)

### Challenge
Great! Now we understand how the MQTT publish/subscribe model works with a broker. McSkidy can use our help; the lights have gone off in all the major factories and halls. Mayor Malware has sabotaged the lighting system implemented by a contractor company. Their support will take some time to respond, as it is the holiday season. But McSkidy needs to get the lights back on now!

To see what Mayor Malware has done, we can run the script `challenge.sh` as shown below. This will open three windows, including the lights controller interface.

![image](https://github.com/user-attachments/assets/38e3ac20-b0df-490a-accf-8e8ae298a67a)

We can see the lights controller interface; however, nothing works, and there is no way to turn the lights back on.

![image](https://github.com/user-attachments/assets/c315aee6-a3a8-4821-8a4a-b7510038c280)

Now that we have basic knowledge of the MQTT protocol, we must find the command to turn the lights back on. The `challenge.pcapng` file is inside the `challenge` directory. This packet capture file contains various MQTT protocol messages related to turning the lights “on” and “off.” Start Wireshark then go to **File** -> **Open**, locate the `challenge.pcapng` file, and open it in Wireshark to take a closer look.  Type `mqtt` to narrow the results.

You plan to publish a message to the MQTT broker in order to turn on the lights. As a result, all the lighting devices subscribed to this MQTT broker will turn their lights on. You can achieve this from the command line using the `mosquitto_pub` command. Consider the following example:

`mosquitto_pub -h localhost -t "some_topic" -m "message"`

We are publishing a message under a specific topic to a broker.

- `mosquitto_pub` is the command-line utility to publish an MQTT message
- `-h localhost` refers to the MQTT broker, which is `localhost` in this task
- `-t "some_topic"` specifies the **topic**
- `-m "message"` sets the **message**, such as `"on"` and `"off"`

To determine the correct topic and message to turn on the lights, you must study the captured packets in `challenge.pcapng`.

Once you figure it out, ensure you are running `challenge.sh` and use `mosquitto_pub` to publish a message to turn on the lights. Once you successfully manage to turn on the lights, a flag will appear on the lights controller interface.

In exploring the captured packets, we can see a message `6f6e` that seems to code the word `"on"`.

![image](https://github.com/user-attachments/assets/b86be695-0dcd-4765-8fdd-d44bd955497d)

This means our topic is: `d2FyZXZpbGxl/Y2hyaXN0bWFzbGlnaHRz`

Now in the terminal type:
`mosquitto_pub -h localhost -t "d2FyZXZpbGxl/Y2hyaXN0bWFzbGlnaHRz" -m "on"`

Return to the lights controller interface. We see the lights are now on and we have the flag.

![image](https://github.com/user-attachments/assets/e9d1bd46-02e3-4b68-ba0d-c8004d796a47)

### Answers
1. What is the flag? (Hint: Focus on the Publish Message packets.) **THM{Ligh75on-day54ved}**.
