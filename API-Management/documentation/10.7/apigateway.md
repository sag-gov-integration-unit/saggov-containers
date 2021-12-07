# webmethods API Gateway Containers

## Product Documentation

## Introduction

The webMethods APIGateway stack is comprised of: 
- API Gateway core
- Elastic Search (Storage layer required for the storage of runtime configuration)
- Kibana (Dashboarding layer)
- Terracotta BigMemory (Distributed caching layer for state-sharing accross a stateful cluster)

Depending on the deployment type, all these components will be either distributed in their own containers (most likely use case for PROD deployments), or collocated in a single standalone container (easiest to deploy, most likely usage to try the software)

## Container Options

As such, we provide 2 APIGateway containers:
- webmethods-apigateway
- webmethods-apigateway-standalone

### webmethods-apigateway

This is the core apigateway container that only includes the artifacts required to run a webMethods APIGateway engine, without any of the required dependencies.
This container option is ideal for clustered deployments of webMethods APIGateway where the Elastic Stack (Elastic Search, Kibana) is installed, deployed, and running externally to the APIGateway (ie. in their own containers)

The main benefit is that this container is the smallest, without any unneeded components built-in.
The draw-back of this container is that it will not start a fully functionning APIGateway if the required dependencies are not available and configured.

### webmethods-apigateway-standalone

With this container, everything is included in one container. 
The main benefit is that a single instance of this container will start a fully functionning APIGateway.
The draw-back is that it's a larger container that will include unneeded components for a production-like clustered deployment (since in such deployment, Elastic stack should be running separately, for example in its own cluster of containers)

## Container Parameters

Non-Root user: saguser (id=1724)

Core env:

JAVA_MIN_MEM: 512m
JAVA_MAX_MEM: 512m
JAVA_OPTS: ""
RUNTIME_WATT_PROPERTIES: ""
EXTERN_ELASTICSEARCH="true"
PORT_RUNTIME="${gateway_runtime_port}"
PORT_RUNTIME_SSL="${gateway_runtime_port_ssl}"
PORT_GATEWAYUI="${gateway_port}"
PORT_GATEWAYUI_SSL="${gateway_port_ssl}"

APIGateway external config envs:

      APIGW_ELASTICSEARCH_TENANTID: apigateway
      APIGW_ELASTICSEARCH_HOSTS: elasticsearch:9200
      APIGW_ELASTICSEARCH_AUTOSTART: "false"
      APIGW_ELASTICSEARCH_HTTP_USERNAME: elastic
      APIGW_ELASTICSEARCH_HTTP_PASSWORD: SomeStrongPassword!
      APIGW_ELASTICSEARCH_HTTPS_ENABLED: "false"
      APIGW_KIBANA_DASHBOARDINSTANCE: http://kibana:5601
      APIGW_KIBANA_AUTOSTART: "false"
      CLUSTER_AWARE: "true"
      CLUSTER_NAME: APIGatewayTSAcluster
      CLUSTER_TSAURLS: terracotta:9510
      CLUSTER_SESSTIMEOUT: 20
      CLUSTER_ACTIONONSTARTUPERROR: shutdown
      CLUSTER_CONNECTTIMEOUT: 30000


