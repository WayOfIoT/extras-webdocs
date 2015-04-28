# AllJoyn&trade; System Description

## Release history

| Release version | Date | What changed |
|---|---|---|
| 14.06 | 9/26/2014 | Initial release |
| 14.12 | 12/17/2014 | <p>Updates for new functionality added in 14.12 release:</p><ul><li>UDP Transport design</li><li>TCP vs UDP Transport selection logic at the router</li><li>mDNS-based discovery for the router at the TCL</li><li>Updates to SLS fetch backoff design to support linear+exponential backoff</li><li>Router Probing mechanism to detect missing applications</li><li>Router logic to detect and disconnect slow reader nodes</li></ul><p>Other updates:</p><ul><li>Endpoints usage by AllJoyn Transport</li><li>TCP Transport data plane architecture and state machine</li><li>AllJoyn Protocol Version mapping for different releases</li><li>Link timeout mechanism between routers to detect missing routers</li></ul>|
| 15.04 | 4/29/2015 | <p>Updates In Thin Apps section related to:</p><ul><li>Security and adding description of Router Selection</li><li>Some general clean up including fixing typos and readability and consistency changes</li></ul><p>Other changes:</p><ul><li>General cleanup</li><li>Removed references to RSA and PIN authe mechanisms as they are not longer supported</li>|

This section describes in detail how AllJoyn works at the system level.

## System Overview

### IoE overview

The Internet of Everything (IoE) is an exciting vision which 
promises to connect people with things and things with each 
other in ways; this will create new capabilities and richer 
experiences and overall make our life simpler. IoE promises 
to bring people, process, data, and things together to make 
networked connections more relevant and valuable than ever 
before, turning information into actions and enabling capabilities 
never possible before. 

IoE will result in smart things or devices at homes, offices, 
cars, streets, airports, malls etc. and these devices will work 
together to provide contextual and real time experiences to users. 
The IoE devices nearby each other will form proximal IoE networks, 
e.g., at home, in the car, or in one's office. The IoE vision will 
enable internetworking among multiple proximal IoE networks.

It is worthwhile to draw a comparison between Internet today and 
Internet of Everything. The Internet today consists of millions of 
registered top-level domains names centrally managed by the Internet 
Assigned Numbers Authority (IANA). Discovery of these domains happen 
through a hierarchical lookup via the Domain Name System (DNS). 
In an IoE network, there will be potentially tens of billions of 
IoE devices. It would be impossible to manage registration for 
all of these devices via a central entity from a scalability 
perspective. Also, in an IoE network, proximity-based interactions 
among devices reduces latency as well as the need for each device 
to connect directly to the Internet. So, the discovery should happen 
automatically based on proximity criteria.  Security and privacy 
issues become even more important in an IoE network where more 
and more personal and home devices expose interfaces for connection 
and control.

The following figure shows an example IoE network with multiple 
proximal IoE networks interconnected with each other via the Internet. 

![ioe-network-example][ioe-network-example]

**Figure:** IoE network example

Smart devices within each IoE proximal network can dynamically 
discover and communicate with each other over direct peer-to-peer 
connection. For cases where some of these devices are behind NAT, 
they can discover each other via some cloud-based discovery 
services. The cloud-based discovery can also be used for 
discovery and connection among IoE devices in different IoE proximal 
networks. The overall IoE network can have additional cloud-based 
services to provide specific functionality, e.g., remote home 
automation, remote diagnostics/maintenance, data collection/reporting, etc. 
The IoE network can also integrate with some of the existing cloud-based 
services, e.g., integration with Facebook or Twitter for device status updates.

In any IoE network, interoperability among devices within and across 
IoE proximal networks is of utmost importance to enable a rich, 
scalable IoE ecosystem of applications and services for these devices. 
Any IoE system must consider specific key design aspects including 
device advertisement and discovery, mobility and dynamic IoE network 
management, security and privacy, interoperability across multiple 
bearers/OS, lightweight solution to support thin/dumb device, 
extensibility and overall scalability. For an IoE system to be 
truly adapted and successful, it must be open and provide a 
horizontal solution that can be used across different vertical 
use cases. 

The AllJoyn system aims to address these key design aspects. 
It provides an open-source software framework to enable proximity-based, 
peer-to-peer, bearer-agnostic networking among IoE devices. 
The AllJoyn system provides a way for devices and applications 
to advertise and discover each other using peer-to-peer protocol 
within a proximal network.

The AllJoyn open-source software system provides a framework for 
enabling communication among IoE devices across heterogeneous 
distributed systems. The AllJoyn system is a proximity-based, 
peer-to-peer communication platform for devices in a distributed 
system. It does not require a centralized server for communication 
across such devices. AllJoyn-enabled devices run one or more AllJoyn 
applications and form a peer-to-peer AllJoyn network. The AllJoyn 
system is a distributed software platform which enables applications 
running on IoE devices to advertise, discover and connect to each 
other for making use of services offered on these devices. 
The AllJoyn framework enables these applications to expose 
their functionality over the network via discoverable APIs 
which are the contracts that define the functionality provided 
by the application.  

In the proximal AllJoyn network, AllJoyn applications installed 
on IoE devices are peers to each other. An AllJoyn-enabled 
application can play the role of a provider, a consumer or 
both depending upon the service model. Provider applications 
implement services and advertise them over the AllJoyn network. 
Consumer applications interested in these services discover 
them via the AllJoyn network. Consumer applications then 
connect to provider applications to make use of these services 
as desired. An AllJoyn application can act as both provider and 
consumer at the same time. This means that the app can advertise 
a certain set of services it supports, and can also discover 
and make use of services provided by other apps in the 
proximal AllJoyn network.

The following figure shows an AllJoyn network with 4 devices. 

![alljoyn-network][alljoyn-network]

**Figure:** AllJoyn network

Device 1 and Device 2 have only Provider applications providing 
AllJoyn services. Device 3 has only consumer applications consuming 
services from other provider devices. Device 4 has an application 
that acts as both provider and consumer. The application on 
Device 4 consumes services from the application on Device 2. 
It also provides services which get consumed by applications 
on Device 3. Arrow directions are from provider to consumer 
indicating consumption of services.

The AllJoyn framework establishes an underlying bus architecture 
for communication among IoE devices. AllJoyn applications on 
IoE devices connect and communicate to each other via the 
AllJoyn Bus. The AllJoyn bus provides a framework for applications 
to expose their services to other AllJoyn applications. The AllJoyn 
bus provides a platform- and radio-link agnostic transport mechanism 
for applications on IoE devices to send notifications or exchange 
data. The AllJoyn bus takes care of adapting to an underlying 
physical network-specific transport. 

Each AllJoyn app connects to a local AllJoyn bus. One or more 
applications can connect to a given local AllJoyn bus. AllJoyn 
bus enables attached AllJoyn applications to advertise, discover, 
and communicate with other. AllJoyn buses on multiple devices 
communicate with each other using underlying network technology 
such as Wi-Fi.

The AllJoyn framework's open-source implementation provides an 
ecosystem where various parties can contribute by adding new 
features and enhancements to the AllJoyn system. It supports 
OS independence via an OS abstraction layer allowing the AllJoyn 
framework and its applications to run on multiple OS platforms. 
The AllJoyn framework supports most standard Linux distributions, 
Android 2.3 and later, common versions of Microsoft Windows OS, 
Apple iOS, Mac OS X and embedded OSs such as OpenWRT and 
RTOSs like ThreadX.

The AllJoyn framework also supports multiple programming 
languages for writing applications and services for IoE 
devices, which enable a wide ecosystem for developing AllJoyn 
applications and services. The AllJoyn framework currently, 
supports C, C++, Java, C#, JavaScript, and Objective-C.

### The AllJoyn system and D-bus specification

The AllJoyn system implements a largely compatible version 
of the D-Bus over-the-wire protocol and conforms to many of 
the naming conventions and guidelines in the D-Bus specification. 
The AllJoyn system has extended and significantly enhanced D-Bus 
message bus to support a distributed bus scenario. The AllJoyn 
system makes use of the D-Bus specification as follows: 

* It uses the D-Bus data type system and D-Bus marshaling format. 
* It implements an enhanced version of the D-Bus over-the-wire 
protocol by adding new flags and headers (detailed in [Message 
format][message-format]). 
* It uses D-Bus naming guidelines for naming well-known names 
(Service names), interface names, interface member names 
(methods, signals and properties) and object path names.
* It uses a D-Bus defined Simple Authentication and Security 
Layer (SASL) framework for application layer authentication 
between AllJoyn-enabled applications. It supports authentication 
mechanisms beyond what are defines by the D-Bus specification.

The D-Bus specification can be found at (http://dbus.freedesktop.org/doc/dbus-specification.html).

### AllJoyn system key concepts

As previously stated, the AllJoyn framework provides an underlying 
bus architecture for applications to advertise, discover, and 
make use of each other's functionality. To achieve this, the 
AllJoyn framework provides an object-oriented software framework 
for applications to interact with each other. 

#### AllJoyn router

The AllJoyn Router component provides core functionality of 
the AllJoyn system, including peer-to-peer advertisement/discovery, 
connection establishment, broadcast signaling and control/data 
messages routing. The AllJoyn router implements software bus 
functionality and an application connects to this bus to avail 
core functions of the AllJoyn framework. Each instance of the 
AllJoyn router has an associated globally unique identifier (GUID) 
which is self-assigned. Currently, this GUID is not persisted, 
so a new GUID is assigned whenever the AllJoyn router starts up.
An AllJoyn router can be either bundled with each application 
(bundled model), or can be shared across multiple applications 
(standalone model) on the device as shown below.

![alljoyn-bundled-standalone-router-examples][alljoyn-bundled-standalone-router-examples]

**Figure:** AllJoyn bundled and standalone router examples

An AllJoyn router has an associated AllJoyn protocol version 
that defines the set of functionality it supports. This protocol 
version is exchanged between AllJoyn routers on an AllJoyn 
network when they establish connection with each other as 
part of AllJoyn session establishment. 

#### AllJoyn bus

An AllJoyn router provides software bus functionality where 
one or more applications can connect to it to exchange messages. 
AllJoyn router instances on a device form a logical AllJoyn 
bus local to the device as shown below. 

![logical-router-bus-mapping][logical-router-bus-mapping]

**Figure:** Logical mapping of AllJoyn router to AllJoyn bus

The logical AllJoyn bus maps to a single AllJoyn router in two cases:  

* Bundled deployment model with only one app on the device, 
shown as UC2. 
* Standalone deployment model with one or more apps on the device, 
shown as UC3. 

The logical AllJoyn bus maps to multiple AllJoyn router instances 
in the bundled deployment model with multiple apps on the device, 
shown as UC1.

**NOTE:*8 The AllJoyn router and AllJoyn bus terminology are used 
interchangeably in this document as these refer to same set of 
bus functionality provided by the AllJoyn system.

The following figure shows a simplistic view of the local 
AllJoyn bus on two different devices with multiple applications 
connecting to the bus. 

![alljoyn-bus][alljoyn-bus]

**Figure:** AllJoyn bus

The AllJoyn bus provides a medium for communication between 
apps connected to the bus. AllJoyn buses on multiple devices 
communicate with each other using the underlying network 
technology such as Wi-Fi.

Multiple instances of AllJoyn buses across multiple devices 
form a logical distributed AllJoyn software bus as shown below.

![distributed-alljoyn-bus][distributed-alljoyn-bus]

**Figure:** Distributed AllJoyn bus

The distributed AllJoyn bus hides all the communication 
link details from the applications running on multiple devices. 
To an application connected to the AllJoyn bus, a remote application 
running on another device looks like an app that is local to 
the device. AllJoyn distributed bus provides a fast lightweight 
way to move messages across the distributed system.

#### AllJoyn service

As described earlier, provider AllJoyn applications provide 
services that can be consumed by other applications in the 
AllJoyn network. For example, a TV may provide a picture rendering 
service to display pictures from another AllJoyn device 
(e.g., smartphone). An AllJoyn Service is a notional/logical 
concept and is defined by one or more AllJoyn interfaces (described 
in [AllJoyn interfaces][alljoyn-interfaces]) which expose 
service functionality to consumers. 

An AllJoyn application can act as both provider and consumer 
by providing and consuming AllJoyn services at the same time.

#### Unique name

Each AllJoyn application connects to a single AllJoyn router. 
To enable addressing for individual applications, an AllJoyn 
router assigns a unique name to each connecting application. 
The unique name uses AllJoyn router GUID as the prefix. It 
follows the format below:

```
Unique Name = ":"<AJ router GUID>"."<Seq #>
```

**NOTE:** The ":<AJ router GUID>.1" unique name is always given 
to the AllJoyn router local endpoint.

The following figure shows the unique name assignment for three connected 
apps to an AllJoyn bus by a single AllJoyn router with GUID=100. 

![uniquename-assignment-1][uniquename-assignment-1]

**Figure:** AllJoyn unique name assignment 1 (multiple apps connected to single AllJoyn router)

This scenario illustrates a device with multiple AllJoyn 
applications connected to a single AllJoyn router. It is 
expected that a large number of AllJoyn-enabled devices 
will be single-purpose devices (e.g., refrigerator, oven, 
light bulb, etc.), and will have only one application residing 
on the device and connecting to the AllJoyn bus. However, there 
can be devices where a single instance of an AllJoyn router will 
support multiple applications, such as a TV.

The following fiture shows the unique name assignment for AllJoyn 
apps with multiple instances of an AllJoyn router forming an 
AllJoyn bus. 

![uniquename-assignment-2][uniquename-assignment-2]

**Figure:** AllJoyn unique name assignment 2 (each app has instance of AllJoyn router)

**NOTE:** The GUID part in each unique name is different and 
corresponds to the GUID for the associated AllJoyn router. 

The following figure shows the unique name assignment for 
AllJoyn apps on two different devices connected over a 
distributed AllJoyn bus.

![uniquename-assignment-3][uniquename-assignment-3]

**Figure:** AllJoyn unique name assignment 3 (AllJoyn apps on two devices connected over distributed AllJoyn bus)

#### Well-known name

An AllJoyn application can decide to use well-known names for 
its services. A well-known name is a consistent way to refer 
to a service (or collection of services) offered over the 
AllJoyn bus. An app can use a single well-known name for all 
the services it offers, or it can use multiple well-known names 
across these services. 

An application can request use of one or more well-known names 
from the AllJoyn bus for services it provides. If the requested 
well-known name is not already in use, exclusive use of that 
well-known name is granted to the application. This ensures 
that well-known names represent unique addresses on the AllJoyn 
bus at any point. The well-known name uniqueness is guaranteed 
only within the local AllJoyn bus. Global uniqueness for a 
well-known name should be achieved by adapting certain naming 
guidelines and format.

The AllJoyn well-known name follows the reverse domain name 
format. There can be multiple instances of a given application 
on a distributed AllJoyn bus, for example, the same refrigerator 
application running on two different refrigerators from the same 
vendor in the proximal network (one in the kitchen and one in 
the basement). To distinguish multiple instances of a given 
app on the AllJoyn bus, the well-known name should have a 
unique app specific identifier as a suffix, e.g., a GUID 
identifying the app instance. 

The AllJoyn well-known name (WKN) follows the D-Bus specification 
guidelines for naming and has following format:

```
WKN = <reverse domain style name for service/app>"."<app instance GUID>
```

For example, a refrigerator service can use the following 
well-known name:

```
com.alljoyn.Refrigerator.12345678
```

#### AllJoyn object

AllJoyn applications implement one or more AllJoyn objects 
to support AllJoyn services functionality. These AllJoyn objects 
are called service objects and are advertised over the AllJoyn 
bus. Other AllJoyn applications can discover these objects from 
the AllJoyn bus and access them remotely to consume services provided 
by them. 

A consumer application accesses an AllJoyn service object 
through a proxy object. A proxy object is a local representation 
of a remote service object that is accessed through the AllJoyn bus. 

The following figure shows the distinction between the AllJoyn 
service object and proxy object.

![alljoyn-service-object-proxy-object][alljoyn-service-object-proxy-object]

**Figure:** AllJoyn service object and AllJoyn proxy object

Each AllJoyn service object instance has an associated object 
path that uniquely identifies that object instance. This object 
path gets assigned when a service object gets created on the 
provider. The proxy object requires an object path to establish 
communication with the remote service object. The object path 
scope is within a given application, so object paths must be 
unique only with the associated application implementing the 
objects. Hence, object path naming does not need to follow 
reverse domain naming convention, and it can be of any form 
chosen by the application. 

The object path naming also adheres to the D-Bus specification 
naming guidelines. An example object path for the service 
object implemented by a refrigerator can be:

```
/MyApp/Refrigerator
```

#### AllJoyn interfaces

Each AllJoyn object exposes its functionality over the AllJoyn 
bus through one or more AllJoyn interfaces. An AllJoyn interface 
defines a contract for communication between an entity implementing 
the interface specification and other entities interested in 
making use of the services provided by the interface. The AllJoyn 
interfaces are candidates for standardization to enable interoperability 
among AllJoyn enabled IoE devices.

An AllJoyn interface can include one or more of following 
types of members:

* Methods: A method is a function call that typically takes 
a set of inputs, performs some processing using the inputs, 
and typically returns one or more outputs reflecting the results 
of the processing operation. Note that it is not mandatory for 
methods to have input and/or output parameters. It is also not 
mandatory for methods to have a reply.
* Signals: A signal is an asynchronous notification that is 
generated by a service to notify one or more remote peers of 
an event or state change. Signals can be delivered over an
already-established peer-to-peer AllJoyn connection (AllJoyn session), 
or they can be broadcast globally to all AllJoyn peers over 
the distributed AllJoyn bus. Signals can be of three types: 
  * Session-specific signals: These signals get delivered to 
  one or more peers connected over a given AllJoyn session 
  in the proximal network. If a destination is specified, 
  the signal is delivered to only that destination node connected 
  over the AllJoyn session. If no destination is specified, 
  the signal gets delivered to all nodes connected over the 
  given session except the node that generated the signal. 
  If the session is a multi-point session, such a signal 
  is sent over multicast to all the other participants. 
  * Session broadcast signals: These signals get delivered 
  to all the nodes connected via any AllJoyn session in 
  the proximal network. 
  * Sessionless signals: These signals get delivered to all 
  the nodes in a proximal network that have expressed interest 
  in receiving sessionless signals. Nodes do not need to be 
  connected over an AllJoyn session to receive such signals. 
  Sessionless signals are essentially broadcast signals independent 
  of a session connection.
* Properties: A property is a variable that holds values and 
it may be read-only, read-write or write-only.

Every AllJoyn interface has a globally unique interface name 
that identifies the grouping of methods, signals, and properties 
provided by that interface. The AllJoyn interface name gets 
defined as part of standardizing the interface. Similar to 
the well-known name, the AllJoyn interface name also follows 
reverse domain name format and D-Bus specification naming guidelines.

For example, a refrigerator could support the following standard 
AllJoyn refrigerator interface. 

```
org.alljoyn.Refrigerator
```

#### AllJoyn core library

The AllJoyn Core Library exposes AllJoyn bus functionality to 
AllJoyn applications. Each application links with a single 
instance of AllJoyn core library to connect with the AllJoyn 
bus. The AllJoyn core library acts as an application's gateway 
for peer-to-peer communications with other remote AllJoyn apps. 
It can be used to connect to the bus, to advertise services, 
to discover services, to establish connection with remote peer, 
to consume services, and many other AllJoyn functions. An 
application registers its objects with the AllJoyn core library 
to advertise these over the AllJoyn bus.

The following figure shows three apps connecting to a given 
AllJoyn bus via the AllJoyn core Library. 

![alljoyn-core-library][alljoyn-core-library]

**Figure:** AllJoyn core library

An AllJoyn core library can be a Standard Core Library (SCL), 
developed for use by AllJoyn standard applications or a 
Thin Core Library (TCL) developed for use by AllJoyn thin 
applications. Most of the system design in the document 
is described using standard core library deployment. 
For thin core library design details, see [Thin Apps][thin-apps].

#### About feature

The AllJoyn framework supports the About feature as part 
of the AJ Core Library. The About feature enables an application 
to expose key information about itself including app name, app 
identifier, device name, device identifier and a list of AllJoyn 
interfaces supported by the app among other details. This 
feature is supported by org.alljoyn.About interface implemented 
by the org.alljoyn.About object.

An application advertises key information about itself via an 
Announce signal defined by the About interface. This signal is 
sent as a sessionless signal on the proximal AllJoyn network. 
Any AllJoyn applications interested in discovering services 
via the AllJoyn interfaces make use of the Announce signal 
for discovery. The About feature also provides a mechanism 
to fetch application data via a direct method call. See the 
[About HLD] for design details on the About feature.

#### AllJoyn endpoints

AllJoyn applications exchange data in the form of D-Bus 
formatted messages. These messages specify source and destination 
as Endpoints. An AllJoyn Endpoint represents one side of an 
AllJoyn communication link.Endpoints are used to route 
messages to appropriate destinations. 

Both the Core Library and AllJoyn Router maintain endpoints 
to enable message routing. The Core Library maintains the 
following endpoints:
* **Local Endpoint**: The local endpoint within the 
Core Library represents a connection to the attached application. 
* **Remote Endpoint**: The remote endpoint within the 
Core Library represents the connection to the AllJoyn 
router.This is applicable only for the case when AllJoyn 
router is not bundled.

An endpoint maintained by the AllJoyn router is uniquely 
identified by a unique name assigned to it.The AllJoyn 
router supports the following endpoints:

* **Local Endpoint**: A local endpoint is an endpoint within the 
AllJoyn router itself. It identifies a connection to self 
and is used to exchange AllJoyn control messages between 
AllJoyn routers. This is the first endpoint which gets assigned 
and always has the unique name ":<AJ router GUID>.1"
* **Remote Endpoint**: A remote endpoint identifies the connection 
between the application and the AllJoyn router. Messages destined 
to applications get routed to app endpoints.
* **Bus-to-Bus Endpoint**: A Bus-to-Bus (B2B) endpoint is a 
specialized kind of remote endpoint that identifies 
the connection between two AllJoyn routers. This endpoint 
is used as next hop to route messages between AllJoyn routers.

A routing table is maintained at the AllJoyn router that is 
responsible for routing messages to different types of endpoints. 
Control messages between two AllJoyn routers (e.g., AttachSession 
message) get routed to the local endpoint. AllJoyn messages 
between two applications get routed to app endpoints. These 
messages will have app endpoints as source and destination 
within the message. B2B endpoints are used as the next hop 
when routing messages (app-directed or control messages) 
between two AllJoyn routers. 

The following figure shows different endpoints in the AllJoyn system.

![alljoyn-endpoints][alljoyn-endpoints]

**Figure:** AllJoyn endpoints

#### Introspection

The AllJoyn system supports D-Bus defined introspection 
feature that enables AllJoyn objects to be introspected at 
runtime, returning introspection XML describing that object. 
The object should implement org.freedesktop.DBus.Introspectable 
interface. This interface has an Introspect method that can be 
called to retrieve introspection XML for the object.

#### AllJoyn entity relationship

It is useful and important to understand how different high-level 
AllJoyn entities relate to each other. 

The following figure captures the relationship between various 
high-level AllJoyn entities including device, application, 
objects, interfaces, and interface members. 

![alljoyn-entity-relationship][alljoyn-entity-relationship]

**Figure:** AllJoyn entity relationship

An AllJoyn-enabled device can support one or more AllJoyn 
applications. Each AllJoyn application supports one or more 
AllJoyn objects that implement desired application functionality. 
Application functionality can include providing AllJoyn 
services or consuming AllJoyn services, or both. Accordingly, 
objects supported by the AllJoyn application can be service 
objects, proxy objects, or combination of both. A service 
object exposes its functionality via one or more AllJoyn 
interfaces. Each AllJoyn interface can support one or more 
of methods, signals, and properties.

An AllJoyn service is implemented by one or more AllJoyn 
service objects. An AllJoyn service object can implement 
functionality for one or more AllJoyn services. Hence, AllJoyn 
service and AllJoyn service object have an n:n relationship as 
captured in the following figure.

![alljoyn-service-service-object-relationship][alljoyn-service-service-object-relationship]

**Figure:** AllJoyn service and AllJoyn service object relationship

### AllJoyn services

An AllJoyn application can support one or more service frameworks 
and some application layer services.

#### AllJoyn service framework

AllJoyn service frameworks provide some of the core and 
fundamental functionality developed as enablers for higher-layer 
application services. Service frameworks sit on top of the AllJoyn 
router and provide APIs to application developers to invoke their 
functionality. Initial AllJoyn service frameworks include 
Configuration service framework, Onboarding service framework, 
Notification service framework, and Control Panel service framework. 

**NOTE:** Service frameworks are also referred to as base services.

Example: a refrigerator application can make use of the Onboarding 
service framework to onboard a refrigerator to a home network 
and send out notifications to user devices using the Notification 
service framework.

#### Application layer service

An application layer service is an app-specific service provided 
by the AllJoyn application to achieve desired application 
layer functionality. These application layer services can 
make use of service frameworks to achieve their functionality.

Example: a refrigerator application can offer an application 
layer service to change refrigerator and freezer temperature. 
This service can make use of the Notification service framework 
to send out a notification when the temperature setting goes 
out of a specified range to notify the user.

### AllJoyn transport

The AllJoyn Transport is an abstract concept that enables 
connection setup and message routing across AllJoyn applications 
via AllJoyn routers. The AllJoyn transport logic in turn 
supports transmitting messages over multiple underlying 
physical transports including TCP transport, UDP transport 
and Local Transport (e.g., UNIX domain sockets).

The AllJoyn transport logic delivers the advertisement and 
discovery messages based on specified list of transports by 
the app.  Similarly, the AllJoyn transport enables session 
establishment and message routing over multiple underlying 
transports based on transport selection made by the application. 
The set of underlying transports supported by the AllJoyn 
transport is specified by a TransportMask as captured in 
[AllJoyn Transport in Networking Model][alljoyn-transport-in-networking-model].
If an app does not specify any transport(s), the AllJoyn 
transport value defaults to TRANSPORT_ANY.

See [AllJoyn Transport][alljoyn-transport-section] for more information.

### Advertisement and discovery 

The AllJoyn framework provides a means for applications to 
advertise and discover AllJoyn services. The AllJoyn discovery 
protocol manages the dynamic nature of services coming in 
and going out of the proximal AllJoyn network and notifies 
AllJoyn applications of the same. The AllJoyn framework 
leverages an underlying transport-specific mechanism to 
optimize the discovery process. The AllJoyn framework makes 
use of IP multicast over Wi-Fi for service advertisement and 
discovery. The details of underlying mechanism are hidden 
from the AllJoyn applications.

The following sections details the ways that applications 
can use to advertise and discover services over the AllJoyn framework.

#### Name-based discovery

In the name-based discovery, advertisement and discovery 
typically happens using a well-known name. In this approach, 
the unique name can also be used for discovery per an application's 
discretion (e.g., if a well-known name was not assigned). 
A provider application advertises supported well-known names 
over the proximal AllJoyn network leveraging the underlying 
transport specific mechanism (IP multicast over Wi-Fi). 
These well-known names get advertised as part of an advertisement 
message generated by the AllJoyn router. 

A consumer application interested in a given well-known name 
can ask the AllJoyn router to begin discovering that name. 
When the provider app advertising that name comes in the 
proximity, the AllJoyn router receives the corresponding 
advertisement. The AllJoyn router then sends a service discovery 
notification to the application for the well-known name.

The advertisement message carries connectivity information 
back to the provider app. After discovery, the consumer app 
can request AllJoyn router to establish a connection with 
the discovered provider app for consuming the service. 
The AllJoyn router uses the connectivity information to 
connect back to the provider app.

#### Announcement-based discovery

Since AllJoyn services are ultimately implemented by one or 
more interfaces, service discovery can be achieved by discovering 
associated AllJoyn interfaces. In the announcement-based discovery, 
advertisement and discovery happens using AllJoyn interface names. 
This mechanism is intended to be used by devices to advertise 
their capabilities.

The provider application creates a service announcement message 
specifying a list of AllJoyn interfaces supported by that 
application. The service announcement message is delivered 
as a broadcast signal message using sessionless signaling 
mechanism (described in detail in [Sessionless Signal][sessionless-signal-section]).

Consumer applications interested in making use of AllJoyn 
services look for these broadcast service announcement messages 
by specifically registering its interest in receiving these 
announcements with AllJoyn router. When the consumer device 
is in the proximity of a provider, it receives the service 
announcement that contains the AllJoyn interfaces supported 
by the provider.

The AllJoyn router maintains connectivity information to 
connect back to the provider from which the service announcement 
message was received. After discovery, the consumer app can 
request the AllJoyn router to establish a connection with 
the provider app that supports the desired interfaces for 
consuming the service. The AllJoyn router uses connectivity 
information to connect back to the provider app.

#### Discovery enhancements in the 14.06 release

The AllJoyn discovery feature was enhanced in the 14.06 
release to enable the discovery of devices/apps that support 
a certain set of interfaces in a more efficient way. The 
enhanced discovery is referred to as Next-Generation Name 
Service (NGNS). NGNS supports a multicast DNS (mDNS)-based 
discovery protocol that enables specifying AllJoyn interfaces 
in an over-the-wire discovery message. In addition, the mDNS-based 
protocol is designed to provide discovery responses over unicast 
to improve performance of the discovery protocol and minimize 
overall multicast traffic generated during the AllJoyn discovery process. 

The presence detection mechanism for AllJoyn devices/apps 
has been enhanced by adding an explicit mDNS-based ping() 
message that is sent over unicast to determine if the remote 
endpoint is still alive. The ping() mechanism is driven by 
the application based on application logic.

### AllJoyn session

Once a client discovers an AllJoyn service of interest, 
it must connect with the service in order to consume that 
service (except for the Notification service framework, 
which relies completely on sessionless signals). Connecting 
with a service involves establishing an AllJoyn session with 
that service. A session is a flow-controlled data connection 
between a consumer and provider, and as such allows the client 
to communicate with the service. 

A provider app advertising a service binds a session port with 
the AllJoyn bus and listens for clients to join the session. 
The action of binding and listening makes the provider the 
session host. The session port is typically known ahead of 
time to both the consumer and the provider app. In the case 
of announcement-based discovery, the session port is discovered 
via the Announcement message. After discovering a particular 
service, the consumer app requests the AllJoyn router to 
join the session with the remote service (making it a session 
joiner) by specifying the session port and service's unique 
name/well-known name. After this, the AllJoyn router takes 
care of establishing the session between the consumer and 
the provider apps. 

Each session has a unique session identifier assigned by the 
provider app (session host). An AllJoyn session can be one 
of the following:

* Point-to-point session: A session with only two participants-the 
session host and the session joiner. 
* Multi-point session: A session with multiple participants-a 
single session host and multiple session joiners.

After session establishment, the consumer application must 
create a proxy object to interact with the provider app. The 
proxy object should be initialized with a session ID and the 
remote service object path. Once complete, the consumer app 
can now interact with the remote service object via this proxy object.

### Sessionless signals

The AllJoyn framework provides a mechanism to broadcast signals 
over the proximal AllJoyn network. A broadcast signal does 
not require any application layer session to be established 
for delivering the signal. Such signals are referred to as 
sessionless signals and are broadcast using a sessionless 
signaling mechanism supported by the AllJoyn router. 

The delivery of sessionless signals is done as a two-step process. 

1. The provider device (sessionless signal emitter) advertises 
that there are sessionless signals to receive. 
2. Any consumer devices wishing to receive a sessionless 
signal will connect with the provider device to retrieve new signals. 

Using the sessionless signal mechanism, a provider application 
can send broadcast signals to the AllJoyn router. The AllJoyn 
router maintains a cache for these signals. The content of the 
sessionless signal cache is versioned. The AllJoyn router sends 
out a sessionless signal advertisement message notifying other 
devices of new signals at the provider device. The sessionless 
signal advertisement message includes a sessionless signal-specific 
well-known name specifying the version of the sessionless signal cache. 

The consumer app interested in receiving the sessionless signal 
performs discovery for the sessionless signal-specific well-known 
name. The AllJoyn bus on the consumer maintains the latest sessionless 
signal version it has received from each of the provider AllJoyn router. 
If it detects a sessionless signal advertisement with an updated 
sessionless signal version, it will fetch new set of sessionless 
signals and deliver them to the interested consumer applications.

#### Sessionless signal enhancement in the 14.06 release

The sessionless signal feature was enhanced in the 14.06 release 
to enable a consumer application to request sessionless signals 
from provider applications that support certain desired AllJoyn 
interfaces. The following sessionless signal enhancements were made:

* The sessionless signal advertised name was enhanced to add 
<INTERFACE> information from the header of the sessionless signal. 
Consumers use this to fetch sessionless signals only from those 
providers that are emitting signals from the <INTERFACE> it 
is interested in. A separate sessionless signal name is advertised 
one for each unique interface in the sessionless signal cache.
* A mechanism was added for the consumer app to indicate receiving 
Announce sessionless signal only from applications implementing 
certain AllJoyn interfaces.

Sessionless signals are only fetched from those providers that 
support desired interfaces. This improves the overall performance 
of the sessionless signal feature.

### Thin apps

An AllJoyn Thin App is designed for use in embedded devices 
such as sensors. These types of embedded devices are optimized 
for a specific set of functions and are constrained in energy, 
memory and computing power. An AllJoyn thin app is designed to 
bring the benefits of the AllJoyn framework to embedded systems. 
The thin app is designed to have a very small memory footprint. 

A thin AllJoyn device makes use of lightweight thin application 
code along with the AllJoyn Thin Core Library (AJTCL) running 
on the device. It does not have an AllJoyn router running on 
that device. As a result, the thin app must use an AllJoyn 
router running on another AllJoyn-enabled device, essentially 
borrowing the AllJoyn router functionality running on that device. 

At startup, the thin application discovers and connects with 
an AllJoyn router running on another AllJoyn-enabled device. 
From that point onwards, the thin app uses that AllJoyn router 
for accomplishing core AllJoyn functionality including service 
advertisement/discovery, session establishment, signal 
delivery, etc. If a thin app is not able to connect to previously 
discovered AllJoyn router, it attempts to discover another 
AllJoyn router to connect to.

An AllJoyn thin app is fully interoperable with an AllJoyn 
standard application. It uses same set of over-the-wire protocols 
as a standard AllJoyn app. This ensures compatibility between 
the thin app and standard apps. An AllJoyn standard app communicating 
with a thin app will not know that it is talking to a thin app 
and vice versa. However, there are some message size constraints 
that apply to the thin app based on available RAM size.

### AllJoyn protocol version

Functionality implemented by the AllJoyn Router is versioned through an AllJoyn
Protocol Version (AJPV) field. The following table shows the AJPV for various
AllJoyn releases; unless otherwise noted the AJPV for the major release version
applies to all the patch release versions as well. The AJPV is exchanged
between routers as part of the BusHello messaging during the AllJoyn session
establishment and between the leaf and routing node when the leaf node connects
to the router. This field is used by the core libraries to identify
compatibility with the router, and specifically by thin apps to determine
whether or not to connect to a particular router or keep searching for another
one.  It is also used by the router to determine if functionality is available
at the leaf (e.g. self-join, SessionLostWithReason, etc.)

** Table: ** AllJoyn Release to Protocol Version mapping

| &#160; Release version &#160; | &#160; AJPV &#160; |
|:------------------------------:|:-----------------:|
|        legacy 03.04.06         |        9          |
|        v14.02                  |        9          |
|        v14.06                  |        10         |
|        v14.12                  |        11         |
|        v15.04                  |        12         |


[list-of-subjects]: /learn/core/system-description/
[message-format]: /learn/core/system-description/data-exchange#message-format
[alljoyn-interfaces]: #alljoyn-interfaces
[sessionless-signal-section]: /learn/core/system-description/sessionless-signal
[thin-apps]: /learn/core/system-description/thin-apps
[alljoyn-transport-section]: /learn/core/system-description/alljoyn-transport
[alljoyn-transport-in-networking-model]:  /learn/core/system-description/alljoyn-transport#alljoyn-transport-in-networking-model


[ioe-network-example]: /files/learn/system-desc/ioe-network-example.png
[alljoyn-network]: /files/learn/system-desc/alljoyn-network.png

[alljoyn-bundled-standalone-router-examples]: /files/learn/system-desc/alljoyn-bundled-standalone-router-examples.png
[logical-router-bus-mapping]: /files/learn/system-desc/logical-router-bus-mapping.png
[alljoyn-bus]: /files/learn/system-desc/alljoyn-bus.png
[distributed-alljoyn-bus]: /files/learn/system-desc/distributed-alljoyn-bus.png
[uniquename-assignment-1]: /files/learn/system-desc/uniquename-assignment-1.png
[uniquename-assignment-2]: /files/learn/system-desc/uniquename-assignment-2.png
[uniquename-assignment-3]: /files/learn/system-desc/uniquename-assignment-3.png
[alljoyn-service-object-proxy-object]: /files/learn/system-desc/alljoyn-service-object-proxy-object.png
[alljoyn-core-library]: /files/learn/system-desc/alljoyn-core-library.png
[alljoyn-endpoints]: /files/learn/system-desc/alljoyn-endpoints.png
[alljoyn-entity-relationship]: /files/learn/system-desc/alljoyn-entity-relationship.png
[alljoyn-service-service-object-relationship]: /files/learn/system-desc/alljoyn-service-service-object-relationship.png
