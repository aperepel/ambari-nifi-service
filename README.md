#### An Ambari Service for NiFi
Ambari service for easily installing and managing NiFi on HDP cluster

Author: [Ali Bajwa](https://www.linkedin.com/in/aliabajwa)

#### Setup

- Download HDP 2.3 sandbox VM image (Sandbox_HDP_2.3_VMware.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Sandbox_HDP_2.3_VMware.ova into VMWare and set the VM memory size to 8GB
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and start Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```

- To download the NiFi service folder, run below
```
cd /var/lib/ambari-server/resources/stacks/HDP/2.2/services
git clone https://github.com/abajwa-hw/NiFi-stack.git   
```

- To customize the default users/groups, you can copy the base.ldif/groups.ldif/users.ldif files from ```/var/lib/ambari-server/resources/stacks/HDP/2.2/services/NiFi-stack/package/scripts``` dir of ambari server into any dir (e.g. /root) and make your changes (e.g. replace ali user with tom user). In Ambari add services wizard below when you get to configuration page, just point ldap.ldifdir property to this directory. 

- Restart Ambari
```
sudo service ambari-server restart
```

- Then you can click on 'Add Service' from the 'Actions' dropdown menu in the bottom left of the Ambari dashboard:

On bottom left -> Actions -> Add service -> check NiFi server -> Next -> Next -> Change any config you like (e.g. port) -> Next -> Deploy
![Image](../master/screenshots/screenshot-vnc-config.png?raw=true)

- On successful deployment you will see the NiFi service as part of Ambari stack and will be able to start/stop the service from here:
![Image](../master/screenshots/screenshot-vnc-stack.png?raw=true)

- When you've completed the install process, NiFi server will appear in Ambari 
![Image](../master/screenshots/screenshot-nifi-stack.png?raw=true)

- You can see the parameters you configured under 'Configs' tab
![Image](../master/screenshots/screenshot-nifi-stack-config.png?raw=true)

- One benefit to wrapping the component in Ambari service is that you can now monitor/manage this service remotely via REST API
```
export SERVICE=NIFI
export PASSWORD=admin
export AMBARI_HOST=localhost
export CLUSTER=Sandbox

#get service status
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X GET http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#start service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Start $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "STARTED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE

#stop service
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X PUT -d '{"RequestInfo": {"context" :"Stop $SERVICE via REST"}, "Body": {"ServiceInfo": {"state": "INSTALLED"}}}' http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE
```



#### Browse users

- The NiFi webUI login page should come up at the below link: 
http://sandbox.hortonworks.com:9090/nifi

- You can also open it from within Ambari via [iFrame view](https://github.com/abajwa-hw/iframe-view)
![Image](../master/screenshots/screenshot-nifi-view.png?raw=true)

#### Remove service

- To remove the Nifi service: 
  - Stop the service via Ambari
  - Unregister the service
  
    ```
export SERVICE=NIFI
export PASSWORD=admin
export AMBARI_HOST=localhost
export CLUSTER=Sandbox    
curl -u admin:$PASSWORD -i -H 'X-Requested-By: ambari' -X DELETE http://$AMBARI_HOST:8080/api/v1/clusters/$CLUSTER/services/$SERVICE
    ```
   - Clear LDAP dir to reset the data in LDAP
```
rm -rf /opt/nifi*
```   