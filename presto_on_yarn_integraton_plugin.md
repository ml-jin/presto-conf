# presto - yarn project

[TOC]

## [Introduction](https://prestodb.io/presto-yarn/#introduction)

This project contains the code and needed to integrate Presto [Presto](https://prestodb.io/) with Apache Hadoop YARN using [Apache Slider](https://slider.incubator.apache.org/)

Presto on YARN can be set up either manually using Apache Slider or via Ambari Slider Views if you are planning to use HDP distribution.

## Content

- Manual Installation on a YARN-Based Cluster
  - [Deploying Presto on a YARN-Based Cluster](https://prestodb.io/presto-yarn/installation-yarn-manual.html#deploying-presto-on-a-yarn-based-cluster)
  - [Using Apache Slider to Manually Install Presto on a YARN-Based Cluster](https://prestodb.io/presto-yarn/installation-yarn-manual.html#using-apache-slider-to-manually-install-presto-on-a-yarn-based-cluster)
  - [Debugging and Logging](https://prestodb.io/presto-yarn/installation-yarn-manual.html#debugging-and-logging)
  - [Links](https://prestodb.io/presto-yarn/installation-yarn-manual.html#links)
- Automated Installation on a YARN-Based Cluster
  - [Deploying Presto on a YARN-Based Cluster](https://prestodb.io/presto-yarn/installation-yarn-automated.html#deploying-presto-on-a-yarn-based-cluster)
  - [Using Ambari Slider View to Install Presto on a YARN-Based Cluster](https://prestodb.io/presto-yarn/installation-yarn-automated.html#using-ambari-slider-view-to-install-presto-on-a-yarn-based-cluster)
  - [Additional Configuration Options](https://prestodb.io/presto-yarn/installation-yarn-automated.html#additional-configuration-options)
  - [Debugging and Logging](https://prestodb.io/presto-yarn/installation-yarn-automated.html#debugging-and-logging)
  - [Links](https://prestodb.io/presto-yarn/installation-yarn-automated.html#links)
- [Presto Installation Directory Structure on YARN-Based Clusters](https://prestodb.io/presto-yarn/installation-yarn-directory-structure.html)
- Presto Configuration Options for YARN-Based Clusters
  - [appConfig.json](https://prestodb.io/presto-yarn/installation-yarn-configuration-options.html#appconfig-json)
  - [resources.json](https://prestodb.io/presto-yarn/installation-yarn-configuration-options.html#resources-json)
- Advanced Configuration Options for YARN-Based Clusters
  - [Configuring Memory, CPU, and YARN CGroups](https://prestodb.io/presto-yarn/installation-yarn-configuration-options-advanced.html#configuring-memory-cpu-and-yarn-cgroups)
  - [Failure Policy](https://prestodb.io/presto-yarn/installation-yarn-configuration-options-advanced.html#failure-policy)
  - [Using YARN label](https://prestodb.io/presto-yarn/installation-yarn-configuration-options-advanced.html#using-yarn-label)
- [Debugging and Logging for a YARN-Based Cluster](https://prestodb.io/presto-yarn/installation-yarn-debugging-logging.html)
- Developers
  - [Create Presto App Package](https://prestodb.io/presto-yarn/developers.html#create-presto-app-package)