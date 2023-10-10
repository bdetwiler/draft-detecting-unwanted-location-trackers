# Detecting Unwanted Location Trackers (DULT)


## Background
Location-tracking accessories provide numerous benefits to users (e.g., such as being able to find where they left their keys), but can also have security and privacy implications if used for malicious purposes. These accessories can be misused to track another person’s location without their knowledge.


To address this threat, accessory manufacturers have developed independent solutions for protecting users from unwanted tracking. However, this requires users to know about the threat of unwanted tracking, download multiple apps, and constantly be checking for the threat of unwanted tracking. In order to build a scalable solution for detecting unwanted tracking, trackers require a consistent protocol and set of behaviors that will enable protection from unwanted tracking using any tracker.


## Goals


The goal of the DULT WG is to standardize a protocol for information exchange between location-tracking accessories and nearby devices, along with actions that these accessories and devices should take once unwanted tracking is detected. The intent of this WG is to make it easier for arbitrary devices to detect unwanted tracking by these accessories. The protocols and interactions between devices may be limited to certain states or modes, such as the accessory being separated from a paired/owner device.


The working group will define privacy and security properties of its solution, and evaluate the tradeoffs.


The WG protocol design will be guided by an intent to:

* Minimize hardware changes needed in tracking accessories to implement this protocol; and
* Not preclude adoption by manufacturers of larger devices whose primary purpose is not location tracking, but have location tracking capabilities (e.g., headphones, bicycle, smartphone)


## Program of Work


The WG is expected to:

1. Standardize a protocol between tracking accessories and nearby devices, which may:

 * Allow a tracking accessory to identify & advertise its presence when in a detectable mode
 * Allow a nearby device to trigger behavior on an unwanted tracking accessory to aid in determining its physical location
 * Allow nearby devices to fetch additional information about a tracker accessory


2. Specify practices that accessory manufacturers can implement to deter malicious use of tracking accessories and support the implementation of the WG-specified protocol.


3. Specify guidance for non-owner device platforms necessary to support implementation of the WG specified protocol



The WG will not standardize an end-to-end platform-based unwanted tracking detection system or define requirements for interactions between accessory manufacturers and law enforcement. In addition, these items are out-of-scope:

 * Mechanisms for detecting tracking accessories that do not implement the protocol specified by the WG, and
 * Mechanisms for detecting whether a tracking accessory implements the protocol or allowing a tracking accessory to attest that it implements the protocol


Since most of the existing tracking accessories use Bluetooth, the DULT WG will coordinate as needed with the IETF 6lo WG and Bluetooth SIG.


### Milestones

* Submit an informational document about the state of tracker accessory platforms and how they work for publication
* Submit a standards document defining the protocol to detect and interact with unwanted tracker accessories for publication
