# webmethods API Gateway Containers

**NOTE**: This page is specically for **Version 10.7** of SoftwareAG webmethods API Gateway

## SoftwareAG Product Documentation

Find the official SoftwareAG webMethods APIGateway product documentation here: 

[webmethods API Gateway 10.7 Product documentation](https://documentation.softwareag.com/webmethods/api_gateway/yai10-7/10-7_API_Gateway_webhelp/index.html)

Specifically, in the context of containers, here are 2 important sections in the documentation:

- [webmethods API Gateway 10.7 Product documentation > Docker Configuration](https://documentation.softwareag.com/webmethods/api_gateway/yai10-7/10-7_API_Gateway_webhelp/index.html#page/api-gateway-integrated-webhelp%2F_api_gtw_integrated_webhelp_diba2.1.130.html%23)

- [webmethods API Gateway 10.7 Product documentation > Kubernetes Support](https://documentation.softwareag.com/webmethods/api_gateway/yai10-7/10-7_API_Gateway_webhelp/index.html#page/api-gateway-integrated-webhelp%2F_api_gtw_integrated_webhelp_diba2.1.144.html%23)

## Introduction

The webMethods APIGateway stack is comprised of: 
- API Gateway core
- Elastic Search (Storage layer required for the storage of runtime configuration)
- Kibana (Dashboarding layer)
- Terracotta BigMemory (Distributed caching layer for state-sharing accross a stateful cluster)

Depending on the deployment type, all these components will be either distributed in their own containers (most likely use case for PROD deployments), or collocated in a single standalone container (easiest to deploy, most likely usage to try the software)

## Container Options

As such, we provide 2 APIGateway containers:
- [webmethods-apigateway](https://github.com/orgs/softwareag-government-solutions/packages/container/package/webmethods-apigateway)
- [webmethods-apigateway-standalone](https://github.com/orgs/softwareag-government-solutions/packages/container/package/webmethods-apigateway-standalone)

### webmethods-apigateway

[webmethods-apigateway](https://github.com/orgs/softwareag-government-solutions/packages/container/package/webmethods-apigateway) is the core apigateway container that only includes the artifacts required to run a webMethods APIGateway engine, without any of the required dependencies.
This container option is ideal for clustered deployments of webMethods APIGateway where the Elastic Stack (Elastic Search, Kibana) is installed, deployed, and running externally to the APIGateway (ie. in their own containers)

The main benefit is that this container is the smallest, without any unneeded components built-in.
The draw-back of this container is that it will not start a fully functionning APIGateway if the required dependencies are not available and configured.

### webmethods-apigateway-standalone

With [webmethods-apigateway-standalone](https://github.com/orgs/softwareag-government-solutions/packages/container/package/webmethods-apigateway-standalone), everything is included in one container. 
The main benefit is that a single instance of this container will start a fully functionning APIGateway.
The draw-back is that it's a larger container that will include unneeded components for a production-like clustered deployment (since in such deployment, Elastic stack should be running separately, for example in its own cluster of containers)

## Container Parameters

Non-Root user used by the container: saguser (id=1724)

### Core Environment variables

| ENV variables           | Defaults | Description                                                                                                |
|-------------------------|----------|------------------------------------------------------------------------------------------------------------|
| JAVA_MIN_MEM            |  512m    | Minimum Assigned Memory for the Runtime                                                                    |
| JAVA_MAX_MEM            |  1024m   | Maximum Assigned Memory for the Runtime                                                                    |
| JAVA_OPTS               | -        | Java properties for the runtime                                                                            |
| RUNTIME_WATT_PROPERTIES | -        | APIGateway Runtime WATT properties - space separated list of   properties                                  |
| EXTERN_ELASTICSEARCH    | TRUE*    | Whether to enable APIGateway internal Elastic Search - *   default to "false" for the Standalone container |
| PORT_RUNTIME            | 5555     | The APIGateway runtime port (plain) - used for API calls                                                   |
| PORT_RUNTIME_SSL        | 5543     | The APIGateway runtime port (SSL) - used for API calls                                                     |
| PORT_GATEWAYUI          | 9072     | The APIGateway UI port (plain) - use strictly for the   Administration UI                                  |
| PORT_GATEWAYUI_SSL      | 9073     | The APIGateway UI port (SSL) - use strictly for the   Administration UI                                    |
| IDS_HEAP_SIZE           | -        | This property will set the HEAP memory used by the internal Elastic search (if using it)                   |
| IDS_PROPERTIES_ADD      | -        | Space separated list of specific Elastic Search properties to add to the internal Elastic search (if using it)      |
| IDS_PROPERTIES_REMOVE   | -        | Space separated list of specific Elastic Search properties to remove from the internal Elastic search (if using it) |

### APIGateway External Configuration By File

The container supports the setup of an external configuration file as explained:

[webmethods API Gateway 10.7 Product documentation > Externalizing Configurations](https://documentation.softwareag.com/webmethods/api_gateway/yai10-7/10-7_API_Gateway_webhelp/index.html#page/api-gateway-integrated-webhelp%2Fco-externalizing_configurations_overview.html%23)

To load external configuration files into the gateway, simply mount well formatted configuration files (either YAML or PROPERTIES, as defined by the above product documentation) into the container at the following PATH: /configs

For example, in docker, you would create a local folder containing some well formatted configuration files (either YAML or PROPERTIES, as defined by the above product documentation), and mount the directory at docker run command using the "mount volume" argument... ie:

```
docker run -v /my/local/configs:/configs ${REG}webmethods-apigateway:prod-10.7-latest
```

As stated in the documentation, a sample external configuration file is as follows:

```
apigw:
  elasticsearch:
    tenantId: apigateway
    hosts: localhost:9200
    autostart: false
    http:
      username: elastic
      password: changeme
      keepAlive: true
      keepAliveMaxConnections: 10
      keepAliveMaxConnectionsPerRoute: 100
      connectionTimeout: 1000
      socketTimeout: 10000
      maxRetryTimeout: 100000
    https:
      enabled: false
      keystoreFilepath: C:/softwares/elasticsearch/config/keystore-new.jks
      truststoreFilepath: C:/softwares/elasticsearch/config/truststore-new.ks
      keystoreAlias: root-ca
      keystorePassword: 6572b9b06156a0ff778c
      truststorePassword: manage
      enforceHostnameVerification: false
    sniff:
      enable: false
      timeInterval: 1000
    outboundProxy:
      enabled: false
      alias: somealias
    clientHttpResponseSize: 1001231
```

### APIGateway External Configuration By Environment variables

All the external configurations can also be set via environment variables.

NOTE: It is possible to setup **both a configuration file AND environment Variables** (ie. for default values that change less frequently are setup in a file, and the changing values are setup in environment variables). If a value is specified in both a config file AND an environment variable, the environment variables is the one that wins.

The environment variables follow the configuration file structure with the following format:
 <parent>_<child>_<child>=<value>

For example, to specify an external elastic search, the following environment variables would be specified on the container:

```
apigw_elasticsearch_hosts=host:port
apigw_elasticsearch_https_enabled=("true" or "false")
apigw_elasticsearch_http_username=user
apigw_elasticsearch_http_password=password
```
