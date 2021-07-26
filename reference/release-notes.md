---
description: >-
  All notable changes to this project will be documented in this file. This
  project adheres to Semantic Versioning.
---

# Release Notes

## Version 1.4.0

### Added

* \[e2e\] Add back test\_ipv4 which passes
* \[e2e\] Add support for fork repositories in github workflows
* \[helm\] Add curiefense to Istio-helm charts
* \[docker\] Add missing packages to curielogger \(to run contrib scripts\)
* \[ui\] Add options to configure links to Kibana & Grafana
* \[curielogger\] Add docker-compose e2e tests
* \[e2e\] Add tests to last missing components, fix referral bug in url maps editor, chang coverage thresholds, remove unused code
* \[ui\] Add autocomplete support to WAF Policies editor and resolve a bug in URL Maps editor
* \[ui\] Add a requirement of at least one tag for Tag Rule tags list in Tag Rules json schema
* \[e2e\] Add test of flow control editor in case of multiple limit option keys
* \[helm\] Add v2 deployment tests
* \[e2e\] Add test on fluentd
* \[helm\] Add filebeat to the helm deployment
* \[curielogger\] Add logrotate container
* \[e2e\] Add a testcase for pairwith limits
* \[e2e\] Add Rust formatting tests to Makefile
* Add configs and templates for Elasticsearch 6.x
* Add an nginx-ingress container
* Add map to define request\_map
* Add knob to disable Kibana initialization \(es6 init script\)

### Updated

* \[ui\] Update dependencies with found security vulnerabilities.
* \[ui\] Update version to 1.3.0 to match the achieved milestone and overall\* system version
* \[docker\] Update Envoy configuration version to v3
* \[e2e\] Update log patterns
* \[docker\] Update Istio image to use Envoy binary for 1.9.2
* \[helm\] Update curiefense EnvoyFilters to v3
* \[docker\] Update Envoy binary for Istio
* \[ci\] Update minikube to fix CI
* \[e2e\] Update Rust unit tests to include urldecode
* \[curieproxy\] Update iptools.so in curieproxy with new url decode function
* Update iptools.so for lua
* Update iptools.so with fixed urldecode
* Update with new urldecode algorithm

### Improved

* \[e2e\] Improve general coverage of UI unit tests in DocumentEditor.vue and Publish.Vue for a total coverage of 89%+
* \[e2e\] Improve general coverage of UI unit tests, add types to unit tests, fix small issues throughout the UI

### Removed

* \[helm\] Remove helm install
* \[e2e\] Remove test for feature that does not exist anymore
* \[helm\] Remove references & variables for postgres & curielogserver
* \[deploy\] Remove remaining postgres configuration values
* Remove the [ROADMAP.md](http://roadmap.md/) file in favor of [RELEASES.md](http://releases.md/)
* Remove ILM for ES 6.x as it was added in 7.x
* Remove logstashs' from e2e-ci.yml

### Fixed



* \[ci\] use more recent shellcheck version, fix remaining errors
* \[e2e\] Fix ratelimit countby tests
* \[e2e\] Fix WAF Rules tests
* \[e2e\] Fix arguments passed to [deploy.sh](http://deploy.sh/). Fixes e2e tests.
* \[e2e\] Fix elasticsearch port for tests on minikube
* \[ci\] Fix deployment & tests following Istio update
* \[e2e\] Fix latency tests \([deploy-gke.sh](http://deploy-gke.sh/)\)
* \[ci\] Fix environment for rust & lua tests
* \[docker-compose\] Fix curieproxy metrics scrape
* \[ui\] Fix referral bug in url maps editor
* \[curielogger\] Fix test\_logs Elasticsearch query
* \[docker-compose\] Fix CI
* \[curielogger\] Fix tag rules logging
* \[curieproxy\] fix geo-related ratelimit counters
* \[curieproxy\] fix geo-related ratelimit scope checks
* Fix challenge in flow control
* Fix start\_curiefense script
* Fix flow checks tags
* Fix default return codes
* Fix nginx failure with unknown remote ip
* Fix curiefense/images/uiserver/Dockerfile to reduce vulnerabilities

### Enhanced

* N/A

