![](doc/logo/ccn-lite-logo-712x184.png)

# Status of Automatic Tests

[![Build Status](https://semaphoreci.com/api/v1/cn-unibas/ccn-lite/branches/master/badge.svg)](https://semaphoreci.com/cn-unibas/ccn-lite)

(Currently, CCN-lite V2 supports only automatic build testing, there are no functionality tests)

# Overview

*Jump to the [Table of Contents](#toc)*

CCN-lite is a reduced and lightweight -- yet functionally
interoperable -- implementation of the CCNx and NDN protocols. It covers:

- PARC's [Content Centric Networking Protocol](http://www.ccnx.org/), both the old (v0.8) and new (v1.0) variant,
- the [Named-Data Networking project](http://named-data.net/)

CCN-lite supports multiple platforms, including:

- Linux and OS X user space
- Linux kernel
- Android
- RIOT OS

CCN-lite is meant as a code base for class room work, experimental
extensions and simulation experiments. The ISC license makes it an
excellent starting point for commercial products.

CCN-lite has been included in the RIOT operating system for the
Internet of Things (IoT): http://www.riot-os.org/


<a name="toc"></a>
### Table of Contents
1. [Getting started](#gettingstarted)
2. [Rationale for CCN-lite](#rationale)
3. [Extensions](#extensions)
4. [CCN-lite supported platforms](#platforms)
5. [Command line tools](#lof)
6. [Useful links](#links)
7. [Tentative roadmap](#roadmap)
8. [Changelog](#changelog)
9. [Credits](#credits)



<a name="gettingstarted"></a>
## 1. Getting started

To start right now with CCN-lite visit the [tutorial](tutorial/tutorial.md). It
covers installation on Unix and basic functionality of CCN-lite.

For more information about CCN-lite supported platforms and how to build, see
[CCN-lite supported platforms](#platforms). The source code of CCN-lite is
available on [GitHub](https://github.com/cn-uofbasel/ccn-lite/), either the
`master` branch or `dev-master`. If you found a bug or want to report a problem
with CCN-lite, feel free to
[create an issue](https://github.com/cn-uofbasel/ccn-lite/issues) on GitHub - we
appreciate it!



<a name="rationale"></a>
## 2. Rationale for CCN-lite

The original motivation in 2011 for creating CCN-lite was that PARC's CCNx
router software had grown huge. CCN-lite provides a lean alternative for
educational purposes and as a stepping stone. It's for those who want a simple
piece of software for experimentation or own development, but who do not need
all features of the full thing.

This brings the interesting question of interoperability: What minimum
functionality is necessary in order to participate in a CCNx network? With
CCN-lite, we have made a first attempt at identifying that set, but we are aware
that this set might be incomplete, or may change depending on how CCNx evolves.
We consider this question of "sufficient CCNx functionality" to be of
engineering and academic interest of its own and welcome contributions from
everybody.

Hence, with CCN-lite, we did a race-to-the-bottom and strived for a kind of
"Level-0" interoperability. Level-0, as we understood it, covers:

- ccnb encoding of messages (or any other encoding flavor, including the TLV variants)
- PIT and FIB, basic CCN data structures
- longest and exact prefix matching for basic CCN operations,
  including minsuffixcomp und maxsuffixcomp handling
- matching of publisher's public key to fight cache poisoning
- nonce and/or hop limit tracking to avoid loops as a minimal safeguard

As an interoperability goal we set for ourselves the threshold that the CCN-lite
software must be able to route at the CCN level between fully fledged CCNx
forwarders. Of course, this is a moving target, but we regularly use our tools
to interface with the NDN testbed, for example.

We deliberately do not cover in our CCN-lite code:

- sophisticated data structures for performance optimizations
- exclusion filtering in interests
- all TCP connectivity and the old CCNx cmd line utilities that have TCP
  hardwired into them
- crypto functionality which is not our prime concern (yet)
- repository functionality, SYNC server, ...

A second reason to create CCN-lite was the software license (the original CCNx
project picked GNU, but NDN also has chosen this path) where we prefer a
Berkeley Software Distribution style of "do whatever you want" as we believe
that this will help adopting CCN technology. Therefore, CCN-lite is released
under the ISC license. We have learned that several companies have picked up
CCN-lite for their internal experiments, so we think our license choice was
right.

What you get with CCN-lite is:

- a tiny core and forwarder
- multiple platform support
- little memory usage, runs on IOT platforms with less than 10k ram
- libraries for applications (e.g. packet encoding libraries)
- partially interoperable management protocol implementation
- a simple HTTP server to display the relay's internal configuration
- plus some interesting extensions of our own, see the next section.
- little dependencies: cmake and openssl



<a name="extensions"></a>
## 3. Extensions

In several selected areas we have started our own contributions that are now
part of CCN-lite:

- Experimental *RPC functionality* for letting neighbors mutually invoke
  functions, which could be the starting point both for network management
  functionality and for data marshalling (of interests and data objects) using
  the TLV encoding.

- Clean *packet scheduler support* at chunk level as well as packet or fragment
  level (symbol `USE_SCHEDULER`)

- *Packet fragmentation* and lost packet detection support for running the CCNx
  protocol natively over Ethernet (symbol `USE_FRAG`). This is somehow outdated
  and waits for protocol specs to emerge.

Other features that you can switch on and off at compile time are:

Featue             | Description
:----------------- | :---------------------------------------------------------
`USE_CCNxDIGEST`   | Enable digest component, requires crypto lib.
`USE_CHEMFLOW`     | Experimental scheduler based on chemical networking, source not included.
`USE_DEBUG`        | Basic data structure dumping.
`USE_DEBUG_MALLOC` | Compile with memory armoring.
`USE_DUP_CHECK`    | Check for duplicate nonces.
`USE_ECHO`         | Enable an echo prefix, returning the current time.
`USE_LINKLAYER`    | Talk to Ethernet, W-LAN, 802.15.4 devices, raw frames.
`USE_FRAG`         | Enable fragments, to run CCNx over Ethernet.
`USE_HMAC256`      | Enables hash-based message authentication codes.
`USE_HTTP_STATUS`  | Provide status info for web browsers.
`USE_IPV4`         | Enable IP support.
`USE_LOGGING`      | Enable log messages.
`USE_KITE`         | Routing along the return path, not yet supported.
`USE_MGMT`         | React to CCNx management protocol messages.
`USE_SCHEDULER`    | Rate control at CCNx msg and fragment level.
`USE_SIGNATURES`   | Authenticate management messages.
`USE_STATS`        | Enable statistics.
`USE_SUITE_*`      | Enable a specific protocol: CCNB, NDNTLV (NDN 2014), CCNTLV (CCNx2014), or LOCALRPC and COMPRESSED (for packet compression)
`USE_UNIXSOCKET`   | Add UNIX IPC to the set of interfaces.

The approach for these extensions is that one can tailor a CCN forwarder to
including only those features really necessary.  We have strived to make these
choices as orthogonal as possible and invite you to attempt the same for your
additions.



<a name="platforms"></a>
## 4. CCN-lite supported platforms and how to compile

CCN-lite currently supports five platforms. To find out how to install and use
CCN-lite on each individual platform, refer to the platform-specific readme
files:

 * [Unix](doc/README-unix.md)
 * [Linux kernel](doc/README-kernel.md)
 * [Android](doc/README-android.md)

Additionally, CCN-lite has a pre-built [Dockerfile](Dockerfile) to enable the usage of
[Docker](https://www.docker.com/) with CCN-lite. See the
[Docker-specific readme file](doc/README-docker.md) for more information.

<a name="lof"></a>
## 5. Command line tools

The commandline tools can be found in the ccnl-utils folder
The main command line tools and their corresponding source file that are shipped
with CCN-lite are the following:

Tool                           | Description
:----------------------------- | :---------------------------------------------
`ccn-lite-relay`               | CCN-lite forwarder: user space.
`ccn-lite-lxkernel`            | CCN-lite forwarder: Linux kernel module
`ccn-lite-ctrl`                | Command line program running the CCNx management protocol (over Unix sockets). Used for configuring a running relay either running in user space or as a kernel module.
`ccn-lite-ccnb2xml`            | Simple CCNB packet parser
`ccn-lite-cryptoserver`        | Used by the kernel module to carry out compute intensive crypto operations in user instead of kernel space.
`ccn-lite-fetch  `             | Fetches both a single chunk content or a series of chunks for larger named data. Only the content is returned without any protocol bytes.
`ccn-lite-mkC  `               | Simple content composer, to stdout, without crypto.
`ccn-lite-mkF  `               | Simple tool to split a large file into a fragment series.
`ccn-lite-mkI  `               | Simple interest composer, to stdout.
`ccn-lite-peek`                | Simple interest injector waiting for a content chunk.
`ccn-lite-pktdump`             | Powerful packet dumper for all known packet formats. Output is in hexdump style, XML or content only.
`ccn-lite-produce`             | Creates a series of chunks for data that does not fit into a single PDU.
`ccn-lite-rpc`                 | Send an RPC request and return the reply.
`ccn-lite-valid`               | Demo application for validating a packet's signature.



<a name="links"></a>
## 6. Links:

 - Source code repository:
   https://github.com/cn-uofbasel/ccn-lite
 - CCN-lite web site:
   http://www.ccn-lite.net/
 - CCNx site:
   http://www.ccnx.org/
 - NDN site:
   http://named-data.net/


<a name="changelog"></a>
## 8. Changelog
### [Release 2.0.1 beta](https://github.com/cn-uofbasel/ccn-lite/releases/tag/2.0.1) (Jun 2018)

 - Last NFN Release, further versions will not contain NFN, see [FAQ](https://github.com/cn-uofbasel/ccn-lite/wiki/Frequently-asked-questions)
 - Add: Documentation
 - Bugfixes
 - CVE patches
 - Removed unused IOTTLV and CISTLV Packet formats.

### [Release 2.0.0 beta](https://github.com/cn-uofbasel/ccn-lite/releases/tag/2.0.0) (Nov 2017)

 - Restructuration of the Code to a library system
   Code was split to following libraries:
	- ccnl-core: basic data structures
        - ccnl-fwd:  forwarding functionality of CCN-lite
        - ccnl-pkt:  packet encoding library, can be used for end point application with no forwarding requirements
        - ccnl-riot: RIOT integration library 
 - change build system to cmake
 - add further RIOT support 
        - add compressed packet format in an early stage
 - Fix a lot of Bugs
 - Fix some CVEs (discovered by Eric Sesterhenn / X41 D-Sec):
   CVE-2017-12412, CVE-2017-12464, CVE-2017-12465, CVE-2017-12466, CVE-2017-12467, CVE-2017-12468, CVE-2017-12469, CVE-2017-12471, CVE-2017-12472 

### [Release 0.3.0](https://github.com/cn-uofbasel/ccn-lite/releases/tag/0.3.0) (Jul 2015)

 - Demonstrated interoperability with now-stabilized CCNx1.0, which can run
   side-by-side with NDN in a single CCN-lite relay
 - OMNeT++ integration is back, as it has been requested many times
 - New platforms and transport:
    - Arduino and RFduino
    - Android and Bluetooth Low Energy
 - New functionality: "begin-end" fragmentation for CCNx1.0 and NDN
 - Improved build quality for Ubuntu and OSX
 - Improved READMEs all over the release, easy tutorials
 - Named Function Networking (NFN) over NDN now has Python bindings to make the
   publishing of named functions easier.



<a name="credits"></a>
## 9. Credits
### [Release 2.0.1 beta](https://github.com/cn-uofbasel/ccn-lite/releases/tag/2.0.1) (Jun 2018)

#### Code contributions:
Michael Frey (MSA Safety)     
Cenk G??ndogan (HAW Hamburg)    
Eric Sesterhenn (X41-DSec)    
Peter Kietzmann (HAW Hamburg)    
Martine Lenders (FU Berlin)    
Claudio Marxer (Uni Basel)     
Dima Mansour (Uni Basel)    
Christian Tschudin (Uni Basel)
Christopher Scherb (Uni Basel)    

### [Release 2.0.0 beta](https://github.com/cn-uofbasel/ccn-lite/releases/tag/2.0.0) (Nov 2017)

#### Code contributions:
Christopher Scherb   
Balazs Faludi  
Claudio Marxer  
Cenk G??ndogan  
Eric Sesterhenn  
Christian F. Tschudin  

### [Release 0.3.0](https://github.com/cn-uofbasel/ccn-lite/releases/tag/0.3.0) (Jul 2015)

#### Code contributions:
Lukas Beck  
Christopher Scherb  
Manolis Sifalakis  
Christian F. Tschudin  

#### Feedback and other contributions:
Fateh Al-Mufti  
Frederik Brix  
Patrick Buder  
Daniel Federau  
Ilir Fetai  
Nenad Kokeza  
David Kordsmeier  
Dima Mansour  
Marc Mosko  
Mihai Daniel Rapcea  
Fabian Rauschenbach  
Yanick Salzmann  
Maarten Schenk  
Marco Suter
Marc-Andrea Tarnutzer  
Marco Dieter Vogt  
Akan Yilmaz  


### [Release 0.2.0](https://github.com/cn-uofbasel/ccn-lite/releases/tag/0.2.0) (Dec 2014)

#### Code contributions:
Basil Kohler  
Christian Mehlis  
Massimo Monti  
Christopher Scherb  
Manolis Sifalakis  
Christian F. Tschudin  

#### Code and documentation feedback:
Samuel Bader  
Christoph Betschart  
Nore Derguti  
Ralph Droms  
Wilson A. Eghonghon  
Daniel Federau  
Ralph Gasser  
Cedric Geissmann  
Rasa Liebfried  
Dario Maggi  
Frank M??ller  
Marko Obradovic  
Michaja Pressmar  
Mihai D. Rapcea  
Thomas Simonsen
Alexander Stiemer  
Ziba Tavassoli  
Simon Wang  
Mario Weber  


### [Release 0.1.0](https://github.com/cn-uofbasel/ccn-lite/releases/tag/0.1.0) (Jul 2013)

#### Code contributions:
Stefan Braun  
Pierre Imai  
Basil Kohler  
Thomas Meyer  
Massimo Monti  
Christopher Scherb  
Manolis Sifalakis  
Christian F. Tschudin  
