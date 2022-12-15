---
title: "How Torrents Work"
layout: post
date: 2018-08-20 08:31:19 -0700
---


Most of us must have used Torrents at some point of our lives. It's an amazing utility that allows one to download large files with relative ease.
Often, the technology behind Torrents, gets overshadowed by the constant inquisition about the morality of it's usage.

Only on delving deeper into the technology behind it, does one get to truly appreciate it's brilliance. Without further ado, let's jump right into it.

Let's start by understanding what a _p2p_ application is.


## A Brief about p2p networking

It's an alternative to traditional Client Server architecture, wherein each member in the network is equal (more or less) and contributes in delivering the service the network is intended to provide.
This implies that the role of a central server is nearly redundant.
Such a system brings in some noteworthy advantages over the traditional approach -
* It improves availability by removing single point failures (though admittedly a well distributed server client system can do the same)
* There is no money spent on making data warehouses, large servers or establishing connections with monumental bandwidths. It distributes the storage, compute and bandwidth required across all the users on the network.

It builds a self reliant ecosystem where everyone contributes and benefits.
An application which uses such an approach to deliver the service is called a p2p application. Torrents happen to be amongst the mostly widely used p2p applications.
Following this, let's introduce ourselves to the BitTorrent (protocol): The primary Protocol used by torrents for the p2p transfer of files.


## BitTorrent Protocol

### Terminology
1. __Client__ : The program that enables p2p file sharing via the BitTorrent protocol.

2. __Peer__   : A peer is one instance of a BitTorrent client running on a computer on the Internet to which other clients connect and transfer data.

3. __Seeder__ : A peer that has the complete file and is now acting as a source for others. Having high numbers of seeders is huge positive as it's a measure of availability.

4. __Leecher__ : They're peers who are in the process of downloading the file. They have parts of the file and keep gathering more parts over time. Once they finish their download, they become Seeders.
They also act as sources of the parts of the files they've downloaded.


### Format of the File
The file is divided into pieces (also referred to as fragments), which are further split into sub-pieces. Clients can download a sub-piece from only one peer at a time. Therefore, to parallelise the download process, multiple sub-pieces are downloaded simultaneously from different peers.


### Tracker
A tracker plays a very important role in the BitTorrent ecosystem. The tracker maintains information about all clients utilising each torrent and the meta-data about the torrent. Specifically, the tracker identifies the network location of each client either uploading or downloading the files associated with a torrent. It also tracks which fragment of that file each client possesses, to assist in efficient data sharing between clients.
It's essential to note that the tracker does not hold a copy of the file. It merely contains the meta data about the peers having or building copies of the file and information about the fragments that make up the complete file. So when a client approaches the Tracker to download the file, it simply reverts with the information about the peers involved in the file transfer and info about the file itself.


### Initial Upload
When a new torrent is released, it's seeded by a single peer, the peer releasing it. The peer registers the torrent with the Tracker (or with any of the Trackers in case of multi-tracker network). A torrent file is generated for the new release containing the address of the Tracker.
Initially the original seeder is the only peer having a copy of the file. As peers start to download the file, the numbers of seeders and leechers increase, thus increasing the available copies of the file.


### Downloading Requests
When a peer starts downloading a particular torrent, it approaches a tracker associated with it. The Tracker relays meta-data about the torrent and the peers involved in the download. Using the algorithms mentioned later, the peer pings other peers and downloads the various pieces of the file. The actual sub-piece download is done using TCP protocol at network layer. To download the sub-pieces, the peer makes requests to other peers. These request are queued in a pipeline. It's imperative to maintain the pipeline as BitTorrent uses TCP which implements a slow start mechanism. It is thus crucial to always transfer data or else the transfer rate will drop.


### File Sharing Algorithm Used by Peers.
In a p2p network, the Peers themselves decide whom to download the pieces from. They follow the BitTorrent Protocol rules. These rules are essential to facilitate proper functioning of the network.

Following are some of the rules that Clients follow while downloading-

#### Piece Selection Algorithm
This is how the client decide which piece to download next. Piece selection is done to achieve a primary goal- Replication of different pieces on different peers as soon as possible. This will
increase the download speed, and also make sure that all pieces of a file is somewhere in the
network if the seeder leaves.
While downloading the peers use the following rules to maximise they efficiency-

* __Strict Policy__ : It prioritises of completion of a single piece over partial download of multiple pieces. Once a sub-piece is requested by the peer, other sub-pieces of the same piece are requested to be downloaded before others.

* __Rarest First__ : This aligns directly with the goal, wherein the next piece to be downloaded is the one which has fewest replicas. Not only does increase the availability but it also increases the download speeds for other peers looking for the piece.

* __Random First Piece__ : Since Bit-torrent protocol requires a peer to balance it's upload and download, it's wise to download the first piece as soon as possible. In coherence with this idea, the first piece is downloaded randomly instead to following the rarest first policy. Once the first piece is downloaded, the peer commences with the rarest first policy.

* __Endgame Mode__ : Towards the end of the download, when all the only sub-pieces remaining are the ones in the pipeline, the request is broadcasted to all peers. On the arrival of a sub-piece, a corresponding cancel request is sent. The network congestion caused by this is less than expected because of the it's short duration.


#### Resource Allocation

The resource allocation of peers is managed by the peer themselves. Therefore the peers use protocols that award other peers who have been contributing and snubs those who haven't been uploading themselves. This forces all peers to upload in order to maintain good download speeds.
The aim for every node is to establish bidirectional channels with other peers, where both download and upload nearly the same amount to one another. This builds a stable p2p connection between the two nodes. To attain this goal, every peer uses the following strategies-

* The peer cooperates on the first request made by any node. This is to initiate the communication between them.

* On the successive requests, the peers cooperation commensurates with cooperation received from the other peer.

* __Choking Algorithm__ : Peers always maintain a fixed number of unchoked peers. These are the peers that they are willing to upload to. These peers are selected on the upload rate offered by them. This provides an incentive to other peers to be good uploaders. If the unchoked peers are changed too often, the download speeds fall because of the slow start mechanism of TCP transfer. Therefore the list of unchoked peers is updated after a duration of 20 seconds.

* __Optimistic Unchoking__ : Peers select a random peer that they unchoke irrespective of the upload speed it offers. This peer is used as a wildcard entry to replace the peers selected by the choking algorithm. This allows the peer to discover new nodes which might offer better upload speeds. This peer is changed every 30 seconds. If in the duration, this peers is able to offer better upload speed than any of the other unchoked peers, they are swapped.

* __Anti-Snubbing__ : When a peer stops receiving data from a peer for about 60 seconds, it assumes that it has snubbed by that particular peer. To replace it, the node increases the number of Optimistically Unchoked peers. This allows the peer to recover faster in a scenario where it is snubbed by a large at the same time.

* __Upload Only__ : After a peer finishes a download, it uploads to peers having the highest download rate. This allows quick replication of data in the network.

These policies ensure that the pieces are replicated, and that every peer has the largest
probability of retrieving the complete file, as quickly as possible.
Inspired by “tit-for-tat”, the choking algorithm tries to prohibit “free riders” from destroying
the dynamics of the peer-to-peer network. If a peer blocks other peers from uploading, he will
soon be choked by them and thus affecting his download rate. The choking algorithm
correlates the download rate to the upload rate. Uploading is encouraged and the price in
return is a better chance for faster download


The Bit-torrent protocol comes into play once the Client makes contact the Tracker. I'm sure you're wondering how exactly does it manage to reach the tracker?


## Getting to the Tracker
Originally there was a single tracker for each torrent. This would be a special node entrusted to track the fragments of the torrent and the peers involved in the download.
The location to this Tracker would be in the Torrent file. The torrent file would be listed on multiple web portals. Peers use these torrent file to get to the Tracker which in turns provides them the details to commence downloading the Torrent fragments.
It's evident what the drawbacks of such a system are. The single Tracker represents a single point failure and makes the system amenable to Denial Of Service Attacks.
To fix this Multi-Tracker Torrents were introduced.


### Multi-Tracker Torrents
To tackle the aforementioned drawbacks of a single tracker, Multi-Tracker Torrents were devised. In this approach, every node in the network is equivalent. Trackers weren't special nodes anymore. [Distributed Hash Tables (DHTs)](https://github.com/hydra-hoard/hydra/wiki/Distributed-Hash-Table-(DHT)-Algorithm) were used to find and reach the Tracker to any Torrent. The torrent file now only contained the Hash of the Torrent, which would allow Peers to reach the Tracker using DHTs.
Consistency amongst the Trackers is a different story in itself.
