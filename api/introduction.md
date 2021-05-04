# API Introduction

As discussed in [Data Structures](introduction.md), Curiefense's data is stored within:

* Configurations
* Documents
* Entries
* Blobs

A Configuration is a set of Blobs and Documents. A Document is a set of Entries.

All of these data structures can be edited via API:

* A Document is a file treated as a JSON list of entries.
* An entry is a JSON dictionary with an `id` field. The `id` field value must be unique inside the document, and must be a valid part of an URL.
* A Blob is a file treated as binary data.

## Versioning

Each time a Configuration is modified and [published](../console/publish-configuration.md), a new version is created. A Configuration can be [reverted back to a previous version at any time](../git/version-control.md).

## Operations

The following types of operations are accessed as follows.

### **API endpoint namespaces**  \([more info](api-namespaces-and-endpoints.md)\)

* **configs** \(for manipulating Configurations\)
* **tools** \(for publishing, etc.\)
* **db** \(for accessing persistent data storage\)

### **cURL and Curiefense CLI tool** \([more info](untitled-1.md)\)

* **blob** \(for manipulating blobs\)
* **conf** \(for manipulating Configurations\)
* **db** \(for manipulating persistent memory database\)
* **doc** \(for manipulating Documents\)
* **entry** \(for manipulating Entries\)
* **key** \(for manipulating keys inside the database\)
* **sync** \(to maintain synchronization, e.g. the configuration API server and cloud storage\)

