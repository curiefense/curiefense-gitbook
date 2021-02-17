---
description: >-
  All notable changes to this project will be documented in this file. This
  project adheres to Semantic Versioning.
---

# Release Notes

## Version 1.3.0

### Enhanced

* \[curieconf\] Enhance API to allow requesting specific properties of documents

### Added

* \[curieproxy\] Add continent data and more country data
* \[curieproxy\] Add geolocation for requests when data is present
* \[curielogger\] Add config file structure
* \[curieproxy\] Add continent data and more country data
* \[ui\] Added 3 categories to tags \(legitimate, malicious, neutral\)
* \[curieconf, ui\] Added mask boolean field to WAF policy Cookie/Header/Argument
* \[ui\] Added is-single-input-column prop to ResponseAction component
* \[ui\] Added more tests to DocumentSearch component \(98%+ Coverage\)
* \[ui\] Added routing to document editor page per the following schema: /config/branch/doc\_type/doc\_id
* \[ui\] Added search page for documents
* \[ui\] Added units suffix \(seconds\) to TTL in RateLimitsEditor.vue
* \[ui\] Added units suffix \(seconds\) to TTL in FlowControlEditor.vue
* \[ui\] Added ace json editor to DB editor screen
* \[ui\] Added default textarea json editor when failing to load ace json editor
* \[ui\] Added indicator for missing data in DocumentEditor.vue
* \[ui\] Added indicator for loading data in DocumentEditor.vue
* \[ui\] Added indicator for missing data in DBEditor.vue
* \[ui\] Added indicator for loading data in DBEditor.vue
* \[ui\] Added scrollbar to Burma’s dropdown menu item with max height of 12rem
* \[confserver\] Added an example flow control document to bootstrap data
* \[confserver\] Added basic tags to bootstrap data
* \[confserver\] Add geolite2 city
* \[curielogger\] elasticsearch index: add timestamp field
* \[images\] add fluentd image
* \[helm\] add fluentd support
* \[helm\] add elasticsearch support
* \[docker-compose\] add ELK containers
* \[helm\] deployments: add support for google storage buckets
* \[helm\] deployment: add variables to disable parts of the chart
* \[curieproxy\] add missing lua dependency
* \[curielogger\] Added alpha fluentd support
* \[curielogger\] Added logstash support
* \[curielogger\] Added elasticsearch dumb client
* \[curieproxy\] Added envoy-1.16.2 binary with symbols for lua
* \[curieconf\] Added in-place entry edition in curieconf \(server, client, CLI\)
* \[curietasker\] Added 'update\_and\_publish' task
* \[curietasker\] Added 'publish' task
* \[docker-compose\] ELASTICSEARCH\_URL added to curielogger entry in compose yaml
* \[curieproxy\] tests branch configuration added
* \[curieconf\] tests branch configuration added
* \[ui\] added links to side menu
* \[curieproxy\] benchmark and tests added
* \[curieproxy\] added debug message for Rust Sig
* \[curieproxy\] rust calls with debug -- :add instead of .add
* \[curieproxy\] debug messages added
* \[curietasker\] log info added for remote debugging
* \[curietasker\] adding jsonschema to setup.y install\_requires
* \[curieproxy\] debug added to tagging matching points

### Updated

* \[curielogger\] Update deployment configs/files
* \[helm\] Update logstash pipeline configuration
* \[curielogger\] Update cflog format and add pg back
* \[curielogger\] Update curielogger for new format
* \[docker-compose\] update env variables for curielogger
* \[helm\] update chart following changes in curielogger
* \[bootstrap-script\] updated version to 1.2.10
* \[bootstrap-script\] Updated axios version to avoid security vulnerability
* \[confserver\] update bootstrap waf signatures
* \[confserver\]: update profiling lists for confdb bootstrap bundle

### Improved

* \[ui\] Improved performance for EntriesRelationList prop validator
* \[ui\] Replaced all download functions with a new downloadFile function in Utils
* \[curieconf\] Enhance api to allow requesting specific properties of documents
* \[ui\] Use the new enhanced api to drastically improve loading time of “Search Document” page
* \[ui\] Added scrollbar to Burma’s dropdown menu item with max height of 12rem
* \[ui\] Fixed download button in AccessLog component
* \[ui\] Changed colors of JSON Editor menu and mode selector to greyscale
* \[ui\] Hiding "Clear all sections" button in tagrules when source of list is not "self-managed"
* \[ui\] Fix indentation of WAF policy editor and WAF signature viewer content
* \[ui\] Fix bug preventing docs from being downloaded
* \[ui\] Fix name of all download buttons \(removed unneeded 'x'\)
* \[ui\] Now using new document defaults for missing props of documents

### Removed

* \[ui\] Flow Control - Removed the regex symbol next to Method, Host, and Path
* \[curielogger\] removed addition of hardcoded path to logstash url
* \[curieproxy\] removed reference to bt
* \[curieproxy\] schema removed, waf sig corrected
* \[curieproxy\] Removed old unused schema files
* \[confserver\] Removed unneeded params from bootstrap flow control action prop
* \[ui\] Removed unneeded module from dependencies
* \[ui\] Removed inline styling, !important, tag
* \[ui\] Added main.scss containing general classes
* \[ui\] Removed unneeded whitespace
* \[ui\] Improvements to the ui - general alignment of components throughout the system
* \[ui\] Removed deprecated html tag
* \[curieproxy\] removed logs
* \[curieproxy\] hscan.lus removed
* \[curieproxy\] globals.WAF removed
* \[ui\] Removed unnecessary card wrapper
* \[ui\] Removed conversion of old entries + relation data structure to the new structure, should be received from server in correct new structure
* \[curietasker\] return false removed
* \[curietasker\] removed some more debug prints
* \[curieconf\] remove references to now removed mongobackend package
* \[curieconf\] Removed outdated mongo backend
* \[curieconf\] Removed any loose API calls
* \[curieconf\] Removed duplicated API call in AccessLog.vue

### Fixed

* \[curielogger\] Fix indexpatter for non data stream environments
* \[deploy, docker-compose\] fix use of fluentd
* \[deploy\] deploy-dev.sh: fix log check step
* \[currielogger\] elasticsearch mapping: fix types for arrays
* \[curielogger\] omit keys in upstream when there is not upstream \(fixes ES\)
* \[ui\] fix jest global window
* \[curielogger\] fix json names
* \[curielogger, helm\] elasticsearch: deployment fixes
* \[ui\] Fixed Bug - When creating multiple documents in a row throws an error
* \[curielogger\] Fixed missing fields in request attribute structure
* \[curieproxy\] waf exclude restrict bug fixed
* \[curieproxy\] acl bug fixed
* \[curieproxy\] more debug messages -- bad data type number error fixed
* \[ui\] Fixed download button in AccessLog component
* \[curieproxy\] exclude sig new format bug fixed
* \[ui\] waf reason UI fixed
* \[images\] grafana image: fix permissions for provisioned dashboards & datasources
* \[ui\] Fix indentation of WAF policy editor and WAF signature viewer content
* \[ui\] Fix bug preventing docs from being downloaded
* \[ui\] Fix name of all download buttons \(removed unneeded 'x'\)
* \[ui\] Fix bug where first load of document editor page would not load docs correctly
* \[confserver\] Fixed wrong bucket name in bootstrap data \(duplicated prod instead of prod + devops\)
* \[ui\] Fix bug where first load of document editor page would not load docs correctly

