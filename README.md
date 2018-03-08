# ambari-api-work
Using Ambari API to add new service on existing cluster and modify properties of a service

## Add new service to the existing 

Below is an example to add TEZ service to the existing cluster

**Versions:** HDP2.6.4 on Ambari 2.6.1

**ambari-server-host:** *pavan0.field.hortonworks.com*

**cluster-name:** *test*


##### Step 1 - Add a Service to the cluster
```$ curl -u <username>:<password> -i -X POST -d '{"ServiceInfo":{"service_name":"<service-name>"}}' http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name>/services```

Example:

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d '{"ServiceInfo":{"service_name":"TEZ"}}' http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/services```


##### Step 2 - Add Components to the service
```$ curl -u <username>:<password> -i -X POST http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name>/services/<service-name>/components/<component-name>```

Example:

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/services/TEZ/components/TEZ_CLIENT```

##### Step 3 - Get configuration from existing cluster

```$ curl -u admin:admin -H "X-Requested-By: ambari" -X GET "http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations?type=tez-env&tag=version1" > tez-env.json ```

```$ curl -u admin:admin -H "X-Requested-By: ambari" -X GET "http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations?type=tez-interactive-site&tag=version1" > tez-interactive-site.json ```

```$ curl -u admin:admin -H "X-Requested-By: ambari" -X GET "http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations?type=tez-site&tag=version1" > tez-site.json```


##### Step 4 - Create configuration
```$ curl -u <username>:<password> -i -X POST -d '{"type": "<config-type>", "tag": "<config-tag>", "properties" : { "key1" : "value1", "key2" : "value2", "key3" : "value3"  }}' http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name>/configurations```

Example:

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d @tez-env.json http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations```

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d @tez-interactive-site.json http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations```

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d @tez-site.json http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations```

*Note:* "type", "tag" and "properties" are already extracted to json files (tez-env.json, tez-interactive-site.json, tez-site.json) in step 3

##### Step 5 - Apply configuration to the cluster
```$ curl -u <username>:<password> -i -X PUT -d '{"Clusters": {"desired_configs": { "type": "<config-type>", "tag" :"<config-tag>" }}}' http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name> ```

Example:

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X PUT -d '{ "Clusters" : {"desired_configs": {"type": "tez-env", "tag" : "version1" }}}'  http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test```

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X PUT -d '{ "Clusters" : {"desired_configs": {"type": "tez-interactive-site", "tag" : "version1" }}}'  http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test```

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X PUT -d '{ "Clusters" : {"desired_configs": {"type": "tez-site", "tag" : "version1" }}}'  http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test```

##### Step 6 - Create host components
```$ curl -u <username>:<password> -i -X POST -d '{"host_components" : [{"HostRoles":{"component_name":"<component-name>"}}] }' http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name>/hosts?Hosts/host_name=<host-names>```

Example:

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d '{"host_components" : [{"HostRoles":{"component_name":"TEZ_CLIENT"}}] }' http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/hosts?Hosts/host_name=pavan0.field.hortonworks.com```

Note: This is the same hostname used during registration of the host. Multiple hosts can be assigned the same Role using this API

##### Step 7 - Install & Start the service
Note: Make sure the service components are created and the configurations attached by making a GET call.

```http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name>/services/<service-name>```

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X GET http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/services/TEZ```

###### Install the service.
```$ curl -u <username>:<password> -i -X PUT -d '{"ServiceInfo": {"state" : "INSTALLED"}}'  http://<ambari-server-host>:8080/api/v1/clusters/<cluster-name>/services/<service-name>```

Example:

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X PUT -d '{"ServiceInfo": {"state" : "INSTALLED"}}'  http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/services/TEZ```

###### Restart all required services
```$ curl  -u admin:admin -H "X-Requested-By: ambari" -X POST  -d '{"RequestInfo":{"command":"RESTART","context":"Restart all required services","operation_level":"host_component"},"Requests/resource_filters":[{"hosts_predicate":"HostRoles/stale_configs=true"}]}' http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/requests```

Reference: 

https://cwiki.apache.org/confluence/display/AMBARI/Adding+a+New+Service+to+an+Existing+Cluster


## Modify Configurations
Modifying the propertiy "yarn.timeline-service.leveldb-timeline-store.ttl-interval-ms" value to "333333" under YARN site configuration

#####1. Find the latest version of the config type that you need to update.
```$ curl -u admin:admin -H "X-Requested-By: ambari" -X GET  http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test?fields=Clusters/desired_configs```

#####2. Read the config type with correct tag
```$ curl -u admin:admin -H "X-Requested-By: ambari" -X GET "http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations?type=yarn-site&tag=version1" > yarn-site.json```

###### Remove all tags except "type", "tag" and "properties" in yarn-site.json
###### modify the properties eg: "yarn.timeline-service.leveldb-timeline-store.ttl-interval-ms" : "333333"

#####3. Save a new version of the config and apply it as desired config
```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X POST -d @yarn-site.json http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/configurations```

```$ curl -u admin:admin -i -H 'X-Requested-By: ambari' -X PUT -d '{ "Clusters" : {"desired_configs": {"type": "yarn-site", "tag" : "version3" }}}'  http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test```

#####4. Restart all requiredd components or services to have the config change take effect
```$ curl  -u admin:admin -H "X-Requested-By: ambari" -X POST  -d '{"RequestInfo":{"command":"RESTART","context":"Restart all required services","operation_level":"host_component"},"Requests/resource_filters":[{"hosts_predicate":"HostRoles/stale_configs=true"}]}' http://pavan0.field.hortonworks.com:8080/api/v1/clusters/test/requests```


Reference: 

https://cwiki.apache.org/confluence/display/AMBARI/Modify+configurations
https://gist.github.com/glinmac/d9b1ce2e47dfbac6ad17