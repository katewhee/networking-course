---
title: Chat Client
number: 9
repo: https://github.com/byu-ecen426-classroom/chat_client.git
---

> Consistency underlies all principles of quality.
> 
> [Frederick P. Brooks, Jr.](https://en.wikipedia.org/wiki/Fred_Brooks){:target="_blank"}

## GitHub Classroom

1. Use the GitHub Classroom link posted in the Slack channel for the lab to accept the assignment.

2. Use [this template repository]({{ page.repo }}){:target="_blank"} to start the lab.

## Overview

For this lab, you will be writing a chat client using [MQTT](https://mqtt.org){:target="_blank"}. Your chat client will need to be able to chat with other students through the MQTT broker. There are two parts to this lab, the "protocol" (how to format the payload) and the user interface.


### Protocol

Like the previous lab, you will be using MQTT as the main protocol. However, for this to work with other students, we need to standardize some way of formatting the MQTT message's payload and topic. There are two different topic sets: `chat/<netid>/message` where all messages get published and `chat/<netid>/status` where all status updates get published.

Your chat client will subscribe to two topics: `chat/+/message` and `chat/+/status`. These topics will allow you to receive all messages that are published from any NetID (including your own...). When you want to send a message, and assuming your NetID is "le0nh4rt", your client will publish to `chat/le0nh4rt/message`, and when you want to update your status, you will publish to `chat/le0nh4rt/status`.

The payload for the `chat/<netid>/message` publications must contain [`json`](https://www.json.org/json-en.html){:target="_blank"} data with the following keys:

- `timestamp`: The time the message was published, formatted as the [epoch time (or Unix time)](https://en.wikipedia.org/wiki/Unix_time){:target="_blank"}.
- `name`: The name of the person sending the message. This is optionally provided as an argument to your interface. If no name is provided, you must use your NetID.
- `message`: The message you are sending.

For example, if you are publishing a message, "Hello world" at 8:13:00 AM on Nov 24, 2020, then the payload would be the following:

```json
{"timestamp": 1606230780, "name": "Dr. Phil", "message": "Hello world"}
```

The payload for the `chat/<netid>/status` publications must contain `json` data with the following keys:

- `name`: The name of the person sending the message.
- `online`: An integer showing if the person is online or not. 0 for offline and any other value for online.

For example, to update your status to "online", the payload of the status update would be:

```json
{"name": "Dr. Phil", "online": 1}
```

You must publish an "online" message when you start your client and register a [last will](https://mntolia.com/mqtt-last-will-testament-explained-with-examples/){:target="_blank"} with the broker that publishes an "offline" message when you disconnect. To make the status messages more useful, you must set the retain flag for all status message publications.

### User Interface

This interface will be different from any other lab. Rather than using a command-line interface, you will be building a graphical interface. A user must be able to provide the following information to your application:

- NetID: the NetID of the person using your application
- Name: the human readable name of the person using your application. This is optional and if not provided, you can use the NetID as the name.
- Hostname: The hostname of the broker. The default should be `localhost`.
- Port: The port of the broker. The default should be 1885 or 1886 for WebSockets.

You can choose to accept these arguments/options through the command line or through the graphical interface.

## Objectives

- Build something that interfaces with other people's code.

- Get exposure to integrating with UI code or writing your own UI code.

- Have fun!


## Requirements

- Your program must be able to accept all arguments and options as outlined above.

- The default port must be `1885` (or `1886` for [WebSockets](https://github.com/mqttjs/MQTT.js){:target="_blank"}), and the default hostname must be `localhost`.

- Your client must be able to work with other chat clients.

- Your client must subscribe and publish using QoS of 1.

- Your client must publish an "online" status message when first connecting to the broker and register a last will to publish an "offline" status for when your client disconnects.

- Your status messages must set the retain flag to `true`.

- To support multiple instances of your chat client, you must add randomness to the client ID you use to connect to the broker. You are free to choose how you want to add randomness to your client ID. 

- Your user interface must show the following information:
  - Messages that have been sent and received. It must include the time the message was sent, the name of the person that sent the message, and the message itself. The timestamp must be human-readable and not in epoch time format.
  - Show when people have left or joined the chat server. This can be inline with the chat messages.

- For extra credit, you can add the following or improve on the previous requirements:
  - +1 points: Mark that the message was delivered.
  - +1 points: Show who is online/offline. Instead of putting these messages inline with the chats, it should be off to the side with lists of names and an indication of who is offline/online. For example, you could have a list of names and green circles next to people that are online and red circles next to people who are offline.
  - +1 points: Automatically reconnect to the chat server if you get disconnected and have some kind of status indicator showing if you are online or offline.

## Testing

The chat server will be hosted at lundrigan.byu.edu:1885 (or lundrigan.byu.edu:1886 for WebSockets).

One good way of testing your chat client is to bring up multiple instances of your client. That way, you can see how it responds to people coming online and offline.

## Submission

To submit your code, push it to your Github repository. Tag the commit you want to be graded with a tag named `final`. For more complicated applications, you will have to demo the app for me before classes end.


## Resources

- 🤷‍♂️
