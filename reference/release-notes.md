---
description: 'aka Version History, Changelog'
---

# Release Notes

### Version 1.X.X

#### Enhanced

* \[curieconf\] Enhance API to allow requesting specific properties of documents

#### Added

* \[tests\] Add CodeQL Security Scanning
* \[curieproxy\] Add continent data and more country data
* \[curieproxy\] Add geolocation for requests when data is present
* \[e2e\] Add script to run nightly tests
* \[iptools\] Add city lookup capabilities
* \[curielogger\] Add config file structure
* \[tests\] Add workflows for Rust
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
* \[tests\] Added full coverage for RateLimitsEditor.vue
* \[tests\] Added full coverage for WAFSigsEditor.vue
* \[ui\] Added scrollbar to Burma’s dropdown menu item with max height of 12rem
* \[confserver\] Added an example flow control document to bootstrap data
* \[confserver\] Added basic tags to bootstrap data
* \[confserver\] Add geolite2 city
* \[e2e\] Add checklogs-compose.sh
* \[e2e\] add script to run nightly tests
* \[curielogger\] elasticsearch index: add timestamp field
* \[images\] add fluentd image
* \[helm\] add fluentd support
* \[helm\] add elasticsearch support
* \[docker-compose\] add ELK containers
* \[helm\] deployments: add support for google storage buckets
* \[helm\] deployment: add variables to disable parts of the chart
* \[e2e, tests\] add config for rate limit action tests
* \[e2e, tests\] add ratelimit countby tests
* \[e2e, tests\] add rate limit event test with attributes
* \[e2e\]: add rate limit query&tag&authority scope tests
* \[e2e\] add rate limit asn & method scope tests
* \[e2e\] add rate limit company scope tests
* \[e2e\] add rate limit country scope tests
* \[e2e\] add test for ratelimit scope by ip
* \[e2e\] add tagrule test with AND
* \[e2e\] add TagRules test for ASN, country, ipv4, ipv6. Use new rule format.
* \[e2e\] add minikube latency testing script
* \[e2e\] add logs smoke tests
* \[curieproxy\] add missing lua dependency
* \[curielogger\] Added alpha fluentd support
* \[curielogger\] Added logstash support
* \[curielogger\] Added elasticsearch dumb client
* \[curieproxy\] Added envoy-1.16.2 binary with symbols for lua
* \[iptools\] added test\_regex\(\)
* \[iptools\] Added geoipdb bindings
* \[iptools\] Added sis\_set.clear\(\) method
* \[iptools\] Added test\_sigset.lua
* \[iptools\] Added sigset object to create lua objects able to match multiple regex signatures at once
* \[iptools\] Added iptonum\(\) in iptools lib
* \[curieconf\] Added in-place entry edition in curieconf \(server, client, CLI\)
* \[iptools\] Added modhash\(\) to iptools
* \[iptools\] Added AVLTreeMap.dump\(\)
* \[iptools\] Added InsertSide enum
* \[iptools\] Add more tests for AVLTreeMap
* \[iptools\] add Node.dump
* \[iptools\] Added AVL rebalancing
* \[iptools\] add avltree.rs dependence in makefile
* \[iptools, tests\] Added tests to avltree
* \[iptools\] Added height computation
* \[curietasker\] Added 'update\_and\_publish' task
* \[curietasker\] Added 'publish' task
* \[iptools\] Add ability to use "\*" in place of config and branch lists in task db
* \[iptools\] Add confserver's tools/publish endpoint to client and CLI
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

#### Updated

* \[curielogger\] Update deployment configs/files
* \[tests\] Update .github/workflows/codeql-analysis.yml
* \[e2e\] update smoke test patterns & script
* \[helm\] Update logstash pipeline configuration
* \[e2e, tests\] update ratelimit for test action-ban-503
* \[e2e\] update curiefense-fluentd-patterns.txt
* \[curielogger\] Update cflog format and add pg back
* \[curielogger\] Update curielogger for new format
* \[docker-compose\] update env variables for curielogger
* \[helm\] update chart following changes in curielogger
* \[bootstrap-script\] updated version to 1.2.10
* \[bootstrap-script\] Updated axios version to avoid security vulnerability
* \[confserver\] update bootstrap waf signatures
* \[bootstrap-script\]
* \[confserver\]: update profiling lists for confdb bootstrap bundle

#### Improved

* \[ui\] Improved performance for EntriesRelationList prop validator
* \[iptools\] improved Node.dump\(\)
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

#### Removed

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
* \[curieproxy\]
* \[curieproxy\] :waf debug -- removed the inner loop
* \[ui\] Removed unnecessary card wrapper
* \[ui\] Removed conversion of old entries + relation data structure to the new structure, should be received from server in correct new structure
* \[curietasker\] return false removed
* \[curietasker\] removed some more debug prints
* \[curieconf\] remove references to now removed mongobackend package
* \[curieconf\] Removed outdated mongo backend
* \[curieconf\] Removed any loose API calls
* \[curieconf\] Removed duplicated API call in AccessLog.vue

#### Fixed

* \[e2e, helm\] fix log deletion in checklogs-helm.sh
* \[curielogger\] Fix indexpatter for non data stream environments
* \[iptools\] fix iptools mmdb test
* \[e2e\] fix action-related rate limit rule tests
* \[deploy, docker-compose\] fix use of fluentd
* \[deploy\] deploy-dev.sh: fix log check step
* \[currielogger\] elasticsearch mapping: fix types for arrays
* \[curielogger\] omit keys in upstream when there is not upstream \(fixes ES\)
* \[ui\] fix jest global window
* \[curielogger\] fix json names
* \[curielogger, helm\] elasticsearch: deployment fixes
* \[ui\] Fixed Bug - When creating multiple documents in a row throws an error
* \[curielogger\] Fixed missing fields in request attribute structure
* \[e2e\] fix test TestTagRules.test\_ipv4
* \[curieproxy\] waf exclude restrict bug fixed
* \[curieproxy\] acl bug fixed
* \[curieproxy\] more debug messages -- bad data type number error fixed
* \[ui\] Fixed download button in AccessLog component
* \[curieproxy\] exclude sig new format bug fixed
* \[curieproxy\]
* \[ui\] waf reason UI fixed
* \[images\] grafana image: fix permissions for provisioned dashboards & datasources
* \[ui\] Fix indentation of WAF policy editor and WAF signature viewer content
* \[ui\] Fix bug preventing docs from being downloaded
* \[ui\] Fix name of all download buttons \(removed unneeded 'x'\)
* \[ui\] Fix bug where first load of document editor page would not load docs correctly
* \[confserver\] Fixed wrong bucket name in bootstrap data \(duplicated prod instead of prod + devops\)
* \[ui\] Fix bug where first load of document editor page would not load docs correctly

