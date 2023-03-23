# Open Charge Metering Format

**DRAFT**

**This english version is mainly auto-translated and needs revision.**

Revision: 1.0.1

## General

### Goal

The aim of this concept is to describe an independent and generally usable data format for recording meter readings from charging stations that are relevant under calibration law. Furthermore, this concept can serve as a basis for discussion for utilization in the DKE. In addition, it is to serve for the implementation of the evaluation and signature verification of the format by the transparency software to be created within the SAFE initiative.

At the time of writing, no uniform data format for the transmission of signed meter readings has been adopted at the standardization level. A corresponding application guideline is currently being developed. The presented data format originates from an internal draft of ABL and was discussed with has.to.be and KEBA.

A uniform format brings the advantage of interoperability and enables uniform verification software at the operator and the end user.
The concept presented here is based on the end-to-end security concept for a custody transfer ("GL") solution for the transmission of measurement data to remote displays by Dr. Martin Kahmann [MEMOGL]. It further builds on ABL's internal concept for legal metrology compliance.

This document is structured as follows: First, a classification and presentation of the transmission of the
data format is given, in the further sections the data format is explained.


### References

 - OCPP 1.5: Open Charge Alliance: Open Charge Point Protocol - Interface description between Charge Point and Central System, Version 1.5, as of June 8, 2012. 
 - MEMO-GL: Measurement systems for charging equipment: End-to-end security concept for a custody transfer ("GL") solution for the transmission of measurement data to remote displays, Dr. Martin Kahmann, Braunschweig, Germany. 
 - OBIS: IEC 62056-6-1 and -6-2.
 - JSON: Introduction to JSON, https://www.json.org/json-de.html (retrieved: 2018-04-04).


### Verwendete Abkürzungen

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
* UID: Unique IDentification
* UTC: Universal Time Coordinated

### 1.5 Versioning of the document

The version number of this document is structured as follows:

**Major Version:**
Changes at this point mean that the format has changed fundamentally. Changes have been made to the format that are no longer compatible with previous versions.

**Minor:**
The format has been supplemented new fields or the values of a field have been added (
Changes to the format. Backward compatibility is given).

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

While the reading of the previous values is possibly done with a time that is not in conformity with the calibration law (OCPP-based), the signed
the signed meter values are based on one. Even the two resulting times
can be correctly represented separately in two MeterValue objects.

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

The connection between several individual data records is ensured by continuous pagination.
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
2. section containing the actual **user data**.
3. signature over the payload data
4. optional: public key

The contents of the record serve as the input format into the [Link](https://transparenz.software/ transparency software). The [Link](https://transparenz.software/ transparency software) is available online for anyone to use to validate the dataset.

In the following chapters, the data format for the representation and transmission of the signed meter values is defined.

## JSON-based OCMF format.

JSON notation was chosen for the individual sections of the format for better readability and easier traceability. To save data volume, the following measures were taken:

 - Identifiers are kept short (since some need explanation anyway, not much clarity is lost here
lost)
 - In some places encapsulation was deliberately omitted. Instead, the identifiers are canonicalized.

The sections within the data format are separated with the special character "|". Within the sections the format corresponds to the JSON notation. Thus, the use of the special character "|" is not allowed within the sections.

That the format corresponds to the JSON notation with spaces and line feeds at
white space, it makes it easier for the user to read the text.
also makes it easier to read.

ValidationBetween signing and validation, the payload section must not be manipulated (removing and adding white spaces), otherwise positive validation is not possible.

In the following, the different sections of the format are explained. Furthermore, an example is given.

### Sections

At the beginning of the format there is a header for unique identification of the format.
The user data section and the signature section follow, separated by pipe characters.
This results in the following structure for the sections:

    OCMF|<utility data section>|<signature section>.

The words user data section and signature section are to be replaced by the contents of the respective sections.
replace.



#### Header

The header is used to uniquely identify the transmission format. In the case of OCMF, the string "OCMF" must be specified in this section.

#### User data section

The payload section consists of a JSON object. This contains several value mappings or
sub-objects. The contents of the payload section must not be modified between signing and verification of the signature.

##### General information

The information in this section refers primarily to the signature-generating component, which is referred to here as the gateway for the sake of simplicity. Depending on the structure, it may be more appropriate to refer to the measuring capsule or the signing component.

| Key: | Typ    |Card.| Description:                                                                                                                                                                                                                                                                                |
|------|--------|-----|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| FV | String |0..1 | Format-Version: Version des Datenformats in der Darstellung <major>.<minor> Die Versionsangabe wird entsprechend der Version dieses Dokumentes codiert, d.h. 0.4 entspricht Major 0 und Minor 4. Die Revision (dritte Stelle) wird nicht übertragen, da dies am Format selbst nichts ändert. |
| GI | String |0..1 | Gateway-Identification: Bezeichner des Herstellers für das System, welches die vorliegenden Daten erzeugt hat (Hersteller, Modell, Variante, etc.).                                                                                                                                          |
| GS | String |0..1 | Gateway-Serial: Seriennummer des o.g. Systems Dieses Feld ist bedingt obligatorisch.                                                                                                                                                                                                         | 
| GV | String |0..1 | Gateway-Version: Versionsbezeichnung des Herstellers für die SW auf o.g. |
<small>Table 1</small>

##### Pagination

The pagination indicates the reference to a transaction and arranges the total data record in the stream of the
readings via the meters in a comprehensible way. Several contexts are available for pagination,
each context has its own counter reading. The respective pagination counter is counted monotonically in ascending order with
with an increment of 1. I.e. successive values of the pagination counter count continuously in the space of natural numbers.

The Transaction context must be supported in all cases. The Fiscal context is optional and may be supported by the signature component optionally
supported by the signature component if signed readings outside transactions are desired.


| Key: | Type: |Card.| Description: |
|------|--------|-----|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PG | String |1..1 | Pagination of the entire data set, i.e. the data that fall together under one signature. Format: `<Kennzeichen><Zahl>` <br> Der String setzt sich aus einem kennzeichnenden Buchstaben für den Kontext und einer Zahl ohne führende Nullen zusammen. Für jeden Kontext existiert ein eigener unabhängiger Paginierungszähler. Folgende Kennzeichen sind definiert: <br> F: Fiscal – Ablesungen unabhängig von Transaktionen (optional) <br> T: Transaction – Ablesungen im Transaktionsbezug (obligatorisch) <br> Der jeweilige Paginierungszähler wird nach jeder Verwendung für einen Datensatz erhöht. |
<small>Table 2</small>

##### Meter identification

The meter identification is optional if the signature generating component has
already been identified in the general data.

<small>Table 3</small>
| Key: | Typ:   |Card.| Description:                                                              |
|------|--------|-----|----------------------------------------------------------------------------|
| MV   | String |0..1 | Meter-Vendor: Hersteller-Identifikation des Zählers, Name des Herstellers. |
| MM   | String |0..1 | Meter-Model: Modell-Identifikation des Zählers.                            |
| MS   | String |1..1 | Meter-Serial: Seriennummer des Zählers Dieses.                             |
| MF   | String |0..1 | Meter-Firmware: Firmware-Version des Zählers.                              |



##### Referencing the user

Fields belonging to this are included exactly when there is a transaction reference. Even if no user can be transaction reference no user can be assigned yet, the section is included. If there is no transaction reference exists, this group of fields is missing.

<small>Table 4</small>
| Key: | Typ:             |Card.| Description:                                                                                                                                                                                                                                                                                              |
|------|------------------|-----|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| IS   | Boolean          |1..1 | Identification-Status: Genereller Status zur Benutzerzuordnung: true: Benutzer erfolgreich zugeordnet, false: Benutzer nicht zugeordnet.                                                                                                                                                                   |
| IL   | String           |0..1 | Identification-Level: Aufgeschlüsselter Gesamtstatus der Benutzerzuordnung, dargestellt durch einen Bezeichner aus Tabelle 10.                                                                                                                                                                             |
| IF   | Array of  String |0..4 | Identification-Flags: Detailaussagen zur Nutzerzuordnung, dargestellt durch einen oder mehrere Bezeichner aus Tabelle 12 bis Tabelle 15. Die Bezeichner werden stets als String- Elemente in einem Array notiert. Auch ein oder kein Element muss als Array notiert werden.                                |
| IT   | String           |1..1 | Identification-Type: Typ der Identifikationsdaten, Bezeichner siehe Tabelle 16                                                                                                                                                                                                                             |
| ID   | String           |0..1 | Identification-Data: Die eigentlichen Identifikationsdaten entsprechend des Typs aus Tabelle 16, also z.B. eine Hex-Codierte UID entsprechend ISO14443.                                                                                                                                                    |


##### Referencing the charge point

This optional user data section provides a way to identify the charging point. This can be an alternative to identification by serial numbers or can be used additionally.

| Key: | Typ:   |Card.| Description:                                                                                                    |
|------|--------|-----|------------------------------------------------------------------------------------------------------------------|
| CT   | String |0..1 | Charge-Point-Identification-Type: Typ der Angabe zur Identifikation des Ladepunktes, Bezeichner siehe Tabelle 17 |
| CI   | String |0..1 | Charge-Point-Identification: Identifikations-Angabe für den Ladepunkt.                                           |
<small>Table 5</small>

##### Ablesungen

One or more readings can be stored in a record. Each reading is encapsulated in a sub-object. These objects are noted as an array under the key "RD" for Reading(s). This array thus contains one or more anonymous objects.
Each of these objects is constructed as described in Table 6.
For the readings, fields that have an identical value to the previous reading are omitted. However, this only applies within a signed record. This means concretely that, for example, only in the first sub-object the field "RI" is defined with a value (here the OBIS code) and is thus valid for all further readings. Likewise, for example, the field "TX" can be defined in the first sub-object as `B`, in the second as `C`, in the last with `E`, which means that the readings three to the penultimate take the value `C`.
The fields `RV`, `RI` , `RU` and `RT` can be omitted if only the occurrence of an error condition (event) of the meter is to be indicated.
The fields `RI` and `RU`, form a group. Fields of a group are either all present together or omitted together. 
A power failure during time-based billing represents an error event.

| Key: | Typ:   |Card| Description:                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
|------|--------|----|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| TM   | String |1..1| Time: Angabe zur Systemzeit der Ablesung und Synchronisationszustand. <br> Die Zeit wird nach ISO 8601 mit einer Auflösung von Millisekunden beschrieben. Das Format wird dementsprechend nach folgendem Schema gebildet: <br> `<Jahr>-<Monat>-<Tag>T<Stunden>:<Minuten>:<Sekunden>,<Millisekunden><Zeitzone>` <br> Dabei wird das Jahr vierstellig dargestellt, Monat, Tag, Stunden, Minuten und Sekunden zweistellig, Millisekunden dreistellig. Die Angabe zur Zeitzone besteht aus einem Vorzeichen und einer vierstelligen Angabe für Stunden und Minuten. Beispiel: <br> `2018-07-24T13:22:04,000+0200` <br> Der Synchronisationszustand besteht aus einem Großbuchstaben als Bezeichner. Dieser wird, getrennt durch ein Leerzeichen, der Zeit hinangestellt. Verfügbare Zustände siehe Tabelle 18.                                                             |
| TX   | String |0..1| Transaction: Ablesungsgrund, Bezug der Ablesung zur Transaktion, notiert als Großbuchstabe: <br> B – Beginn der Transaktion <br> C – Charging = während der Ladung (kann optional verwendet werden) <br> _ X – Exception = Fehler während der Ladung, Transaktion wird fortgesetzt, Zeit und/oder Energie sind ab dieser Ablesung (einschl.) nicht mehr verwertbar. <br> E – Ende der Transaktion, alternativ genauere Codes: <br> _ L – Ladevorgang wurde lokal beendet <br> _ R – Ladevorgang wurde aus der Ferne beendet <br> _ A – (Abort) Ladevorgang wurde durch Fehler abgebrochen <br> _ P – (Power) Ladevorgang wurde durch Stromausfall beendet <br> S – Suspended = Transaktion aktiv, aber gerade keine Ladung (kann optional verwendet werden) <br> T – Tarifwechsel <br> Dieses Feld fehlt, wenn kein Transaktionsbezug vorhanden ist. (Fiscal Metering) |
| RV   | Number |1..1| Reading-Value: Der Wert der Ablesung Hier wird auf das JSON-Datenformat Number zurückgegriffen, dies erlaubt unter anderem eine genaue Auszeichnung der gültigen Nachkommastellen. Die Darstellung darf durch weitere Behandlungsmethoden (z.B. Verarbeitung durch JSON-Parser) jedoch nicht umgeformt werden (Umschreibung der Zahl mit anderem Exponenten, Kürzung von Nachkommastellen, etc.) da damit die Darstellung der physikalischen Größe und damit potentiell die Anzahl der gültigen Stellen verändert würde. Entsprechend der Anwendungsregel wird empfohlen, den Messwert mit zwei Nachkommastellen Genauigkeit darzustellen, falls es sich um kWh handelt.                                                                                                                                                                                               |
| RI   | String |0..1| Reading-Identification: Bezeichner, welche Größe abgelesen wurde, gemäß OBIS-Code.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| RU   | String |1..1| Reading-Unit: Einheit der Ablesung, z.B. kWh.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| RT   | String |0..1| Reading-Current-Type: Die vom Zähler gemessene Stromart, z.B. Wechselstrom oder Gleichstrom, gemäß Tabelle 21: Vordefinierte Stromarten. <br> Dieses Feld ist optional. Es ist kein Default-Wert definiert.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| EF   | String |0..1| Error-Flags: Aussage, welche Größen durch einen Fehler nicht mehr verwertbar zur  Abrechnung sind. Jedes Zeichen in diesem String kennzeichnet eine Größe. Folgende Zeichen sind definiert: <br>  E – Energie <br>  t – Zeit                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| ST   | String |1..1| Status: Zustand des Zählers zum Zeitpunkt der Ablesung. Notiert als Kürzel gemäß Tabelle 9.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
<small>Table 6</small>


##### Extension points user data section

In the user data section it may be necessary for different manufacturers to be able to add their own data.
their own data. For this purpose, the following extension points are reserved in the namespace of the JSON object for free use
depending on the manufacturer:

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
| SA   | String |1..1| Signature-Algorithm: Wählt den zur Signaturbildung verwendeten Algorithmus aus. Dies umfasst den Signatur-Algorithmus, dessen Parameter und den Hash-Algorithmus der auf den zu signierenden Daten angewendet wird. Diese Angabe ist optional. Wird sie ausgelassen, ist der Default-Wert gültig. Tabelle 22 zeigt, welche Werte möglich sind.                                       |
| SE   | String |0..1| Signature-Encoding: Bezeichnet, wie die Signaturdaten kodiert sind, um im JSON-String abgelegt werden zu können. Diese Angabe ist optional. Wird sie ausgelassen, ist der Default-Wert gültig. Folgende Werte sind möglich: <p>hex – Die Signaturdaten sind im JSON-String hexadezimal codiert dargestellt (Default) <p>base64 – Die Signaturdaten sind entsprechend base64 abgelegt.|
| SM   | String |0..1| Signature-Mime-Type: Bezeichnet, wie die Signaturdaten zu interpretieren sind. Diese Angabe ist optional. Wird sie ausgelassen, ist der Default-Wert gültig. Folgende Werte sind möglich:<p>application/x-der – Ablage als ASN.1-codierte .der-Datei (Default)                                                                                                                       |
| SD   | String |1..1| Signature-Data: Die eigentlichen Signaturdaten entsprechend obiger Formatangaben.                                                                                                                                                                                                                                                                                                    |
<small>Table 7: Signaturdatenfelder der Signatursektion</small>

##### Extension points

Analogous to the extension points in section 0, unprotected extension points may also be desirable.
are desirable. The following are reserved for this purpose:

 * Reserved for vendor use are all top-level names in the signature data section JSON object that begin with the following letters:
	- U
	- V
	- W
	- X
	- Y
	- Z
 * The following initial letters are reserved accordingly for use by components in downstream processing of data:
     - A
	 - B
	 - C
	 - D
	 - E
	 - F

#### Reference of serial numbers, loading point and public key

To establish a reference between the public key to be used for signature verification and the
one or more serial numbers; alternatively, the loading point can also be identified directly by ID.
alternatively, the charging point can be identified directly by ID.

The public keys to the charging point must be transmitted to the verification component by means other than this protocol (out-of-band) in conjunction with the ID or serial numbers used, e.g. via a central register.

To uniquely identify a signature component, the following characteristics are defined in this protocol:

 - If the signature component is attributable to the energy meter, its serial number is sufficient.
 - If the signature component itself is the gateway and only one charging point is served, the serial number of the gateway or the energy meter is sufficient.
 - If a gateway is responsible for more than one charging point, the combination of the serial numbers of the gateway and energy meter is used for unique identification.
 - Alternatively, a direct identification of the charging point can be made.

Since these conditions depend on the design of the charging station, the corresponding serial number fields are
are marked below as conditionally mandatory.

### Example

Examples of the resulting JSON-based formats:

#### Beispieldatei Darstellungsformat

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
        "RD": [
            {
                "TM": "2018-07-24T13:22:04,000+0200 S",
                "TX": "B",
                "RV": 2935.6,
                "RI": "1-b:1.8.0",
                "RU": "kWh",
                "RT": "AC",
                "EF": "",
                "ST": "G"
            }
        ]
    }|{
    "SD":  "887FABF407AC82782EEFFF2220C2F856AEB0BC22364BBCC6B55761911ED651D1A922BADA88818C9671AFEE7094D7F536"
    }

## Other data fields

### Meter identification

#### Manufacturer and model of the meter

The following table represents the predefined values for the meter manufacturer and model fields.

| Manufacturer:     | Modell:                 |
|-----------------|-------------------------|
| Phoenix Contact | EEM-350-D-MCB           |
| Inepro Metering | Pro 1 Mod Pro 380 Mod   |
| Carlo Gavazzi   | EM340-DIN.AV2.3.X.S1.PF |
|                 | EM111-DIN.AV8.1.X.S1.PF |
|                 | EM210-72D.MV5.3.X.OS.X  |
| NZR             | EcoCount S 85           |
<small>Table 8: Werte für Hersteller/Modell des Zählers</small>

#### State / error of the counter

This field can take a value at the same time.

| Token: | Identifier:  | Description:                                                                          |
|---------|--------------|----------------------------------------------------------------------------------------|
| N       | NOT_PRESENT  | Der Zähler ist nicht vorhanden/gefunden worden                                         |
| G       | OK           | Zähler in Ordnung (Good)                                                               |
| T       | TIMEOUT      | Zeitüberschreitung beim Versuch, den Zähler anzusteuern                                |
| D       | DISCONNECTED | Zähler wurde von der Signaturkomponente getrennt                                       |
| R       | NOT_FOUND    | Zähler nicht mehr gefunden (ist bereits vorher mind. einmal gefunden worden) (Removed) |
| M       | MANIPULATED  | Manipulation erkannt                                                                   |
| X       | EXCHANGED    | Zähler getauscht (Seriennummer stimmt nicht mehr mit der zuletzt bekannten überein.)   |
| I       | INCOMPATIBLE | Zähler-Version oder dessen API nicht mit Signaturkomponente kompatibel                 |
| O       | OUT_OF_RANGE | Gelesener Wert außerhalb des Wertebereichs                                             |
| S       | SUBSTITUTE   | Es wurde ein Ersatzwert gebildet                                                       |
| E       | OTHER_ERROR  | Anderer, nicht näher bekannter Fehler                                                  |
| F       | READ_ERROR   | Zählerregister nicht korrekt ausgelesen; Wert der Auslesung ist nicht gültig           |
<small>Table 9: Zustand/Fehler des Zählers</small>

### User assignment

## State / Error

This field can take only one value at a time. For clarity, the values are divided into several
groups, which are not distinguished in the format.

| Gruppe:               | Status/Fehler: | Description:                                                                                                                              |
|-----------------------|----------------|--------------------------------------------------------------------------------------------------------------------------------------------|
| nicht verfügbar       | -              | Das Feld ist nicht angegeben.                                                                                                              |
| Status ohne Zuordnung | NONE           | Es liegt keine Nutzerzuordnung vor. Die weiteren Daten zur Nutzerzuordnung haben keine Aussagekraft.                                       |
| Status mit Zuordnung  | HEARSAY        | Die Zuordnung ist ungesichert; also z.B. durch Auslesen einer RFID-UID.                                                                    |
|                       | TRUSTED        | Die Zuordnung kann in gewissem Maße vertraut werden, es gibt jedoch keine absolute Verlässlichkeit. Beispiel: Authorisierung durch Backend |
|                       | VERIFIED       | Die Zuordnung wurde durch die Signaturkomponente und spezielle Maßnahmen verifiziert.                                                      |
|                       | CERTIFIED      | Die Zuordnung wurde durch die Signaturkomponente verifiziert mit Hilfe einer kryptographischen Signatur die die Zuordnung zertifiziert.    |
|                       | SECURE         | Die Zuordnung wurde durch eine sicheres Merkmal hergestellt (z.B. sichere RFID-Karte, ISO15118 mit Plug- and-Charge, etc.)                 |
| Fehler                | MISMATCH       | Fehler; UIDs passen nicht zueinander.                                                                                                      |
|                       | INVALID        | Fehler; Zertifikat nicht korrekt (Prüfung negativ).                                                                                        |
|                       | OUTDATED       | Fehler; referenziertes Trust-Zertifikat abgelaufen.                                                                                        |
|                       | UNKNOWN        | Zertifikat konnte nicht erfolgreich geprüft werden (kein passendes Trust-Zertifikat gefunden).                                             |
<small>Table 10: Status- und Fehlerzustände der Nutzerzuordnung</small>

#### Details

This field is composed of four sub-fields and gives details about the status of the user assignment.
Table 11 describes the division into subfields, the following tables describe the valid values.
valid values.

##### Breakdown Details User Assignment

| Inhalt:                                                    |Details siehe |
|------------------------------------------------------------|--------------|
| [Zuordnungsanteil RFID](#rfid)                             | Tabelle 12   |
| [Zuordnungsanteil OCPP](#ocpp)                             | Tabelle 13   |
| [Zuordnungsanteil ISO 15118](#iso15118)                    | Tabelle 14   |
| [Zuordnungsanteil PLMN](#plmn)                             | Tabelle 15   |
<small>Table 11: Aufteilung - Details der Nutzerzuordnung</small>

##### [Zuordnungsanteil RFID](#rfid)

| Identifier:  | Description:                                                                                                       |
|--------------|---------------------------------------------------------------------------------------------------------------------|
| RFID_NONE    | keine Zuordnung per RFID                                                                                            |
| RFID_PLAIN   | Zuordnung über externen RFID-Kartenleser                                                                            |
| RFID_RELATED | Zuordnung über geschützten RFID-Kartenleser                                                                         |
| RFID_PSK     | Ein vorher bekannter gemeinsamer Schlüssel (Pre-shared key) wurde genutzt, z.B. mit einer abgesicherten RFID-Karte. |
<small>Table 12: Zuordnungsanteil RFID </small>

##### [Zuordnungsanteil OCPP](#ocpp)

| Identifier:    | Description:                                                                                      |
|----------------|----------------------------------------------------------------------------------------------------|
| OCPP_NONE      | keine Nutzerzuordnung durch OCPP                                                                   |
| OCPP_RS        | Zuordnung durch OCPP-RemoteStart-Methode                                                           |
| OCPP_AUTH      | Zuordnung durch OCPP-Authorize-Methode                                                             |
| OCPP_RS_TLS    | Transport-Layer-Security wurde zur Übertragung der Zuordnung per OCPP-RemoteStart-Methode benutzt. |
| OCPP_AUTH_TLS  | Transport-Layer-Security wurde zur Übertragung der Zuordnung per OCPP-Authorize-Methode benutzt.   |
| OCPP_CACHE     | Zuordnung durch Authorization-Cache von OCPP                                                       |
| OCPP_WHITELIST | Zuordnung durch White-List von OCPP                                                                |
| OCPP_CERTIFIED | Ein Zertifikat des Backends welches die Nutzerzuordnung zertifiziert wurde eingesetzt.             |
<small>Table 13: Zuordnungsanteil OCPP</small>

#### [Zuordnungsanteil ISO 15118](#iso15118)

| Identifier:   | Description:                        |
|---------------|--------------------------------------|
| ISO15118_NONE | keine Nutzerzuordnung durch ISO15118 |
| ISO15118_PNC  | Plug & Charge wurde benutzt          |
<small>Table 14: Zuordnungsanteil ISO 15118</small>

#### [Zuordnungsanteil PLMN](#plmn)

| Identifier: | Description:   |
|-------------|-----------------|
| PLMN_NONE   | keine Zuordnung |
| PLMN_RING   | Anruf           |
| PLMN_SMS    | Kurznachricht   |
<small>Table 15: Zuordnungsanteil PLMN</small>

#### Type

Table 16 describes the possible types of user mappings, which can be used in the Type field as
unsigned integer in the Type field. These mappings are based on the specifications from
OCPP. However, OCPP currently (version 1.5) only provides 20 characters for the identification feature.
In accordance with the maximum length of an IBAN (34 characters), however, the following assignments were made for the data area here
assignments of up to 40 bytes have been provided for the data area.

| Token: | Identifier:  | Description:                                                                                                                      |
|---------|--------------|------------------------------------------------------------------------------------------------------------------------------------|
| 0       | NONE         | Keine Zuordnung verfügbar                                                                                                          |
| 1       | DENIED       | Zuordnung momentan nicht abrufbar (wg. Zwei-Faktor-Autorisierung)                                                                  |
| 2       | UNDEFINED    | Typ nicht näher benannt                                                                                                            |
| 10      | ISO14443     | UID einer RFID-Karte entsprechend ISO 14443. Dargestellt als 4 oder 7 Bytes in hexadezimaler Schreibweise.                         |
| 11      | ISO15693     | UID einer RFID-Karte entsprechend ISO 15693. Dargestellt als 8 Bytes in hexadezimaler Schreibweise.                                |
| 20      | EMAID        | Electro-Mobility-Account-ID entsprechend ISO/IEC 15118 (String mit der Länge 14 oder 15)                                           |
| 21      | EVCCID       | ID eines Elektrofahrzeugs entsprechend ISO/IEC 15118 (Maximallänge 6 Zeichen)                                                      |
| 30      | EVCOID       | EV-Contract-ID entsprechend DIN 91286.                                                                                             |
| 40      | ISO7812      | Identification-Card-Format entsprechend ISO/IEC 7812 (Kredit- und Bankkarten, etc.)                                                |
| 50      | CARD_TXN_NR  | Kartentransaktionsnummer (CardTxNbr) für eine Zahlung mit Kredit- oder Bankkarte, die in einem Terminal am Ladepunkt benutzt wird. |
| 60      | CENTRAL      | Zentral generierte ID. Kein genaues Format definiert, kann z.B. eine UUID sein. (OCPP 2.0)                                         |
| 61      | CENTRAL_1    | Zentral generierte ID, z.B. durch Start per SMS. Kein genaues Format definiert. (bis OCPP 1.6)                                     |
| 62      | CENTRAL_2    | Zentral generierte ID, z.B. durch Start durch Operator. Kein genaues Format definiert. (bis OCPP 1.6)                              |
| 70      | LOCAL        | Lokal generierte ID. Kein genaues Format definiert, kann z.B. eine UUID sein. (OCPP 2.0)                                           |
| 71      | LOCAL_1      | Lokal generierte ID, z.B. intern durch den Ladepunkt erzeugte ID. Kein genaues Format definiert. (bis OCPP 1.6)                    |
| 72      | LOCAL_2      | Lokal generierte ID, für sonstige Fälle. Kein genaues Format definiert. (bis OCPP 1.6)                                             |
| 80      | PHONE_NUMBER | Internationale Telefonnummer mit führendem „+‟.                                                                                    |
| 90      | KEY_CODE     | User-bezogener, privater Key-Code. Kein genaues Format definiert.                                                                  |
<small>Table 16: Typen der Nutzerzuordnung</small>

### Zuordnung des Ladepunktes

| Identifier: | Description:                                                                                              |
|-------------|------------------------------------------------------------------------------------------------------------|
| EVSEID      | EVSE-ID                                                                                                    |
| CBIDC       | Charge-Box-ID und Connector-ID nach OCPP, als Feldtrenner wird ein Leerzeichen benutzt, z.B. „STEVE_01 1“. |
<small>Table 17: Typen der Ladepunktzuordnung</small>

### Status der Zeit

| Identifier: | Description:                                                                                                                                                                                                                                                                                 |
|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| U           | unbekannt, unsynchronisiert                                                                                                                                                                                                                                                                   |
| I           | informativ (Info-Uhr)                                                                                                                                                                                                                                                                         |
| S           | synchronisiert                                                                                                                                                                                                                                                                                |
| R           | relative Zeitabrechnung mit einem eichrechtlich akkuraten Zeitgeber, basierend auf einer Info-Uhr. D.h. zu Beginn des Ladevorgangs war eine Zeit mit informativer Qualität vorhanden. Während des Ladevorgangs wurde die Zeit (Dauer) entsprechend der eichrechtlichen Anforderungen erfasst. |
<small>Table 18: Status der Zeit</small>


### Nature of the measured value

#### OBIS codes

To define a format in version 1.0, several possible OBIS codes have been listed here. In the
subsequent work, it can still be clarified which of them are to be used. The formats from the previous version
0.3 have been retained.

| OBIS-Code:    | Description:                                                                                              |
|---------------|------------------------------------------------------------------------------------------------------------|
| 1-b:1.8.e     | Bezug von elektrischer Energie (Wirkarbeit) aus dem Stromnetz (des Ladepunktbetreibers) an den Kunden.     |
| 1-b:2.8.e     | Lieferung von elektrischer Energie (Wirkarbeit) von dem Kunden an das Stromnetz (des Ladepunktbetreibers). |
| 1-b:1.8.0     | Bezug von elektrischer Energie (Wirkarbeit) aus dem Stromnetz (des Ladepunktbetreibers) an den Kunden.     |
| 1-b:2.8.0     | Lieferung von elektrischer Energie (Wirkarbeit) von dem Kunden an das Stromnetz (des Ladepunktbetreibers). |
| 1-0:1.8.0     | Bezug von elektrischer Energie (Wirkarbeit) aus dem Stromnetz (des Ladepunktbetreibers) an den Kunden.     |
| 1-0:2.8.0     | Lieferung von elektrischer Energie (Wirkarbeit) von dem Kunden an das Stromnetz (des Ladepunktbetreibers). |

<small>Table 19: Vordefinierte OBIS-Codes</small>

#### Units

| Unit: |
|----------|
| kWh      |
| Wh       |
<small>Table 20: Predefined units</small>

#### Stromart

| Identifier: | Description:       |
|-------------|---------------------|
| AC          | Wechselstrommessung |
| DC          | Gleichstrommessung  |
<small>Table 21: Vordefinierte Stromarten</small>

### Cryptography

#### Signature methods

| Identifier: | Signature algorithm: | Curve name and, if applicable, synonyms:                                                                               | Key length: Hash algorithm: | Block length: |
|-------------------------------------------------------|-------------------------|--------------------------------------------------------------|--------------------|---------------------|---------------|
| ECDSA-secp192k1-SHA256                                | ECDSA                   | secp192k1                                                    | 192 bit            | SHA-256             | 48            |
| ECDSA-secp256k1-SHA256                                | ECDSA                   | secp256k1                                                    | 256 bit            | SHA-256             | 64            |
| ECDSA-secp192r1-SHA256                                | ECDSA                   | secp192r1                                                    | 192 bit            | SHA-256             | 48            |
| ECDSA-secp256r1-SHA256  (Default in OCMF Version 0.4) | ECDSA                   | secp256r1,  NIST P-256, ANSI X9.62 elliptic curve prime256v1 | 256 bit            | SHA-256             | 64            |
| ECDSA-brainpool256r1-SHA256                           | ECDSA                   | brainpool256r1                                               | 256 bit            | SHA-256             | 64            |
| ECDSA-secp384r1-SHA256                                | ECDSA                   | secp384r1,  NIST P-384, ANSI X9.62 elliptic curve prime384v1 | 384 bit            | SHA-256             | 96            |
| ECDSA-brainpool384r1-  SHA256                         | ECDSA                   | brainpool384r1                                               | 384 bit            | SHA-256             | 96            |
<small>Table 22: Vordefinierte Signatur-Algorithmen und ihre Parameter</small>

#### Public-Key-Typen

| Identifier: | Signature algorithm: | Curve name and, if applicable, synonyms:              | Key length: Hash algorithm: | Block length: |
|-------|------------------------------------------------|-------------------------|----------------|----------------------|---------------|
| 1     | ECDSA-secp192k1                                | ECDSA                   | secp192k1      | 192 bit              | 48            |
| 2     | ECDSA-secp256k1                                | ECDSA                   | secp256k1      | 256 bit              | 64            |
| 3     | ECDSA-secp192r1                                | ECDSA                   | secp192r1      | 192 bit              | 48            |
| 4     | ECDSA-secp256r1  (Default in OCMF Version 0.4) | ECDSA                   | secp256r1      | 256 bit              | 64            |
| 5     | ECDSA-brainpool256r1                           | ECDSA                   | brainpool256r1 | 256 bit              | 64            |
| 6     | ECDSA-secp384r1                                | ECDSA                   | secp384r1      | 384 bit              | 96            |
| 7     | ECDSA-brainpool384r1                           | ECDSA                   | brainpool384r1 | 384 bit              | 96            |
<small>Table 23: Vordefinierte Sclüsseltypen</small>