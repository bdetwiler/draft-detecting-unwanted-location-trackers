---
title: "Detecting Unwanted Location Trackers"
category: info

docname: draft-detecting-unwanted-location-trackers-latest
submissiontype: independent
number:
date:
consensus:
v: 3
# area: AREA
# workgroup:
keyword:
 - unwanted tracking
 - location tracker
venue:
#  group:
#  type:
#  mail:
#  arch:
  github:
  latest:

author:
 -
    fullname: Brent Ledvina
    organization: Apple
    email: bledvina@apple.com

 -
    fullname: Zach Eddinger
    organization: Google
    email: zae@google.com

 -
    fullname: Ben Detwiler
    organization: Apple
    email: bdetwiler@apple.com

 -
    fullname: Siddika Parlak Polatkan
    organization: Google
    email: siddikap@google.com



normative:
 BTCore5.4:
   title: Bluetooth Core Specification v5.4
   date: 2023-01-31
   format:
     PDF: https://www.bluetooth.org/DocMan/handlers/DownloadDoc.ashx?doc_id=556599



RFC8126:


informative:


--- abstract
This document lists a set of best practices and protocols for accessory manufacturers whose products have built-in location-tracking capabilities. By following these requirements and recommendations, a location-tracking accessory will be compatible with unwanted tracking detection and alerts on mobile platforms. This is an important capability for improving the privacy and safety of individuals in the circumstance that those accessories are used to track their location without their knowledge or consent.

--- middle

# Introduction

This document’s goal is to, in part, help protect the privacy of individuals from unwanted tracking by location-tracking accessories. Location-tracking accessories provide numerous benefits to consumers, but, as with all technology, it is possible for them to be misused. Misuse of location-tracking accessories can result in unwanted tracking of individuals or items for nefarious purposes such as stalking, harassment, and theft. Formalizing a set of best practices for manufacturers will allow for scalable compatibility with unwanted tracking detection technologies on various smartphone platforms and improve privacy and security for individuals.

Unwanted tracking detection can both detect and alert individuals that a location tracker separated from the owner's device is traveling with them, as well as provide means to find and disable the tracker. This document outlines technical best practices for location tracker manufacturers, which will allow for their compatibility with unwanted tracking detection and alerting technology on platforms.


## Conventions and Definitions

{::boilerplate bcp14-tagged}


## Terminology
Throughout this document, these terms have specific meanings:

- The term platform is used to refer to mobile device hardware and associated operating system. Examples of mobile devices are phones, tablets, laptops, etc.
- The term accessory is used to refer to any product intended to interface with a platform through the means described in this specification.
- The term owner device is a device that is paired to the accessory and can retrieve the accessory’s location.
- The term non-owner device refers to a device that may connect to an accessory but is not an owner device of that accessory.
- The term location-tracking accessory refers to any accessory that has location-tracking capabilities, including, but not limited to, crowd-sourced location, GPS/GNSS location, WiFi location, cell location, etc., and provides the location information back to the owner of the accessory via the internet, cellular connection, etc.
- The term location-enabled state refers to the state an accessory in where its location can be remotely viewed by its owner
- The term location-enabled advertisement payload refers to the Bluetooth (BT) advertisement payload that is advertised when an accessory has recently, is currently, or will in the future provide location updates to its owner
- The term unwanted tracking (UT) refers to undesired tracking of a person, their property, or their belongings by a location-enabled accessory.
- The term unwanted tracking detection refers to the algorithms that detect the presence of an unknown accessory traveling with a person over time.
- The term unwanted tracking alert refers to notifying the user of the presence of an unrecognized accessory that may be traveling with them over time and allows them to take various actions, including playing a sound on the accessory if it’s in Bluetooth Low Energy (LE) range.
- The term platform-compatible method refers to a method of communication between the platform and the accessory/accessory manufacturers to exchange information, including, but not limited to, BT GATT protocol, BT advertisement, HTTP, etc.

# Background


## Applicability
These best practices are REQUIRED for location-enabled accessories that are small and not easily discoverable. For large accessories, such as a bicycle, these best practices are RECOMMENDED.

Accessories are considered easily discoverable if they meet one of the following criteria:  
- The item is larger than 30 cm in at least one dimension.  
- The item is larger than 18 cm x 13 cm in two of its dimensions.  
- The item is larger than 250 cm<sup>3</sup> in three-dimensional space.

# Requirements


## Overview
This section details requirements and recommendations for best practices for location-enabled accessory manufacturers to allow unwanted tracking detection by platform makers.

## Bluetooth Low Energy
The accessory SHALL use Bluetooth Low Energy (LE) as the transport protocol. This enables platforms to detect and connect to accessories.

### Advertising
The accessory SHALL advertise using Bluetooth LE.

### Connection

The accessory MUST support at least one non-owner unencrypted connection in a peripheral role.
The connection interval of the Bluetooth LE link between the device and accessory MAY depend on the type of user interaction. Non-owner connections to the accessory SHALL be implemented using a platform-compatible method, e.g., BT GATT service.

## Location Tracking

The location-enabled accessory has location capabilities via Bluetooth crowd-sourcing, GPS/GNSS location, WiFi location, cellular location, or by some other means. Furthermore, the accessory has a way to communicate its location to its owner via a network (e.g., cell network, crowd-sourced location via Bluetooth, etc.).

The accessory SHALL maintain an internal state that determines when its location is, or has been, available to the owner via a network. This state is called the location-enabled state.

Misuse of location-enabled accessories can occur when the owner’s device is not physically with the accessory. Thereby, the accessory SHOULD maintain a second internal state, denoted the near-owner state, which indicates if the accessory is connected to or nearby one or more of the owner’s devices. Near-owner state can take on two values, either near-owner mode or separated mode. Near-owner mode is denoted as the opposite of separated mode.

{{table-location-enabled-payload}} details the requirements and recommendations for advertising the location-enabled payload based on the location-enabled state and separated state.


~~~
                         +---------------------+
                         |      Location       |
                         |  Currently Enabled  |
                         |         OR          |
                         |  Enabled in Past 24 |
                         |        Hours        |
    +--------------------+---------------------|
    |         near-owner |     SHOULD NOT      |
    |            mode    | advertise location- |
    | Near-              |  enabled payload    |
    | Owner              +---------------------|
    | State    separated |   MUST advertise    |
    |            mode    |  location-enabled   |
    |                    |     payload         |
    +--------------------+---------------------+
~~~
{: #table-location-enabled-payload title="Requirements & Recommendations For Advertising Location-Enabled Payload"}



It is RECOMMENDED that the location-enabled payload is only advertised when the accessory is in the separated state. The reasoning behind this recommendation is that unwanted tracking detection relies on the Bluetooth LE advertisements emitted while in the location-enabled state to determine if an unknown accessory is traveling with someone who is not the owner. If the location-enabled payload is advertised only in the separated state, that minimizes false-positive UT alerts.

As a point of clarity, if the accessory maker chooses to continue advertising the location-enabled payload while in near-owner mode, setting the [near-owner bit](#near-owner-bit) compensates for this.

## Location-enabled Bluetooth LE Advertisement Payload


### Overview
When in location-enabled state, the accessory SHALL advertise a Bluetooth LE format, denoted the location-enabled Bluetooth advertisement payload, that is recognizable to the platforms.

###  Location-enabled advertisement payload format
The payload format is defined in {{table-payload-format}}


|  Bytes |                                          Description                                          | Requirement |
|:------:|:----------------------------------------------------------------------------------------------|:-----------:|
|  0-5   | MAC address                                                                                   |  REQUIRED   |
|  6-8   | Flags TLV; length = 1 byte, type = 1 byte, value = 1 byte                                     |  OPTIONAL   |
|  9-12  | Service data TLV; length = 1 byte, type = 1 byte, value = 2 bytes (TBD value)                 |  REQUIRED   |
|   13   | Protocol ID (TBD value)                                                                       |  REQUIRED   |
|   14   | Near-owner bit (1 bit) + reserved (7 bits)                                                    |  REQUIRED   |
| 15-36  | Proprietary company payload data                                                              |  OPTIONAL   |
{: #table-payload-format title="Location-Enabled Payload Format" }


When Flags TLV are not added, the MAC address type needs to be set to random.
This implies that if Bluetooth LE pairing is supported, the accessory SHALL NOT use its public address as its public identity when exchanging pairing
keys at phase 3 (see Vol.3, Part H, Section 2.1 of the {{BTCore5.4}}) and it SHALL only use a static random address.
Additionally, the LE advertisement needs to be connectable to allow for non-owner unencrypted connections to the accessory.
Further details are discussed
in {{accessory-connections}}.


Proprietary company payload data is both OPTIONAL and variable length.


### Duration of advertising location-enabled advertisement payload

The accessory SHALL broadcast the location-enabled advertisement payload if location is available to the owner or was available any time within the past 24 hours. This allows unwanted tracking detection to operate both between and beyond the specific moments an accessory's location is made available to the owner.

### Maximum duration after physical separation from owner to transition into separated mode
The accessory SHALL transition from near-owner mode to separated mode if it has physically separated from the owner device for a duration no longer than 30 minutes.

### Maximum duration after reunification with owner to transition into near-owner mode
The accessory SHALL transition from separated to near-owner mode if it has reunited with the owner device for a duration no longer than 30 minutes.

## Resolvable and private address {#resolvable-private-address}
The Bluetooth LE advertisement payload SHALL contain a resolvable and private address for the accessory which is the 6-byte Bluetooth LE MAC address.


The address MUST be private and it MUST rotate periodically and be unlinkable; otherwise, if the same address is used for long periods of time, an adversary may be able to track a legitimate person from carrying the accessory.

The [rotation policy](#rotation-policy) defined below aims to reduce this risk.

Lastly, the address MUST be resolvable so owner devices can identify their paired accessories. Further details are described in [Paired Accessory Identification](#paired-accessory-identification).

A general approach to generate addresses meeting this requirement is to construct them using a Pseudo-Random Function (PRF) taking as input a key established during the pairing of the accessory and either a counter or coarse notion of time. The counter or coarse notion of time allows for the address to change periodically. The key allows the owner devices to predict the sequence of addresses for the purposes of recognizing its paired accessories.


### Rotation policy
An accessory SHALL rotate its resolvable and private address on any transition from near-owner state to separated state as well as any transition from separated state to near-owner state.

When in near-owner state, the accessory SHALL rotate its resolvable and private address every 15 minutes. This is a privacy consideration to deter tracking of the accessory by non-owners when it is in physical proximity to the owner.

When in a separated state, the accessory SHALL rotate its resolvable and private address every 24 hours.
This duration allows a platform's unwanted tracking algorithms to detect that the same accessory is in proximity for some period of time, when the owner is not in physical proximity.

## Service data TLV
The Service data TLV with a 2-byte UUID value of TODO provides a way for platforms to easily scan for and detect the location-enabled Bluetooth advertisement.

## Protocol ID
The 1-byte Protocol ID SHALL be set based on a registered value for the manufacturer, as defined in [Manufacturer Protocol ID Registry](#manufacturer-protocol-registry).


## Near-owner bit
It is important to prevent unwanted tracking alerts from occurring when the owner of the accessory is in physical proximity of the accessory, i.e., it is in near-owner mode. In order to allow suppression of unwanted tracking alerts for an accessory advertising the location-enabled advertisement with the owner nearby, the accessory MUST set the near-owner bit to be 1 when the near-owner state is in near-owner mode, otherwise the bit is set to 0. {{table-near-owner-bit}} specifies the values of this bit.


| Near-owner Bit Value | Near-owner state |
|----------------------|------------------|
| 0                    | Near-owner mode  |
| 1                    | Separated mode   |
{: #table-near-owner-bit title="Near-Owner Bit"}

## Bluetooth LE advertising interval

The Bluetooth LE advertising interval SHALL be as described in {{table-advertising-policy}}



|                      | Bluetooth LE Advertising Interval |
|----------------------|:---------------------------------:|
| Advertising policy   | 0.5 - 2 seconds                   |
{: #table-advertising-policy title="Advertising Policy" }


## Accessory Connections {#accessory-connections}
The accessory non-owner service UUID SHALL be TODO. This service SHALL use GATT over LE. The non-owner accessory service SHALL be instantiated as a primary service. The accessory non-owner characteristic UUID SHALL be TODO.

### Byte transmission order
The characteristic used within this service SHALL be transmitted with the least significant octet first (that is, little endian).

## Accessory Information
The following accessory information MUST be persistent through the lifetime of the accessory: [Product data](#product-data), [Manufacturer name](#manufacturer-name), [Model name](#model-name), [Accessory category](#accessory-category), and [Accessory capabilities](#accessory-capabilities).


### Opcodes
The opcodes for accessory information are defined in {{accessory-information-opcodes}}.

|             Opcode                  | Opcode value |        Operands                                   |     GATT subprocedure       |
|:-----------------------------------:|:------------:|:-------------------------------------------------:|:--------------------------: |
|           Get_Product_Data          | 0x306        |          None                                     |    Write; To Accessory      |
|      Get_Product_Data_Response      | 0x311        |      [Product Data](#product-data)                | Indications; From Accessory |
|        Get_Manufacturer_Name        | 0x307        |          None                                     |    Write; To Accessory      |
|    Get_Manufacturer_Name_Response   | 0x312        |    [Manufacturer Name](#manufacturer-name)        | Indications; From Accessory |
|            Get_Model_Name           | 0x308        |          None                                     |    Write; To Accessory      |
|       Get_Model_Name_Response       | 0x313        |       [Model Name](#model-name)                   | Indications; From Accessory |
|        Get_Accessory_Category       | 0x309        |          None                                     |    Write; To Accessory      |
|   Get_Accessory_Category_Response   | 0x314        |   [Accessory Category](#accessory-category)       | Indications; From Accessory |
|      Get_Accessory_Capabilities     | 0x30A        |          None                                     |    Write; To Accessory      |
| Get_Accessory_Capabilities_Response | 0x315        | [Accessory Capabilities](#accessory-capabilities) | Indications; From Accessory |
{: #accessory-information-opcodes title="Accessory Information Opcodes" }


#### Product data
The Product Data operand represents an 8-byte value that is intended to serve as a unique identifier for the accessory make and model.
This value SHALL be available in a public registry as defined in {{product-data-registry}}.

The Product Data operand is 8 bytes, composed of two 8-character hex strings (lowercase zero padded), each of which is 4 bytes.

For example, the Product Data value of 0xdfeceff1e1ff54db, the value converted to binary would be

>0xdfeceff1 11011111 11101100 11101111 11110001<br/>0xe1ff54db 11100001 11111111 01010100 11011011 


| Operand name         | Data type | Size (octets) | Description                         |
|:--------------------:|:---------:|:-------------:|:-----------------------------------:|
| Product Data         | Uint64    | 8             | See [Product data](#product-data)   |
{: title="Product Data Operand" }

#### Manufacturer name
The Manufacturer Name operand contains the name of the company whose brand will appear on the accessory, e.g., ”Acme”.


| Operand name  | Data type | Size (octets) |    Description    |
|:--------------------:|:---------:|:-------------:|:-----------------:|
| Manufacturer Name    | UTF-8     | 64            | Manufacturer name |
{: title="Manufacturer Name Operand" }

#### Model name
The Model Name operand contains the manufacturer specific model of the accessory.


| Operand name  | Data type | Size (octets) | Description |
|:--------------------:|:---------:|:-------------:|:-----------:|
| Model Name           | UTF-8     | 64            | Model name  |
{: title="Model Name Operand" }

#### Accessory category
The Accessory Category operand describes the category the accessory most closely resembles.


| Operand name  | Data type | Size (octets) |                                           Description                                           |
|:--------------------:|:---------:|:-------------:|:------------------------------------------------------------------------------------------------|
|  Accessory Category  |   Uint64  |       8       | Byte 0: Uint8 value of [Accessory Category Value](#accessory-category-value) <br/> Byte 1-7: Reserved |
{: title="Accessory Category Operand" }

#### Accessory capabilities
The Accessory Capabilities operand enumerates the various capabilities supported on the accessory as defined in {{table-accessory-capability}}.


| Operand name  | Data type | Size (octets) |                                                                        Description                                                                        |
|:--------------------:|:---------:|:-------------:|:---------------------------------------------------------------------------------------------------------------------------------------------------------|
| Accessory Capabilities | Uint32    | 4             | Bit 0 : Supports play sound <br/> Bit 1 : Supports motion detector UT <br/> Bit 2 : Supports serial number lookup by NFC <br/> Bit 3 : Supports serial number lookup by BLE |
{: #table-accessory-capability title="Accessory Capabilities Operand"}

For example, an accessory supporting play sound, motion detector UT, and serial number look-up over BT will have the value set as 1011 in binary and 11 as Uint32.

## Non-Owner Finding
Once a user has been notified of an unknown accessory traveling with them, it is REQUIRED they have the means to physically locate the accessory. This is called non-owner finding of the accessory.

### Hardware
This is a description of the REQUIRED and RECOMMENDED hardware to be incorporated into the accessory to enable non-owner finding.

### Motion detector
The accessory SHOULD include a motion detector that can detect accessory motion reliably (for example, an accelerometer). If the accessory includes an accelerometer, it MUST be configured to detect an orientation change of ±10° along any two axes of the accessory.

#### Implementation
The details in this section apply to those accessories that include a motion detector. Values of the variables referenced are specified in {{#table-motion-detector-time-values}}.

<br>
After T<sub>SEPARATED_UT_TIMEOUT</sub> in separated state, the accessory MUST enable the motion detector to detect any motion within T<sub>SEPARATED_UT_SAMPLING_RATE1</sub>.


If motion is not detected within the T<sub>SEPARATED_UT_SAMPLING_RATE1</sub> period, the accessory MUST stay in this state until it exits separated state.


If motion is detected within the T<sub>SEPARATED_UT_SAMPLING_RATE1</sub> the accessory MUST play a sound.
After first motion is detected, the movement detection period is decreased to T<sub>SEPARATED_UT_SAMPLING_RATE2</sub>.
The accessory MUST continue to play a sound for every detected motion.
The accessory SHALL disable the motion detector for T<sub>SEPARATED_UT_BACKOFF</sub> under either of the following conditions:

* Motion has been detected for 20 seconds at T<sub>SEPARATED_UT_SAMPLING_RATE2</sub> periods.
* Ten sounds are played.

If the accessory is still in separated state at the end of T<sub>SEPARATED_UT_BACKOFF</sub>, the UT behavior MUST restart.


A Bluetooth LE connection from a paired device MUST reset the separated behavior and transition the accessory to connected state.


|        Name                                  | Value        |     Description                                                              |
|:---------------------------------------------|:------------:|:-----------------------------------------------------------------------------|
|  T<sub>SEPARATED_UT_SAMPLING_RATE1</sub>     | 10 seconds   | Sampling rate when motion detector is enabled in separated state.            |
|  T<sub>SEPARATED_UT_SAMPLING_RATE2</sub>     | 0.5 seconds  | Motion detector sampling rate when movement is detected in separated state.  |
|  T<sub>SEPARATED_UT_BACKOFF</sub>            | 6 hours      | Period to disable motion detector if accessory is in separated state.        |
|  T<sub>SEPARATED_UT_TIMEOUT</sub>            | random value between 8-24 hours chosen from a uniform distribution                          | Time span in separated state before enabling motion detector.                   |
{: #table-motion-detector-time-values title="Motion Detector Time Values" }

### Sound maker
The accessory MUST include a sound maker (for example, a speaker) to play sound when in separated state, either periodically or when motion is detected.

It MUST also play sound when a non-owner tries to locate the accessory by initiating a play sound command from a non-owner device when the accessory is in range and connectable through Bluetooth LE.
The sound maker MUST emit a sound with minimum 60 Phon peak loudness as defined by ISO 532-1:2017. The loudness MUST be measured in free acoustic space substantially free of obstacles that would affect the pressure measurement. The loudness MUST be measured by a calibrated (to the Pascal) free field microphone 25 cm from the accessory suspended in free space.

### Non-owner controls
Non-owner controls SHALL use the same service and characteristic UUIDs as defined in [Accessory Connections](#accessory-connections). The non-owner control point enables a non-owner device to locate the accessory by playing a sound. The opcodes for the control point are defined in {{table-non-owner-control-pt-opcodes}}.


|           Opcode           | Opcode  value |           Operands                               | GATT subprocedure           |
|:--------------------------:|:-------------:|:------------------------------------------------:|:---------------------------:|
| Sound_Start                | 0x300         | None                                             | Write; To accessory         |
| Sound_Stop                 | 0x301         | None                                             | Write; To accessory         |
| Command_Response           | 0x302         | [Command Response](#command-response)            | Indications; From accessory |
| Sound_Completed            | 0x303         | None                                             | Indications; From accessory |
| Get_Serial_Number          | 0x404         | None                                             | Write; To accessory         |
| Get_Serial_Number_Response | 0x405         | [Serial Number Payload](#serial-number-payload)  | Indications; From accessory |
{: #table-non-owner-control-pt-opcodes title="Non-Owner Control Point Opcodes"}


This control point SHALL be available to the platform only when the accessory is in separated state. In all other states, the accessory SHALL return the Invalid_command error as the ResponseStatus in CommandResponse. See [Command Response](#command-response) for details.


##### Play sound
The Sound_Start opcode is used to play sound on the sound maker of the accessory. The sound maker MUST play sound for a minimum duration of 5 seconds.

* The accessory SHALL confirm the start of the play sound procedure by sending a [Command_Response](#command-response) with the corresponding CommandOpCode and a ResponseStatus value of Success.

* Once the play sound action is completed, the accessory sends the Sound_Completed message.

* The Sound_Stop opcode is used to stop an ongoing sound request.

* If the sound event is completed or was not initiated by the connected non owner device, the accessory responds with the Invalid_state ResponseStatus code.

* If the accessory does not support the play sound procedure, it responds with Invalid_command ResponseStatus code.

* If a Sound_Start procedure is initiated when another play sound action is in progress, it rejects with Invalid_state error code.

* The accessory SHALL confirm the completion of the stop sound procedure by sending  the Sound_Completed message.


##### Command Response

There are 2 components of the command response operands: CommandOpCode and ResponseStatus. The CommandOpCode operand indicates the procedure that the accessory is responding to and ResponseStatus operand indicates the status of the response.
 The accessory SHALL respond to any invalid opcode with Command_Response and Invalid_command as the ResponseStatus.


| Operand        | Data type | Size (octets) | Description                                                                                                                        |
|----------------|-----------|---------------|------------------------------------------------------------------------------------------------------------------------------------|
| CommandOpCode  | Uint16    | 2             | The control procedure matching this response                                                                                       |
| ResponseStatus | Uint16    | 2             | 0x0000 Success<br/>0x0001 Invalid_state<br/>0x0002 Invalid_configuration<br/>0x0003 Invalid_length<br/>0x0004 Invalid_param<br/>0xFFFF Invalid_command |
{: title="Command Response Operands"}


#### Serial Number Payload
The Get_Serial_Number opcode is used to retrieve serial number lookup payload over Bluetooth LE.
This MUST be enabled for 5 minutes upon user action on the accessory (for example, press and hold a button for 10 seconds to initiate serial number read state).
When the accessory is in this mode, it MUST respond with Get_Serial_Number_Response opcode and Serial Number Payload operand.


|        Operand       | Data type | Size (octets) |        Description                           |
|:--------------------:|:---------:|:-------------:|:--------------------------------------------:|
| p | bytes     |      defined by accessory      | Non-identifiable metadata                      |
| e | bytes     |      defined by accessory      | Encrypted serial number when in paired state.  |
{: #table-sn-payload-over-bt title="Serial Number Payload Over Bluetooth"}

If the accessory is not in serial number read state, it MUST send [Command_Response](#command-response) with the Invalid_command as the ResponseStatus. Further considerations for how these operands should be implemented are discussed in [Design of encrypted serial number look-up](#design-of-encrypted-serial-number-look-up).


### Alternate finding hardware
The accessory SHOULD provide alternate means to help find it, e.g. by vibrating or flashing lights, via a platform-compatible method.

### Recommended Finding Options
{{accessory-finding-hw}} lists two RECOMMENDED options on the set of technology in an accessory to make it findable.


|                      | Option A |  Option B |
|----------------------|:--------:|:---------:|
|                      | Good     | Better    |
| Sound maker          | X        | X         |
| Haptics              |          | X         |
| Lights               |          | X         |
{: #accessory-finding-hw title="Accessory Finding Hardware Options"}

### Future hardware
Future technologies for finding MAY be considered in revisions of this document.

## Disablement
The accessory SHALL have a way to disabled such that its future locations cannot be seen by its owner. Disablement SHALL be done via some physical action (e.g., button press, gesture, removal of battery, etc.).

### Disablement instructions
The accessory manufacturer SHALL provide both a text description of how to disable the accessory as well as a visual depiction (e.g. image, diagram, animation, etc.) that MUST be available when the platform is online and OPTIONALLY when offline. Disablement procedure or instructions CAN change with accessory firmware updates.

### Retrieval
A registry which maps [Product data](#product-data) to an affiliated URL supporting retrieval of disablement instructions SHALL be available for platforms to reference, as defined in {{product-data-registry}}. This URL must return a response which can be rendered by an HTML view.


## Serial Number Identification
The serial number SHALL be printed and be easily accessible on the accessory. The serial number MUST be unique for each product ID.


### Serial number retrieval capability {#serial-number-retrieval}
The serial number payload SHALL be readable either through NFC tap (see [Serial Number payload over NFC](#serial-number-over-nfc)) or Bluetooth LE ( see [Serial Number Payload](#serial-number-payload) ).


### Serial number retrieval over Bluetooth LE
For privacy reasons, accessories that support serial number retrieval over Bluetooth LE MUST have a physical mechanism, for example, a button, that SHALL be required to
enable the Get_Serial_Number opcode, as discussed in [Serial Number Payload](#serial-number-payload).

The accessory manufacturer SHALL provide both a text description of how to enable serial number retrieval over Bluetooth LE, as well as a visual depiction (e.g. image, diagram, animation, etc.) that MUST be available when the platform is online and OPTIONALLY when offline. The description and visual depiction CAN change with accessory firmware updates.

A registry which maps [Product Data](#product-data) to an affiliated URL that will return a text description and visual depiction of how to enable serial number look-up over Bluetooth LE SHALL be available for platforms to reference, as defined in {{product-data-registry}}. This URL MUST return a response which can be rendered by an HTML view.


### Serial number retrieval from a server {#serial-number-from-server}
For security reasons, the serial number payload returned from an accessory in the paired state SHALL be encrypted.

A registry which maps [Product Data](#product-data) to an affiliated URL which will decrypt the serial number payload and return the serial number value
SHALL be available for platforms to reference, as defined in {{product-data-registry}}. This URL MUST return a response which can be rendered by an HTML view.
The arguments sent to this URL SHALL match those that are defined in {{table-sn-payload-over-bt}}.
Security considerations are discussed in {{sn-lookup-security}}.


### Serial number over NFC
For those accessories that support serial number retrieval over NFC, a paired accessory SHALL advertise a URL with parameters in {{table-sn-payload-over-nfc}}.
This URL SHALL decrypt the serial number payload and return the serial number of the accessory in a form that can be rendered in the platform's HTML view.


|        Operand       | Data type | Size (octets) |       Description                           |
|:--------------------:|:---------:|:-------------:|:-------------------------------------------:|
| p | bytes     |      defined by accessory      | Non-identifiable metadata                     |
| e | bytes     |      defined by accessory      | Encrypted serial number when in paired state. |
{: #table-sn-payload-over-nfc title="Serial Number Lookup Payload Over NFC"}



## Pairing registry
Verifiable identity information of the owner of an accessory at time of pairing SHALL be recorded and associated with the serial number of the accessory, e.g., phone number, email address.

### Obfuscated owner information {#obfuscated-owner-info}
A limited amount of obfuscated owner information from the pairing registry SHALL be made available to the platform along with a [retrieved serial number](serial-number-retrieval). This information SHALL be part of the response of the [serial number retrieval from a server](serial-number-from-server) which can be rendered in a platform's HTML view.


This MUST include at least one of the following:

* the last four digits of the owner's telephone number. e.g., (\*\*\*) \*\*\*-5555
* an email address with the first letter of the username and entity visible, as well as the entire extension. e.g., b\*\*\*\*\*\*\*\*@i\*\*\*\*\*.com


### Persistence
The pairing registry SHOULD be stored for a minimum of 25 days after an owner has unpaired an accessory. After the elapsed period, the data SHOULD be deleted.

### Availability for law enforcement
The pairing registry SHALL be made available to law enforcement upon a valid law enforcement request.


# Accessory Category Value
Accessory manufacturer’s MUST pick an accessory category value that closest resembles their physical product.
If none of the accessory categories provided in {{table-accessory-category-values}} match the physical product, Other MUST be chosen.

| Accessory Category Name    | Value       |
|:---------------------------|:-----------:
| Finder                     | 1           |
| Other                      | 128         |
| Luggage                    | 129         |
| Backpack                   | 130         |
| Jacket                     | 131         |
| Coat                       | 132         |
| Shoes                      | 133         |
| Bike                       | 134         |
| Scooter                    | 135         |
| Stroller                   | 136         |
| Wheelchair                 | 137         |
| Boat                       | 138         |
| Helmet                     | 139         |
| Skateboard                 | 140         |
| Skis                       | 141         |
| Snowboard                  | 142         |
| Surfboard                  | 143         |
| Camera                     | 144         |
| Laptop                     | 145         |
| Watch                      | 146         |
| Flash drive                | 147         |
| Drone                      | 148         |
| Headphones                 | 149         |
| Earphones                  | 150         |
| Inhaler                    | 151         |
| Sunglasses                 | 152         |
| Handbag                    | 153         |
| Wallet                     | 154         |
| Umbrella                   | 155         |
| Water bottle               | 156         |
| Tools or tool box          | 157         |
| Keys                       | 158         |
| Smart case                 | 159         |
| Remote                     | 160         |
| Hat                        | 161         |
| Motorbike                  | 162         |
| Consumer electronic device | 163         |
| Apparel                    | 164         |
| Transportation device      | 165         |
| Sports equipment           | 166         |
| Personal item              | 167         |
| Reserved for future use    | 2-127, 168+ |
{: #table-accessory-category-values title="Accessory Category Values"}

# Firmware Updates
The accessory SHOULD have firmware that is updatable by the owner.


# Platform Support for Unwanted Tracking
This section details the requirements and recommendations for platforms to be compatible with the accessory protocol behavior described in the document.

## Paired Accessory Identification
Any platform that supports both pairing and unwanted tracking SHOULD also provide the capability to suppress unwanted tracking alerts caused by an owner device's paired accessory.

If an unwanted tracking alert occurs for an accessory and the platform does not already have the installed capability to prevent this alert for the owner of the accessory, then the platform SHOULD explain to the user how those capabilities can be acquired.


### Implementation
Unwanted tracking SHOULD recognize an accessory paired to that owner device by matching the MAC address advertised, as defined in {{table-payload-format}}, against the one(s) expected during that time.

### Platform Software Extension
Platforms SHOULD implement the paired accessory identification capability as a software extension to its unwanted tracking detection.

Accessory manufacturers SHALL provide this set of MAC addresses to the platform. This set MUST account for the uncertainty involved with the [resolvable and private address](#resolvable-private-address).

The protocol ID in the advertisement payload, as specified in {{table-payload-format}}, SHALL be used to associate an accessory detected with the manufacturer's software extension.

### Network Access
Network access MUST NOT be required in the moment that the platform performs paired accessory recognition.

### Removal
The platform MUST delete any local identifying information associated with an accessory if the manufacturer's software is removed or if the platform unpairs from the accessory.



# Security Considerations

## Serial number look-up {#sn-lookup-security}

Serial number look-up is required to display important information to users who encounter an unwanted tracking notification. It helps them tie the notification to a specific physical device and recognize the accessory as belonging to a friend or relative.

However, the serial number is unique and stable, and the partial user information can further make the accessory identifiable. Therefore, it SHOULD NOT be made directly available to any requesting devices. Instead, several security- and privacy-preserving steps SHOULD be employed.

The serial number look-up SHALL only be available in separated mode for a paired accessory.
When requested through any long range wireless interface like Bluetooth, a user action MUST be required for the requesting device to access the serial number. Over NFC, it MAY be acceptable to consider the close proximity as intent for this flow.

To uphold privacy and anti-tracking features like the Bluetooth MAC address randomization, the accessory MUST only provide non-identifiable data to non-owner requesting devices. One approach is for the accessory to provide encrypted and unlinkable information that only the accessory network service can decrypt. With this approach, the server can employ techniques such as rate limiting and anti-fraud to limit access to the serial number. In addition to being encrypted and unlinkable, the encrypted payload provided by the accessory SHOULD be authenticated and protected against replay. The replay protection is to prevent an adversary using a payload captured once to monitor changes to the partial information associated with the accessory, while the authentication prevents an adversary from impersonating any accessory from a single payload.

### Design of encrypted serial number look-up
One way to design this encryption is for the accessory to contain a public key for the accessory network server. For every request received by a device nearby, the accessory would use the public key and a public key encryption scheme (ie: RSA-OAEP, ECIES, or HPKE) to encrypt a set of fields including the serial number, a monotonic counter or one time token and a signature covering both the serial number and counter or token. The signature can be either a public key signature or symmetric signature, leveraging a key trusted by the network server which MAY be established at manufacturing time or when the user sets up the accessory. Some additional non-identifiable metadata MAY be sent along with this encrypted payload, allowing the requesting device to determine which accessory network service to connect to for the decryption, and for the service to know which decryption key and protocol version to use.


# Privacy Considerations

## Obfuscated owner information
In many circumstances when unwanted tracking occurs, the individual being tracked knows the owner of the location-tracker.
By allowing the retrieval of an obfuscated email or phone number when in possession of the accessory, as described in {{obfuscated-owner-info}}, this
provides the potential victim with some level of information on the owner, while balancing the privacy of accessory owners in the arbitrary situations
where they have separated from those accessories.

## Serial number look-up
A serial number both physically on the device, as well as retrievable over NFC or Bluetooth LE, can aid recourse actions in the case of unwanted tracking.
While retrieval of the serial number over NFC implies having physical possession of the accessory, the same conclusion can not be made for Bluetooth given its wireless range.
The procedure required for serial number look-up over Bluetooth LE intends to strike a balance between the privacy of the owner and ability to empower
potential victims, by requiring both the accessory to be in separated state as well as a physical action be performed to enable the serial number retrieval.

## Location-enabled payload

### Stable identifiers
Rotating the resolvable and private address of the location-enabled payload, as described in {{resolvable-private-address}}, balances the risk of nefarious stable identifier tracking with the need for unwanted tracking detection.
If the address were permanently static, then the accessory would become infinitely trackable for the life of its power source.
By requiring rotation, this reduces the risk of a malicious actor having the ability to piece together long stretches of longitudinal data
on the whereabouts of an accessory.

### Proprietary company payload data
Accessory manufacturers SHOULD evaluate the contents of the proprietary company payload data in {{table-payload-format}} to ensure it does not introduce additional privacy risk through the broadcast of stable identifiers or unencrypted sensitive data.



# IANA Considerations
IANA will create a new registry group called "Unwanted Tracking Protocols (UTP)".
This group includes the "Manufacturer Protocol ID" and "Product Data" registries described below.


## Manufacturer Protocol ID Registry {#manufacturer-protocol-registry}

New entries are assigned only for values that have received Expert Review, per {{Section 4.5 of !RFC8126}}.

An entry in this registry contains the following fields:

* Manufacturer Name: the name of an organization that is producing a location-tracker accessory
* Protocol ID: a 1-byte value specifying the Protocol ID associated with the Manufacturer Name



## Product Data Registry {#product-data-registry}
New entries are assigned only for values that have received Expert Review, per {{Section 4.5 of !RFC8126}}.

There SHALL NOT be two entries in this registry with the same Product Data value. Serial Number Look-up Over Bluetooth Instructions field MAY be
left empty if the accessory does not support that capability.

An entry in this registry contains the following fields:

* Product Data: an 8-byte string representing a unique identifier for a product. See [Product Data](#product-data).
* Disablement Instructions: a string representing the URL where disablement instructions can be retrieved.
* Serial Number Look-up Over Bluetooth Instructions: a string representing the URL where the text instructions and visual depictions for enabling
serial number look-up over Bluetooth LE can be retrieved.
* Serial Number Look-up: a string representing the URL where the serial number and obfuscated owner information can be retrieved.


--- back
