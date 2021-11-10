---
title: Secure MQTT Client
number: 8
repo: https://github.com/byu-ecen426-classroom/secure_mqtt_client.git
---

> It's hard enough to find an error in your code when you're looking for it; it's even harder when you've assumed your code is error-free.
> 
> [Steve McConnell](https://en.wikipedia.org/wiki/Steve_McConnell){:target="_blank"}

## GitHub Classroom

1. Use the GitHub Classroom link posted in the Slack channel for the lab to accept the assignment.

2. Use [this template repository]({{ page.repo }}){:target="_blank"} to start the lab.

## Overview

In this lab, you will take your existing MQTT Client from the previous lab and make it secure. There are many ways of making MQTT more secure. The most common way is to [encrypt the communication](https://www.hivemq.com/blog/mqtt-security-fundamentals-tls-ssl/){:target="_blank"} to and from the broker, typically using TLS. However, this requires trusting the broker since the broker must decrypt and then re-encrypt the data when sending it to a different client. The security is not end-to-end. In this lab, we will focus on end-to-end encryption between two clients. We will modify our protocol to allow for a key exchange to derive shared keys using [Diffie-Hellman key exchange](https://en.wikipedia.org/wiki/Diffie–Hellman_key_exchange){:target="_blank"}. Then you will use this shared key to encrypt your request and decrypt the response.

### Protocol

The general flow of the last lab's client will roughly stay the same, but first, you must establish a shared key. To do this, you will publish to `<netid>/key-exchange/request` with your public key (in Diffie-Hellman terms, `A`). The responding client will generate a response on the topic `<netid>/key-exchange/response` with its public key (in Diffie-Hellman terms, `B`). You and the responding node now have enough information to generate a shared key. Once you have generated your shared key, you will use it to encrypt the payload of your action request. For example, if you wanted to uppercase "Networking is the best!" and your NetID is [le0nh4rt](https://en.wikipedia.org/wiki/Squall_Leonhart){:target="_blank"}, then you would publish to the topic `le0nh4rt/uppercase/request` and the message would be the encrypted version of "Networking is the best!". The responding client will decrypt your request, transform the text, encrypt the response, and publish the response.

Below is an overview of all the messages that must be exchanged:

```
            Your Client                          Responding Client
┌────────────────┴───────────────────┐                  │
│             Publish                │                  │
│Topic: <net-id>/key-exchange/request├─────────────────►│
│Payload: public key in PEM format   │                  │
└────────────────┬───────────────────┘                  │
                 │                     ┌────────────────┴────────────────────┐
                 │                     │             Publish                 │
                 │◄────────────────────┤Topic: <net-id>/key-exchange/response│
                 │                     │Payload: public key in PEM format    │
                 │                     └────────────────┬────────────────────┘
  ┌──────────────┴─────────────────┐                    │
  │           Publish              │                    │
  │Topic: <net-id>/<action>/request├───────────────────►│
  │Payload: ...encrypted data...   │                    │
  └──────────────┬─────────────────┘                    │
                 │                       ┌──────────────┴──────────────────┐
                 │                       │           Publish               │
                 │◄──────────────────────┤Topic: <net-id>/<action>/response│
                 │                       |Payload: ...encrypted data...    │
                 │                       └──────────────┬──────────────────┘
```

To be more specific, the Diffie-Hellman parameters (p and g) you must use are provided in [dhparam4096.pem](https://github.com/byu-ecen426-classroom/secure_mqtt_client/blob/main/dhparam4096.pem){:target="_blank"}. The public key sent to the responding client must be in [PEM format](https://www.ssl.com/guide/pem-der-crt-and-cer-x-509-encodings-and-conversions/#ftoc-heading-1){:target="_blank"}. The responding client will respond with it's public key in PEM format. The generated shared key will use Diffie-Hellman to generate the key. The shared key is ran through a key derivation function ([HKDF](https://en.wikipedia.org/wiki/HKDF){:target="_blank"}) to provide [key stretching](https://en.wikipedia.org/wiki/Key_stretching){:target="_blank"}, with the following parameters:

| Parameter         | Value  |
| ----------------- | ------ |
| Length            | 32     |
| Hashing algorithm | SHA256 |
| Salt              | 0x00   |
| Info              | 0x00   |

Encryption and decryption uses the [Fernet](https://github.com/fernet/spec/){:target="_blank"} algorithm. The shared key along with the plaintext or ciphertext are used as inputs into the algorithm. The specification for the algorithm is published [here](https://github.com/fernet/spec/blob/master/Spec.md){:target="_blank"}.

**Note**: If you use the provided Python code, [crypto.py](https://github.com/byu-ecen426-classroom/secure_mqtt_client/blob/main/crypto.py){:target="_blank"}, you don't have to implement any of these details yourself.


### CLI

There are no changes to the CLI from the previous lab. All changes in this lab will be with the underlying protocol.

## Objectives

- Use Diffie-Hellman key exchange

- Learn about end-to-end security

## Requirements

- If you use Python, you must name your program `secure_mqtt_client.py`, and you must provide a [`requirements.txt` file](https://www.idkrtm.com/what-is-the-python-requirements-txt/){:target="_blank"} with all of the dependencies for the lab. If you are using another language, you must provide me with an executable that can run on the Embedded Lab computers named `secure_mqtt_client`.

- The default port must be `1884` (**different from the previous lab**), and the default hostname must be `localhost`.

- Your application must print the response to `stdout`. All other class norms must be followed (e.g., print errors to `stderr`, correct return codes, etc.).

- Your client must subscribe and publish using QoS of 1.

- Your client must use the provided [dhparam4096.pem](https://github.com/byu-ecen426-classroom/secure_mqtt_client/blob/main/dhparam4096.pem){:target="_blank"} file for the Diffie-Hellman parameters.

- To make sure you understand what is going on, there are questions you need to answer in the [README.md](https://github.com/byu-ecen426-classroom/secure_mqtt_client/blob/main/README.md){:target="_blank"} file. You must answer these questions in order to consider your submission complete. You can answer the questions directly in the README.md file of your repo.


## Testing

I have provided an MQTT broker at lundrigan.byu.edu:**1884**. You can use it to test your client. 

You can also install [`mosquitto`](https://mosquitto.org){:target="_blank"}, a popular open-source MQTT broker. Using mosquitto, you can test locally on your machine. You might consider installing `mosquitto-clients`, which gives you [`mosquitto_pub`](https://mosquitto.org/man/mosquitto_pub-1.html){:target="_blank"} and [`mosuquitto_sub`](https://mosquitto.org/man/mosquitto_sub-1.html){:target="_blank"}, which are clients to publish and subscribe with.


## Submission

To submit your code, push it to your Github repository. Tag the commit you want to be graded with a tag named `final`.


## Resources

If you are doing your lab in Python, these resources might be helpful:

- I have provided you with [crypto.py](https://github.com/byu-ecen426-classroom/secure_mqtt_client/blob/main/crypto.py){:target="_blank"} that interacts with the [cryptography](https://cryptography.io/en/latest/){:target="_blank"} Python package.
