---
title: NibbleTorrent Peer
number: 9
repo: https://github.com/byu-ecen426-classroom/nibbletorrent-peer.git
---

> Consistency underlies all principles of quality.
> 
> [Frederick P. Brooks, Jr.](https://en.wikipedia.org/wiki/Fred_Brooks){:target="_blank"}


## GitHub Classroom

1. Use the GitHub Classroom link posted in the Slack channel for the lab to accept the assignment.

2. Use [this template repository]({{ page.repo }}){:target="_blank"} to start the lab.

# TODO

- Set up tracker
- Set up peers with only some of the pieces that never have all of the data.

## Overview

You will be implementing a simplified version of a BitTorrent peer, called NibbleTorrent. NibbleTorrent is a peer-to-peer protocol, just like BitTorrent, but without the [Bencoded data](https://en.wikipedia.org/wiki/Bencode) and simplified interactions between peers. You will be downloading files using the following procedure:

1. Get a torrent file and parse it. You can download them here.
   
2. Make a request to the tracker (listed in the torrent file) to get a list of peers that are seeding that file. I am providing the tracker. By making a request to the tracker, you are adding yourself as a peer for that torrent.
   
3. Go through the list of peers requesting chunks of the file. During this time, a peer might connect with you and request a piece of the file as well.

## Protocol

Below outlines the specifics of the data formats and how requests/responses should be formatted to download a file using NibbleTorrent.

### Torrent File

You will need to parse the torrent file to get the necessary tracker information. The torrent file is a **JSON file** with the following keys:

- `tracker_url`: The URL of the tracker. This is where you go to find peers and register yourself as a peer.
  
- `file_size`: The size of the file the torrent file represents.
  
- `piece_size`: A large file is separated into smaller pieces that you can request from peers. This describes the size of each piece of the file, except for the last piece.
  
- `pieces`: This is a list of SHA-1 hashes of each piece of the file. This lets you verify that you received the correct piece and it has not been modified in any way.
  
- `torrent_id`: A unique identifier of the torrent, which is used to request information from the tracker and peers. It is the SHA-1 of the whole file.

Here is an example of a NibbleTorrent file:

```json
{
    "torrent_id": "84e199cad6b953eee0c9b6d8b72612ddb2488999",
    "tracker_url": "http://ecenetworking-server.et.byu.edu:6969/announce",
    "file_size": 8192,
    "file_name": "file.txt",
    "piece_size": 4096,
    "pieces": [
        "0938307ef77f5c8466f97dec40ccae523557257a",
        "01f7b90d66065f6f8bc4e225ed48b23f8766a382",
    ]
}
```

The torrent files you will be working with are located here.

### Tracker

After you have parsed the torrent file, you must make a request to the `tracker_url` to get a list of peers to download the file from. The tracker uses HTTP and parameters are passed using [query strings](https://en.wikipedia.org/wiki/Query_string). You must pass the following parameters:

- `peer_id`: This is an ID that uniquely identifies yourself. It needs to following this format: `-ECEN426-<NetID>-<5 random characters>`. For example, a peer ID would look like `-ECEN426-philipbl-tkv9s`.

- `ip`: The IP address you will want to receive peer connection to. Because the tracker will only live on BYU campus, you will want to use your BYU private IP address.

- `port`: The port you will want to receive peer connections on.

- `torrent_id`: The ID of the torrent you are interested in. You will get this from the `torrent_id` field in the torrent file.

Using the example from above, your request would look like:

```
GET http://ecenetworking-server.et.byu.edu:6969/announce?peer_id=ECEN426-philipbl-tkv9s&ip=10.35.120.175&port=6981&torrent_id=84e199cad6b953eee0c9b6d8b72612ddb2488999 HTTP/1.1
```
When you make this request with a valid peer ID, IP address, port, and torrent ID, the tracker will return a JSON payload with the following keys:

- `interval`: The interval (in seconds) that you must send a request to the tracker in order to be considered a peer.

- `peers`: A list of all peers that are seeding the torrent.

An example of the payload the tracker return:

```json
{
    "interval": 30,
    "peers": [
        "10.35.120.175:6981",
        "10.35.120.88:6977"
    ]
}
```

### Peer Connections

Now that you have a list of peers, you are ready to make a connection with them and start downloading chunks of the file. This is done in three parts:

1. Connect to peer using a TCP socket. You will use the IP address and port number provided in the peers list.
   
2. Send a NibbleTorrent request. This will ensure that the peer is a NibbleTorrent client and it has the file we are looking for. Assuming the peer has the file we are looking for, it will return what parts of the file it has.
   
3. Request pieces of the file from the peer.

You need to be able to support up to 5 concurrent connections with different peers, requesting different parts of of the file. Here is a diagram to roughly outline the algorithm for connecting to peers and requesting pieces of the file:

<figure class="image mx-auto" style="max-width: 500px">
  <img src="{% link assets/nibble_torrent_request_algorithm.png %}" alt="NibbleTorrent Request Algorithm">
  <figcaption style="text-align: center;">Adapted from <a href="https://blog.jse.li/posts/torrent/">here</a>.</figcaption>
</figure>

### NibbleTorrent Request/Response Format

Peers need to communicate with each other to request parts of a file. For this communication, the following general data format will be used:

```
{% sh monodraw assets/nibbleTorrent_request_format.monopic %}
```

- **Version**: This field is 8 bits and will always be `0x01`. This allows clients to support future versions of the protocol where the format might be different.

- **Type**: This field is 8 bits and describes the type of request/response that the peer is making. There are 4 different types: hello request (0x01), hello response (0x02), piece request (0x03), piece response (0x04), and error response (0x05).

- **Length**: The length of the data field in bytes. This field is 16 bits which means the data field is limited to 65,536 bytes.

- **Data**: This field is variable length. **Based on the type of message, you will interpret the data payload differently.**

The peer you right must be able to **send and receive** all NibbleTorrent requests.


#### Hello Request and Response

After you connected to a peer, you will send them a hello request, with the type field value of `0x01`. This request will ensure that you both are using the same protocol and that the peer actually has the file that you are trying to download. The payload of the request is the torrent ID in binary format. For example, a request might look like this (displayed in hex format):

```
01 01 00 14 84 e1 99 ca d6 b9 53 ee e0 c9 b6 d8 b7 26 12 dd b2 48 89 99
```

The peer will respond with a hello response (`0x02`) and list all of the pieces it has of the file, formatted as a bitfield. A bitfield allows a peer to efficiently encode what pieces it already has. A bitfield is like an array of booleans, where the index in the array corresponds to a piece of the file, and the value tells you if they have the piece or not. For example, if a file was broken into 4 pieces and a peer only had piece 1 and 3, then its bitfield would be `b1010`. If it had only the last piece of the file, then the bitfield would be `b0001`. If a bitfield is not an whole number of bytes, then pad the bitfield with zeros.

Using the previous example (torrent ID 84e199cad6b953eee0c9b6d8b72612ddb2488999, which only has two pieces) and assuming the peer has all of the pieces of the file, then the response would look like (in hex format):

```
01 02 00 01 c0
```

#### Piece Request/Response

Once you have connected with a peer, sent a hello request, and received a hello response, you are ready to request pieces. This is done by sending a request with type 0x03 and with a payload of the index of the piece you are requesting. For example, if you wanted to request the third piece of a file, you would send this request:

```
01 03 00 01 02
```

The responding peer will then send the data using the piece response type (0x04), using the piece data as the payload. For example:

```
01 04 10 00 ...piece data...
```

#### Error Response

If a peer sends a malformed request or an error occurs on the responding peer, then the peer will respond with an error response message (0x05). The payload of this message is ASCII text describing what went wrong. For example if a peer requested a piece that was outside of the range of pieces, the responding peer could respond with "Piece index out of range" error message. It would look like:

```
01 05 00 18 50 69 65 63 65 20 69 6e 64 65 78 20 6f 75 74 20 6f 66 20 72 61 6e 67 65
```

`50 69 65 63 65 20 69 6e 64 65 78 20 6f 75 74 20 6f 66 20 72 61 6e 67 65` is hex for "Piece index out of range".


## Objectives

## Requirements

## Testing

## Submission

To submit your code, push it to your Github repository. Tag the commit you want to be graded with a tag named `final`.


## Resources

- https://blog.jse.li/posts/torrent/

- https://en.wikipedia.org/wiki/SHA-1
  - SHA-1 hashes are always 20 bytes long.
  
<!-- 
https://erdgeist.org/arts/software/opentracker/#invocation

https://hub.docker.com/r/lednerb/opentracker-docker/#!

https://github.com/Lednerb/opentracker-docker/

https://stackoverflow.com/questions/4072234/bittorrent-tracker-request-format-of-info-hash

https://wiki.theory.org/BitTorrentSpecification

https://markuseliasson.se/article/bittorrent-in-python/

https://github.com/eliasson/pieces/ -->
