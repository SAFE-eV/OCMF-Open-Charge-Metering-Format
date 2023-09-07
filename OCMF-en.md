# Open Charge Metering Format

**This english version is mainly auto-translated and needs revision.**

Revision: 1.2


## Contributors

| Role          | Responsible                                | Company    |
|---------------|--------------------------------------------|------------|
| Author        | Andreas Mull, ABL (AM), Daniel Müller (DM) | ABL        |
| Editor        | Gerhard Weidinger, Riegler Florian (FR)    | KEBA       |
| Contribution  | Martin Klässner, Roland Angerer            | has.to.be  |
| participation | Andreas Weber                              | Allego     |
| collaboration | Michael Staubermann                        | Webolution |
| collaboration | Mathieu Lémont                             | LEM        |

## Revision overview

Changes from the previous version are marked with change tracking.

|Revision  | Content                                                                                                                                                                                                                                   																																																																																																																																																																																														| Date         
|----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------
| 1.2.0    | Cable loss Compensation data, definition of new parameters used for compensating Energy loss due to EVSE's charging Cable Resistance                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | 05/24/2023/AM
| 1.1.0    | Ad-hoc loading, adding tariff information.                                                                                                                                                                                                                               																																																																																																																																																																																						| 02/09/2023/FR
| 1.0.2    | Removal of the no longer needed OBIS code table, which was needed for the deleted binary format. Add best practice examples to enable consistent implementation and transfer of OCMF records via OCPP.                                                                                                                                                                                                                             																																																																																																																																												            | 02/15/2020/FR
| 1.0.1    | Definition of the meaning of the version numbers. More detailed definition on the structure of the format. Delete binary format and transmission format (transformation is not possible without recalculation of the signature).                                                                                                                                                                                                                             																																																																																																																																									| 02/15/2020/FR
| 1.0      | Final status for integration in transparency software. Feedback from SAFE AG engineering incorporated: Signature algorithm not in data-to-be-signed space after all, so that theoretically multiple different signatures can be appended independently. Reading type optional. Additional state on time status: relative time accounting based on info clock. OBIS codes expanded to applicable options according to comments from Mr. Weber.  | 21.02.2019/AM 																																																																																																																																								| 02/21/2009/AM
| 0.5      | Further feedback from DKE/GAK 461.0.21 AP5 from January 2019 incorporated: Clarifications on pagination, reference of serial numbers, charge point and public key. Optional user data section for charging point assignment. Error index replaced by erroneous quantities; error during charge as reason for reading. Feedback from Mr. Weber on version 0.3 incorporated: Spiking, current types. Byte offset in binary descriptions removed, because they contradicted with StrEnum types. Feedback from Mr. Staubermann on elliptic curve cryptography: synonymous names of curves, correction of block lengths for public key types.                                                                                                                                                                                                                                                                                                                                                                                       | 02/11/2019/AM 
| 0.4      | Results and clarifications after feedback from conference calls with DKE/GAK 461.0.21 AP5: Document and format versions corrected; binary format clarified: Network byte order; pagination count clarified, pagination defined; signature device identification (gateway) placed above meter identification, integrated into general details, vendor renamed to gateway, serial number mandatory for either gateway or meter; user mapping: status split into general yes/no and mapping level, level split into groups; all status details other than yes/no are now optional. Usage of event counter clarified, pagination defined; note on accuracy for measured value; signature algorithm moved to signed area; signature algorithms extended by future candidates, key types analog; defaults provided with OCMF version. Authorization features extended according to OCPP 2.0; extension points for OCMF defined in user data and signature sections. 																	| 01/11/2019/AM 
| 0.3      | State of the meter added for incorrectly read values.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | 12/11/2018/DM 
| 0.2      | Binary format adapted according to the changes with 0.1, description of the signature algorithm concretized, StrEnum type introduced as flexible extension point into the binary format.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       | 08/16/2018/AM 
| 0.1      | This document is based on ABL data format version 1.14 and replaces it. Initial version after discussion of ABL data format (v1.13) with KEBA and has.to.be: decoupling of the binary transmission format from the custody transfer relevant part, JSON as primary transmission format. Header and overall structure defined. Fields aligned with draft "Requirements for the data structure for metering tuples in electromobility" of 26.04.2018. JSON format structurally simplified, keys shortened. Possibility of multiple readings in one data set created. Firmware version of the meter optionally added. Errors as abort reasons for charging processes. Hash type for signature defined.                                                                                                                                                                                                                                                                                                                            | 08/09/2018/AM 



## General

### Goal

The aim of this concept is to describe an independent and generally usable data format for recording meter readings from charging stations that are relevant under calibration law. Furthermore, this concept can serve as a basis for discussion for utilization in the DKE. In addition, it is to serve for the implementation of the evaluation and signature verification of the format by the transparency software to be created within the SAFE initiative.

At the time of writing, no uniform data format for the transmission of signed meter readings has been adopted at the standardization level. A corresponding application guideline is currently being developed. The presented data format originates from an internal draft of ABL and was discussed with has.to.be and KEBA.

A uniform format brings the advantage of interoperability and enables uniform verification software at the operator and the end user. The concept presented here is based on the end-to-end security concept for a custody transfer ("GL") solution for the transmission of measurement data to remote displays by Dr. Martin Kahmann [MEMOGL]. It further builds on ABL's internal concept for legal metrology compliance.

This document is structured as follows: First, a classification and presentation of the transmission of the
data format is given, in the further sections the data format is explained.


### License

This document is licensed under the *Creative Commons Attribution-NoDerivatives 4.0 International Public License* (https://creativecommons.org/licenses/by-nd/4.0/legalcode).


### References

 - OCPP 1.5: Open Charge Alliance: Open Charge Point Protocol - Interface description between Charge Point and Central System, Version 1.5, as of June 8, 2012. 
 - MEMO-GL: Measurement systems for charging equipment: End-to-end security concept for a custody transfer ("GL") solution for the transmission of measurement data to remote displays, Dr. Martin Kahmann, Braunschweig, Germany. 
 - OBIS: IEC 62056-6-1 and IEC 62056-6-2.
 - JSON: Introducing JSON, https://www.json.org/json-de.html (retrieved: 2018-04-04).


### Abbreviations used

* API: Application Programming Interface
* ASCII: American Standard Code for Information Interchange
* CDR: Charge Data Record
* CPO: Charge Point Operator, Betreiber der Ladestation
* DKE: Deutsche Kommission Elektrotechnik Elektronik Informationstechnik (German Commission for Electrical, Electronic & Information Technologies)
* ECDSA: Elliptic Curve Digital Signature Algorithm
* EMP: Electric Mobility Provider
* ISO: International Standards Organisation
* MID: Measuring Instruments Directive, europäische Richtlinie 2004/22/EG
* OBIS: Object Identification System
* OCPP: Open Charge Point Protocol
* PIN: Personal Identification Number
* PTB: Physikalisch-Technische Bundesanstalt (German national metrology institute)
* RFC: Request For Comments
* RFID: Radio Frequency Identification
* RTU: Remote Terminal Unit
* SHA: Secure Hash Algorithm
* TLS: Transport Layer Security
* UID: Unique Identification
* UTC: Universal Time Coordinated

### Versioning of the document

The version number of this document is structured as follows:

**Major Version:**
Changes at this point mean that the format has changed fundamentally. Changes have been made to the format that are no longer compatible with previous versions.

**Minor:**
The format has been supplemented new fields or the values of a field have been added 
(Changes to the format. Backward compatibility is given).

**Revision:**
Change to the documentation. No change to the format. Increasing the revision does not change the format itself. The documentation is adapted or improved.

Example: 1.0.2
1 = Major
0 = Minor
2 = Revision


## Transmission of signed meter readings

### Embedding in OCPP

OCPP version 1.5 already allows the simultaneous transmission of any number of MeterValue objects as part of the MeterValue.req and StopTransaction.req messages. Each such object may in turn have a timestamp noted along with one or more meter reading values. Each value may optionally be accompanied by attributes, one of which is of the ValueFormat enum type, which has two possible values:

 * Raw: Data is to be interpreted as integer/decimal numeric data.
 * SignedData: Data is represented as a signed binary data block, encoded as hex data.

_Source: [OCPP 1.5]_

This makes it possible to continue to use the functionality provided by OCPP and to combine it with the possibility of transmitting signed meter values.

The data format presented here is therefore intended for parallel transmission of such a SignedData "value" in addition to the previous raw values.

While the reading of the previous values may be based on a time that does not conform to the calibration law (OCPP-based), the signed meter values are based on a time that does not conform to the calibration law.
however, the signed meter values are based on one. Even the two resulting times can be correctly represented separately in two MeterValue objects.

This mechanism is used in the context of:

 - Start of a transaction (StartTransaction)
 - Readings during a transaction (MeterValues)
 - End of transaction (StopTransaction)
 - Charge Data Record over the whole transaction with values at start and end (optional)
 - Readings for so-called "Fiscal Metering" (MeterValuesAlignedData): Here, no reference to ongoing transactions is recorded, but only the absolute meter values of all charge points at previously determined points in time.

It should be noted that, as a rule, the following readings are relevant under calibration law:

 - Start and end value in relation to the charging transaction
 - Tariff change (common practice only at the quarter hour of the clock).

Thus the following OCPP configuration is typically useful:

 - ClockAlignedDataInterval=900 (15min)
 - MeterValuesAlignedData=Active.Energy.Register.Import
 - Controller software takes care of generating additional MeterValues with signed values at the start and end of the charging process beyond the simple integer value that can be part of the StartTransaction/StopTransaction messages.

### Signing and verification process

In order to verify a data set, the data that has been signed must first be restored to its original form.
original form. A hash calculation is performed on this original form. The result is verified via the digital signature using ECDSA.

To produce the original form, the header and signature are separated from the data record. If necessary, the public key is verified via a separate transmission path.

This means that the signature and the public key are shown separately in the unified data record.

The cohesion between several individual data records is ensured by continuous pagination.
is guaranteed. In addition to the signature, this must be verified by a check component. The first record
must be marked as the start of a loading process, the last as the end of the loading process. In between
no data records may have been removed or added. Likewise, intermediate
error conditions (error counters) and the detection of unusable variables must lead to an error during the
test. Furthermore, all data records must come from the same source (serial number).



## Data format
The data format should be as simple as possible and comprehensible by the experienced user. From
a modular data format is desirable, so that future extensions can be implemented.
can be implemented. Another concern is to make the verification process as generic as possible, i.e. also
applicable to other source formats. For this reason, a header section was included in the format.

The format consists of several sections:

1. **Header** to uniquely identify the format.
2. section containing the actual **payload data**.
3. **Signature** over the payload data

The contents of the record serve as the input format into the [Link](https://transparenz.software/ transparency software). The [Link](https://transparenz.software/ transparency software) is available online for anyone to use to validate the dataset.

In the following chapters, the data format for the representation and transmission of the signed meter values is defined.

## JSON-based OCMF format

JSON notation was chosen for the individual sections of the format for better readability and easier traceability. To save data volume, the following measures were taken:

 - Identifiers are kept short (since some need explanation anyway, not much clarity is lost here
lost)
 - In some places encapsulation was deliberately omitted. Instead, the identifiers are canonicalized.

The sections within the data format are separated with the special character "|". Within the sections the format corresponds to the JSON notation. Thus, the use of the special character "|" is not allowed within the sections.

The fact that the format can be enriched according to the JSON notation with spaces and line feeds at
can be enriched with spaces and line feeds at any position, called white space, for display purposes.
also makes it easier for a user to read.

Between signing and validation, the payload section must not be manipulated (removing and adding white spaces), otherwise positive validation is not possible.

In the following, the different sections of the format are explained. Furthermore, an example is given.

### Sections

At the beginning of the format there is a header for unique identification of the format.
The user data section and the signature section follow, separated by pipe characters.
This results in the following structure for the sections:

    OCMF|<payload data section>|<signature section>

The words payload data section and signature section are to be replaced by the contents of the respective sections.


#### Header

The header is used to uniquely identify the transmission format. In the case of OCMF, the string "OCMF" must be specified in this section.

#### Payload data section

The payload data section consists of a JSON object. This contains several value mappings or
sub-objects. The contents of the payload data section must not be modified between signing and verification of the signature.

##### General information

The information in this section refers primarily to the signature-generating component, which is referred to here as the gateway for the sake of simplicity. Depending on the structure, it may be more appropriate to refer to the measuring capsule or the signing component.

| Key | Typ    |Cardi-nality| Description                                                                                                                                                                                                                                                                             |
|------|--------|-----|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FV | String |0..1 | Format-Version: Version of the data format in the representation <major>.<minor> The version specification is coded according to the version of this document, i.e. 0.4 corresponds to major 0 and minor 4. The revision (third digit) is not transmitted, since this does not change anything in the format itself. |
| GI | String |0..1 | Gateway-Identification: Identifier of the manufacturer for the system which has generated the present data (manufacturer, model, variant, etc.).                                                                                                                                          |
| GS | String |0..1 | Gateway serial: Serial number of the above system. This field is conditionally mandatory.                                                                                                                                                                                                         | 
| GV | String |0..1 | Gateway version: Version designation of the manufacturer for the SW of the above system. This field is optional. |
<small>Table 1: General information fields</small>

##### Pagination

The pagination indicates the reference to a transaction and arranges the total data record in the stream of the
readings via the meters in a comprehensible way. Several contexts are available for pagination,
each context has its own counter reading. The respective pagination counter is counted monotonically in ascending order with an increment of 1. That means, successive values of the pagination counter count continuously in the space of natural numbers.

The transaction context (T) must be supported in any case. The fiscal context (F) is optional and can be supported by the signature component as an option if signed readings outside of transactions are desired.


| Key | Type |Cardi-nality| Description |
|------|--------|-----|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PG | String |1..1 | Pagination of the entire data set, i.e. the data that are combined in one signature. Format: `<indicator><number>` <br> The string is composed of an identifying letter for the context and a number without leading zeros. There is a separate independent pagination counter for each context. The following indicators are defined: <br>  T: Transaction – Readings in transaction reference (mandatory) <br> F: Fiscal – Readings independent of transactions (optional) <br> The respective pagination counter is incremented after each use for a record.|
<small>Table 2: Pagination field</small>

##### Meter identification

The meter identification is optional if the signature generating component has already been identified in the general data.

| Key  | Typ    |Cardinality| Description                                                              			|
|------|--------|-----------|-----------------------------------------------------------------------------------|
| MV   | String |0..1      	| Meter-Vendor: Manufacturer identification of the meter, name of the manufacturer. |
| MM   | String |0..1 		| Meter-Model: Model identification of the meter.                              		|
| MS   | String |1..1 		| Meter-Serial: Serial number of the meter .                              			|
| MF   | String |0..1 		| Meter-Firmware: Firmware-Version of the meter.                              		|

<small>Table 3: Fields for identification of the meter</small>

##### User assignment

Fields belonging to this are included exactly when there is a transaction reference. Even if no user can be transaction reference no user can be assigned yet, the section is included. If there is no transaction reference exists, this group of fields is missing.


| Key | Typ                |Cardinality | Description                                                                                                                                                                                                                                                  |
|------|------------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IS   | Boolean          |1..1 		| Identification status: General status for user assignment:<br>true: user successfully assigned, <br>false: User not assigned.                                                                                                                                |
| IL   | String           |0..1 		| Identification-Level: Encoded overall status of the user assignment, represented by an identifier from Table 10.                                                                                                                                             |
| IF   | Array of  String |0..4 		| Identification flags: Detailed statements about the user assignment, represented by one or more identifiers from Table 12 to Table 15. The identifiers are always noted as string elements in an array. Also one or no element must be noted as an array.    |
| IT   | String           |1..1 		| Identification-Type: Type of identification data, identifier see table 16.                                                                                                                                                                                   |
| ID   | String           |0..1 		| Identification-Data: The actual identification data according to the type from table 16, e.g. a hex-coded UID according to ISO 14443.                                                                                                                        |
| TT   | String (0..250)  |0..1 		| TariffText: Text used to identify a unique tariff. This field is intended for the tariff designation in "Direct Payment" use case.                                                                                                                           |
<small>Table 4: Fields for user assignment</small>

##### EVSE Metrologic parameters

Fields described in this section trace EVSE parameters that are influent for metrology.

| Key | Typ                |Cardinality | Description                                                                                                                                                                                                                                                  |
|------|------------------|-------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LC   | object           |0..1 		| Loss Compensation: Characteristics of EVSE's charging cable used for identifying and processing cable loss compensation algorithm. Parameters of these characteristics are defined in Table 23                                                               |
<small>Table 24: Fields for EVSE Metrologic parameters</small>

##### Assignment of the charge point

This optional user data section provides a way to identify the charging point. This can be an alternative to identification by serial numbers or can be used additionally.

| Key | Typ   |Cardi-nality| Description                                                                                                                |
|------|--------|-----|---------------------------------------------------------------------------------------------------------------------------------|
| CT   | String |0..1 | Charge-Point-Identification-Type: Type of the specification for the identification of the charge point, identifier see table 17 |
| CI   | String |0..1 | Charge-Point-Identification: Identification information for the charge point.                                                   |
<small>Table 5: Fields for identification of the charge point</small>

##### Readings

One or more readings can be stored in a record. Each reading is encapsulated in a sub-object. These objects are noted as an array under the key "RD" for Reading(s). This array thus contains one or more anonymous objects.
Each of these objects is constructed as described in Table 6.
For the readings, fields that have an identical value to the previous reading are omitted. However, this only applies within a signed record. This means concretely that, for example, only in the first sub-object the field "RI" is defined with a value (here the OBIS code) and is thus valid for all further readings. Likewise, for example, the field "TX" can be defined in the first sub-object as `B`, in the second as `C`, in the last with `E`, which means that the readings three to the penultimate take the value `C`.
The fields `RV`, `RI` , `RU` and `RT` can be omitted if only the occurrence of an error condition (event) of the meter is to be indicated.
The fields `RI` and `RU`, form a group. Fields of a group are either all present together or omitted together. 
A power failure during time-based billing represents an error event.

| Key | Typ  |Cardi-nality| Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
|------|--------|----|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TM   | String |1..1| Time: Specification to the system time of the reading and synchronization state. <br> The time is described according to ISO 8601 with a resolution of milliseconds. Accordingly, the format is formed according to the following scheme: <br> `<Year>-<Month>-<Day>T<Hour>:<Minute>:<Second>,<Millisecond><Time zone>` <br> The year is displayed with four digits. Month, day, hour, minute and second is displayed with two digits. Millisecond is displayed with three digits. The indication of the time zone consists of a sign and a four-digit indication for hour and minute. Example: <br> `2018-07-24T13:22:04,000+0200` <br> The synchronization state consists of a capital letter as identifier. This is added to the time, separated by a space. Available states see table 18.                                                |
| TX   | String |0..1| Transaction: Meter reading reason, reference of meter reading to transaction, noted as capital letter: <br> B – Begin of transaction <br> C – Charging = during charging (can be used optionally) <br> _ X – Exception = Error during charging, transaction continues, time and/or energy are no longer usable from this reading (incl.). <br> E – End of transaction, alternatively more precise codes: <br> _ L – Loading process was terminated locally <br> _ R – Loading process was terminated remotely <br> _ A – (Abort) Loading process was aborted by error <br> _ P – (Power) Charging process was terminated by power failure <br> S – Suspended = Transaction active, but currently no charge (can be used optionally) <br> T – Tariff change <br> This field is missing if there is no transaction reference. (Fiscal Metering) |
| RV   | Number |1..1| Reading-Value: The value of the reading <br>Here the JSON data format Number is used, this allows among other things an exact marking of the valid decimal places. However, the representation must not be transformed by further handling methods (e.g. processing by JSON parser) (rewriting the number with a different exponent, truncation of decimal places, etc.) since this would change the representation of the physical quantity and thus potentially the number of valid digits. According to the application rule, it is recommended to represent the measured value with two decimal places of accuracy, if it is kWh.                                                                                                                                                                                                         |
| CL   | Number |0..1| Cumulated-Loss: This parameter is optional and can be added only when RI is indicating an accumulation register reading. The value reported here represents cumulated loss withdrawned from measurement when computing loss compensation on RV. CL must be reset at TX=B. CL is given in the same unit as RV which is specified in RU.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| RI   | String |0..1| Reading-Identification: Identifier, which quantity was read, according to OBIS code.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RU   | String |1..1| Reading-Unit: Unit of reading, e.g. kWh, according to Table 19: Predefined units.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| RT   | String |0..1| Reading-Current-Type: The type of current measured by the meter, e.g. alternating current or direct current, according to Table 20: Predefined current types. <br> This field is optional. No default value is defined.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| EF   | String |0..1| Error-Flags: Statement which quantities are no longer usable for billing due to an error. Each character in this string identifies a quantity. The following characters are defined: <br>  E – Energy<br>  t – Time|                            																																																																																																																																																 |
| ST   | String |1..1| Status: state of the meter at the time of reading. Noted as abbreviation according to Table 9.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
<small>Table 6: Fields of a meter reading object</small>

##### Extension points in payload data section

In the payload data section it may be necessary for different manufacturers to be able to add their own data.
their own data. For this purpose, the following extension points are reserved in the namespace of the JSON object for free use depending on the manufacturer:

 * Reserved are all top-level names in the JSON object of the user data section, which start with the following letters:
   - U
   - V
   - W
   - X
   - Y
   - Z
 * Since no changes or additions can be made subsequently in the user data section (sic), no corresponding extension points are defined. [See below](#extension points signature section).


#### Signature section

The signature section also consists of a standalone JSON object. It contains several value mappings related to the signature data.

##### Signaturdaten

| Key: | Typ:   |Card| Description:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|------|--------|----|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| SA   | String |1..1| Signature-Algorithm: Selects the algorithm used to create the signature. This includes the signature algorithm, its parameters, and the hash algorithm that will be applied to the data to be signed. This specification is optional. If it is omitted, the default value is valid. Table 21 shows which values are possible.                                                        |
| SE   | String |0..1| Signature-Encoding: Indicates how the signature data is encoded to be stored in the JSON string. This specification is optional. If it is omitted, the default value is valid. The following values are possible: <br>hex – The signature data is represented in the JSON string in hexadecimal encoding (default) <br>base64 – The signature data is stored according to base64     |
| SM   | String |0..1| Signature-Mime-Type: Indicates how the signature data is to be interpreted. This specification is optional. If it is omitted, the default value is valid. The following values are possible:<br>application/x-der – Filing as ASN.1-coded .der-file (Default)                                                                                                                        |
| SD   | String |1..1| Signature-Data: The actual signature data according to the above format specifications.                                                                                                                                                                                                                                                                                              |
<small>Table 7: Signature data fields of the signature section</small>

##### Extension points

Analogous to the extension points in section 0, unprotected extension points may also be
desirable. The following are reserved for this purpose:

 * Reserved for vendor use are all top-level names in the signature data section JSON object that begin with the following letters:
	- U
	- V
	- W
	- X
	- Y
	- Z
 * The following initial letters are reserved accordingly for use by components in subsequent processing of data:
     - A
	 - B
	 - C
	 - D
	 - E
	 - F

#### Reference of serial numbers, charge point and public key

To establish a reference between the public key to be used for signature verification and the
one or more serial numbers; alternatively, the loading point can also be identified directly by ID.
alternatively, the charging point can be identified directly by ID.

The public keys to the charging point must be transmitted to the verification component by means other than this protocol (out-of-band) in conjunction with the ID or serial numbers used, e.g. via a central register.

To uniquely identify a signature component, the following characteristics are defined in this protocol:

 - If the signature component is attributable to the energy meter, its serial number is sufficient.
 - If the signature component itself is the gateway and only one charging point is served, the serial number of the gateway or the energy meter is sufficient.
 - If a gateway is responsible for more than one charging point, the combination of the serial numbers of the gateway and energy meter is used for unique identification.
 - Alternatively, a direct identification of the charge point can be made.

Since these conditions depend on the design of the charging station, the corresponding serial number fields are marked below as conditionally mandatory.

### Example

Examples of the resulting JSON-based formats:

#### Example file display format

    OCMF|{
        "FV": "1.0",
        "GI": "ABL SBC-301",
        "GS": "808829900001",
        "GV": "1.4p3",
        "PG": "T12345",
        "MV": "Phoenix Contact",
        "MM": "EEM-350-D-MCB",
        "MS": "BQ27400330016",
        "MF": "1.0",
        "IS": true,
        "IL": "VERIFIED",
        "IF": [
            "RFID_PLAIN",
            "OCPP_RS_TLS"
        ],
        "IT": "ISO14443",
        "ID": "1F2D3A4F5506C7",
		"TT": "Tarif 1",
        "LC": {
            "LN": "cable_name",
            "LI": 1,
            "LR": 2,
            "LU": "mOhm"
            }
        "RD": [
            {
                "TM": "2018-07-24T13:22:04,000+0200 S",
                "TX": "B",
                "RV": 2935.6,
                "RI": "1-b:1.8.0",
                "RU": "kWh",
                "RT": "DC",
                "EF": "",
                "ST": "G"
            },
            {
                "TM": "2018-07-24T13:26:04,000+0200 S",
                "TX": "E",
                "RV": 2965.1,
                "CL": 0.5,
                "RI": "1-b:1.8.0",
                "RU": "kWh",
                "RT": "DC",
                "EF": "",
                "ST": "G"
        ]
    }|{
    "SD":  "887FABF407AC82782EEFFF2220C2F856AEB0BC22364BBCC6B55761911ED651D1A922BADA88818C9671AFEE7094D7F536"
    }

<small>Figure 1: Example file display format</small>


## Further data fields

### Meter identification

#### Manufacturer and model of the meter

The following table represents the predefined values for the meter manufacturer and model fields.

| Manufacturer:   | Modell:                 |
|-----------------|-------------------------|
| Phoenix Contact | EEM-350-D-MCB           |
| Inepro Metering | Pro 1 Mod Pro 380 Mod   |
| Carlo Gavazzi   | EM340-DIN.AV2.3.X.S1.PF |
|                 | EM111-DIN.AV8.1.X.S1.PF |
|                 | EM210-72D.MV5.3.X.OS.X  |
| NZR             | EcoCount S 85           |
<small>Table 8: Values for manufacturer/model of the meter</small>

#### State / error of the counter

This field can take one value at the same time.

| Token | Identifier | Description                                                                          |
|---------|--------------|----------------------------------------------------------------------------------|
| N       | NOT_PRESENT  | The counter is not present or has not been found                                 |
| G       | OK           | Counter in order (Good)                                                          |
| T       | TIMEOUT      | Timeout when trying to control the counter                              			|
| D       | DISCONNECTED | Counter was disconnected from the signature component                            |
| R       | NOT_FOUND    | Counter no longer found (has already been found at least once before) (Removed)  |
| M       | MANIPULATED  | Manipulation detected                                                            |
| X       | EXCHANGED    | Meter exchanged (serial number no longer matches the last known one)             |
| I       | INCOMPATIBLE | Meter version or its API not compatible with signature component                 |
| O       | OUT_OF_RANGE | Read value outside the value range                                               |
| S       | SUBSTITUTE   | A substitute value was formed                                                    |
| E       | OTHER_ERROR  | Other, not further known error                                                   |
| F       | READ_ERROR   | Meter register not read correctly; value of the readout is not valid             |
<small>Table 9: State / error of the counter</small>

### User assignment

## State / Error

This field can take only one value at a time. For clarity, the values are divided into several
groups, which are not distinguished in the format.

| Group:                    | Status/Error:  | Description: 																															  |
|---------------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| not available             | -              | The field is not specified.                                                                                                                |
| Status without assignment | NONE 			 | There is no user assignment. The other data for user assignment have no significance.                                                      |
| Status with assignment    | HEARSAY 		 | The assignment is unsecured; e.g., by reading an RFID UID.                                                                                 |
| 							| TRUSTED 		 | The mapping can be trusted to some extent, but there is no absolute reliability. Example: Authorization by backend.                        |
| 							| VERIFIED 		 | The assignment has been verified by the signature component and special measures.                                                          |
| 							| CERTIFIED 	 | The assignment was verified by the signature component using a cryptographic signature that certifies the assignment.                      |
|  							| SECURE 		 | The mapping was established by a secure feature (e.g. secure RFID card, ISO15118 with plug and charge, etc.).                              |
| Error 					| MISMATCH 		 | Error; UIDs do not match.                                                                                                                  |
| 							| INVALID 		 | Error; certificate not correct (check negative).                                                                                           |
| 							| OUTDATED 		 | Error; referenced trust certificate expired.                                                                                               |
| 							| UNKNOWN 		 | Certificate could not be successfully verified (no matching trust certificate found).                                                      |
<small>Table 10: Status and error states of user assignment</small>

### Details

This field is composed of four sub-fields and gives details about the status of the user assignment.
Table 11 describes the division into subfields, the following tables describe the valid values.
valid values.

#### Allocation Details User Assignment

| Content                                                    |For details see |
|------------------------------------------------------------|----------------|
| [Allocation share RFID](#rfid)                             | Table 12       |
| [Allocation share OCPP](#ocpp)                             | Table 13       |
| [Allocation share ISO 15118](#iso15118)                    | Table 14       |
| [Allocation share PLMN](#plmn)                             | Table 15       |
<small>Table 11: Overview of Allocation Details User Assignment</small>

#### [Allocation share RFID](#rfid)

| Identifier   | Description                                                                                                       |
|--------------|-------------------------------------------------------------------------------------------------------------------|
| RFID_NONE    | No assignment via RFID                                                                                            |
| RFID_PLAIN   | Assignment via external RFID card reader                                                                          |
| RFID_RELATED | Assignment via protected RFID card reader                                                                         |
| RFID_PSK     | A previously known shared key (pre-shared key) was used, e.g. with a secured RFID card. 						   |
<small>Table 12: Allocation share RFID </small>

#### [Allocation share OCPP](#ocpp)

| Identifier:    | Description:                                                                                 |
|----------------|----------------------------------------------------------------------------------------------|
| OCPP_NONE      | No user assignment by OCPP                                                                   |
| OCPP_RS        | Assignment by OCPP RemoteStart method                                                        |
| OCPP_AUTH      | Assignment by OCPP Authorize method                                                          |
| OCPP_RS_TLS    | Transport layer security was used to transfer the mapping by OCPP-RemoteStart method.		|
| OCPP_AUTH_TLS  | Transport layer security was used to transfer the mapping by OCPP authorize method.   		|
| OCPP_CACHE     | Assignment by authorization cache of OCPP                                                    |
| OCPP_WHITELIST | Assignment by white list from OCPP                                                           |
| OCPP_CERTIFIED | A certificate of the backend which certifies the user mapping was used.             			|
<small>Table 13: Allocation share OCPP</small>

#### [Allocation share ISO 15118](#iso15118)

| Identifier    | Description                       |
|---------------|-----------------------------------|
| ISO15118_NONE | no user assignment by ISO 15118   |
| ISO15118_PNC  | Plug & Charge was used            |
<small>Table 14: Allocation share ISO 15118</small>

#### [Allocation share PLMN](#plmn)

| Identifier | Description      |
|-------------|-----------------|
| PLMN_NONE   | no assignment   |
| PLMN_RING   | call            |
| PLMN_SMS    | short message   |
<small>Table 15: Allocation share PLMN</small>

### Type

Table 16 describes the possible types of user mappings, which can be used in the Type field as
unsigned integer in the Type field. These mappings are based on the specifications from
OCPP. OCPP currently (version 1.5) only provides 20 characters for the identification feature.
In accordance with the maximum length of an IBAN (34 characters) allocations of up to 40 bytes have been provided for the data area.

|Token    | Identifier   | Description 																								   			|
|---------|--------------|----------------------------------------------------------------------------------------------------------------------|
| 0 	  | NONE 		 | No assignment available                                                                                              |
| 1		  | DENIED 		 | Assignment currently not available (due to two-factor authorization)                                                 |
| 2 	  | UNDEFINED    | Type not specified                                                                                                   |
| 10 	  | ISO14443 	 | UID of an RFID card according to ISO 14443. Represented as 4 or 7 bytes in hexadecimal notation.                     |
| 11	  | ISO15693 	 | UID of an RFID card according to ISO 15693. Represented as 8 bytes in hexadecimal notation.                          |
| 20 	  | EMAID 		 | Electro-Mobility-Account-ID according to ISO/IEC 15118 (string with length 14 or 15)                                 |
| 21      | EVCCID 		 | Electric vehicle account ID according to ISO/IEC 15118 (maximum length 6 characters)                                 |
| 30      | EVCOID 		 | EV Contract ID according to DIN 91286.                                                                               |
| 40      | ISO7812 	 | Identification card format according to ISO/IEC 7812 (credit and bank cards, etc.)                                   |
| 50      | CARD_TXN_NR  | Card transaction number (CardTxNbr) for a payment with credit or bank card used in a terminal at the charging point. |
| 60      | CENTRAL 	 | Centrally generated ID. No exact format defined, can be e.g. a UUID. (OCPP 2.0)                                      |
| 61 	  | CENTRAL_1 	 | Centrally generated ID, e.g. by start via SMS. No exact format defined. (until OCPP 1.6)                             |
| 62 	  | CENTRAL_2 	 | Centrally generated ID, e.g. by operator start. No exact format defined. (until OCPP 1.6)                            |
| 70 	  | LOCAL 		 | Locally generated ID. No exact format defined, may be e.g. a UUID. (OCPP 2.0)                                        |
| 71      | LOCAL_1 	 | Locally generated ID, e.g. ID generated internally by the charge point. No exact format defined. (until OCPP 1.6)    |
| 72 	  | LOCAL_2 	 | Locally generated ID, for other cases. No exact format defined. (until OCPP 1.6)                                     |
| 80 	  | PHONE_NUMBER | International phone number with leading "+".                                                                         |
| 90 	  | KEY_CODE 	 | User-related private key code. No exact format defined.                                                              |
<small>Table 16: User assignment types</small>

### Charging point assignment

| Identifier | Description                                                                                                 |
|-------------|------------------------------------------------------------------------------------------------------------|
| EVSEID      | EVSE-ID                                                                                                    |
| CBIDC       | Charge box ID and connector ID according to OCPP, a space is used as field separator, e.g. „STEVE_01 1“.   |
<small>Table 17: Charging point assignment types</small>

### Time status

| Identifier  | Description                                                                                                                                                                                                                                                                                               |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| U           | unknown, unsynchronized                                                                                                                                                                                                                                                                                   |
| I           | informative (info clock)                                                                                                                                                                                                                                                                                  |
| S           | synchronized                                                                                                                                                                                                                                                                                              |
| R           | relative time accounting with a custody transfer accurate timer based on an info clock. I.e., at the beginning of the charging process, a time with informative quality was available. During the charging process, the time (duration) was recorded in accordance with the legal metrology requirements. |
<small>Table 18: Time status</small>


### Units

| Unit     |
|----------|
| kWh      |
| Wh       |
<small>Table 19: Predefined units</small>

### Current types
| Identifier  | Description            |
|-------------|------------------------|
| AC          | AC current measurement |
| DC          | DC current measurement |
<small>Table 20: Predefined current types</small>

### Cryptography

#### Signature methods
| Identifier 											| Signature algorithm 	  | Curve name and synonyms, if any                              | Key length         | Hash algorithm      | Block length  |
|-------------------------------------------------------|-------------------------|--------------------------------------------------------------|--------------------|---------------------|---------------|
| ECDSA-secp192k1-SHA256                                | ECDSA                   | secp192k1                                                    | 192 bit            | SHA-256             | 48            |
| ECDSA-secp256k1-SHA256                                | ECDSA                   | secp256k1                                                    | 256 bit            | SHA-256             | 64            |
| ECDSA-secp192r1-SHA256                                | ECDSA                   | secp192r1                                                    | 192 bit            | SHA-256             | 48            |
| ECDSA-secp256r1-SHA256  (Default in OCMF Version 0.4) | ECDSA                   | secp256r1,  NIST P-256, ANSI X9.62 elliptic curve prime256v1 | 256 bit            | SHA-256             | 64            |
| ECDSA-brainpool256r1-SHA256                           | ECDSA                   | brainpool256r1                                               | 256 bit            | SHA-256             | 64            |
| ECDSA-secp384r1-SHA256                                | ECDSA                   | secp384r1,  NIST P-384, ANSI X9.62 elliptic curve prime384v1 | 384 bit            | SHA-256             | 96            |
| ECDSA-brainpool384r1-  SHA256                         | ECDSA                   | brainpool384r1                                               | 384 bit            | SHA-256             | 96            |
<small>Table 21: Predefined signature algorithms and their parameters</small>

#### Public-Key-Types

|Value  | Identifier 									 | Signature Algorithm 	   | Curve          | Key length           | Block length  |
|-------|------------------------------------------------|-------------------------|----------------|----------------------|---------------|
| 1     | ECDSA-secp192k1                                | ECDSA                   | secp192k1      | 192 bit              | 48            |
| 2     | ECDSA-secp256k1                                | ECDSA                   | secp256k1      | 256 bit              | 64            |
| 3     | ECDSA-secp192r1                                | ECDSA                   | secp192r1      | 192 bit              | 48            |
| 4     | ECDSA-secp256r1  (Default in OCMF Version 0.4) | ECDSA                   | secp256r1      | 256 bit              | 64            |
| 5     | ECDSA-brainpool256r1                           | ECDSA                   | brainpool256r1 | 256 bit              | 64            |
| 6     | ECDSA-secp384r1                                | ECDSA                   | secp384r1      | 384 bit              | 96            |
| 7     | ECDSA-brainpool384r1                           | ECDSA                   | brainpool384r1 | 384 bit              | 96            |
<small>Table 22: Predefined key types</small>
Note: The term "Key length" refers to the key length of the corresponding private key

### Cable Loss Compensation parameters
The cable Loss data consists in an object under the key "LC". It shall contain Cable Resistance value and may contain optional traceability parameters, as explained in following table.

| Key  | Typ            | Cardinality | Description                                                                                                                                                                                                                                                                                                                                                                                                                                         |
|------|----------------|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| LN   | String (0..20) |0..1         | Loss compensation Naming: This parameter is optional. A meter can use this value for adding a traceability text for justifying cable Loss characteristics.                                                                                                                                                                                                                                                                                          |
| LI   | Number         |0..1         | Loss compensation Identification: This parameter is optional. A meter can use this value for adding a traceability ID number for justifying cable Loss characteristics from an Look up table specified in meter's documentation.                                                                                                                                                                                                                    |
| LR   | Number         |1..1         | Loss compensation cable Resistance: This parameter is mandatory. A meter shall use this value for specifying the Cable Resistance Value used in cable Loss compensation computation. The unit of this value shall be specified in Meter's documentation.                                                                                                                                                                                            |
| LU   | Number         |1..1         | Loss compensation cable resistance Unit: This parameter is mandatory. A meter shall use this field for specifying the Unit of Cable Resistance Value given by LR field used in cable loss compensation computation. The unit of this value can be traced in OCMF format in addition to Meter's documentation. Allowed values are milliohm or microohm; LU value for milliohm shall be "mOhm" or "mΩ"; LU value for microohm shall be "uOhm" or "µΩ".|
<small>Tabelle 23: Cable Loss Compensation parameters</small>

## Best practice - Additional OCPP configuration keys
To inform an OCPP backend about the behavior of a charging station the following additional OCPP configuration keys are recommended by S.A.F.E.


Additional Configuration Keys für OCPP:
* **StopTransactionSignatureFormat**: A signature over start and stop data OR separated (can be R or RW, depending on the implementation) This setting can take the following values:
    * **MR** MultipleReading (Start and stop in an OCMF data set)
    * **SR** SingleReading  (Start and stop in two separate OCMF data sets)
* **StopTransactionSignatureContexts**: ReadingContext from OCPP (CSL, RW). If the list is empty, all transaction records are transferred unsigned.
* **MeterValuesSignatureContexts**: Meter Values Trigger points for which signed OCMF records are to be generated.  Possible values see ReadingContext from OCPP (CSL, RW). If the list is empty, all MeterValues records are transmitted unsigned.
