# Q1: Configure an identity provider
## Configure your OpenShift cluster to use an HTPasswd identity provider with the following requirements:
- The name of the identity provider is: ex280-htpasswd
- The name of the secret is: ex280-idp-secret
- The user account armstrong=indionce
- The user account collins=veraster
- The user account aldrin=roonkere
- The user account jobs=sestiver
- The user account wozniak=glegunge

## Create a textfile loke "vim mypass.txt" and `Source` the mypass.txt file to set the environment variables:
```shell
source mypass.txt
```
## Log in to your OpenShift cluster as a user with administrator privileges:
```shell
oc login -u kubeadmin -p ${kube_pass} ${api_url} 
```
## To Create an `HTPasswd file` for the users, use the following command:

```shell
htpasswd -c -B -b </path/to/users.htpasswd> <user_name> <password>
```
Run the following command to create the file named htpasswd-file:
```shell
htpasswd -c -B -b htpasswd-file-upload armstrong indionce
htpasswd    -B -b htpasswd-file-upload collins veraster
htpasswd    -B -b htpasswd-file-upload aldrin roonkere
htpasswd    -B -b htpasswd-file-upload jobs sestiver
htpasswd    -B -b htpasswd-file-upload wozniak glegunge
```
## Create a `secret` from the `htpasswd-file`:
```shell
oc create secret generic htpass-secret --from-file=htpasswd=<path_to_users.htpasswd> -n openshift-config
```
Create a secret named `ex280-idp-secret`
```shell
oc create secret generic ex280-idp-secret --from-file=htpasswd=htpasswd-file-upload -n openshift-config 
```
## Download a copy of the `oauth/cluster` resource from your OpenShift cluster:
```shell
oc get oauth/cluster -o yaml> oauth.yaml
```
## `Edit the oauth.yaml` file and replace the spec.identityProviders section with the help od documentatiom:
ref: Authentication and authorization document --> 7.1.5. Sample htpasswd CR
```yaml 
spec:
  identityProviders:
  - name: my_htpasswd_provider 1
    mappingMethod: claim 2
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret 3
```
copy the content and follow the strps
```shell
vim oauth.yaml
```
`esc`--> type `:set paste` --> `enter` --> `insert`  --> then paste the content for correct indent pasting

then edit the yaml file to look like the following 
```yaml
spec:
  identityProviders:
  - name: ex280-htpasswd
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: ex280-idp-secret
```
## Replace the oauth/cluster resource with the modified oauth.yaml file:
```shell
oc replace  -f oauth.yaml
```
## (Optional) Create aliases for quick `login as each user`:
```shell
alias _kube="oc login -u kubeadmin -p ${kube_pass} ${api_url}"
alias _armstrong="oc login -u armstrong -p ${armstrong} ${api_url}"
alias _collins="oc login -u collins -p ${collins} ${api_url}"
alias _aldrin="oc login -u aldrin -p ${aldrin} ${api_url}"
alias _jobs="oc login -u jobs -p ${jobs} ${api_url}"
alias _wozniak="oc login -u wozniak -p ${wozniak} ${api_url}"
```
## Log in as each user to `test` authentication:
```shell
_armstrong;_collins;_wozniak;_aldrin;_jobs;
```
## Log back in as the administrator to `verify` the users and identities:
```shell
_kube
oc get users
oc get identity
```

## Conclusion

Configured an HTPasswd identity provider on an OpenShift cluster and test authentication with several user accounts. This was accomplished by creating an HTPasswd file with user accounts and passwords, creating a secret from the file, editing the oauth/cluster resource to use the HTPasswd identity provider, and testing authentication with several user accounts. 

# Q2: Configure cluster permissions

This section describes how to configure an OpenShift cluster to meet the following requirements:

- The user account jobs can perform cluster administration tasks
- The user account wozniak can create projects
- The user account wozniak cannot perform cluster administration tasks
- The user account armstrong cannot create projects
- The user account kubeadmin is not present

## Adding a user to cluster-admin role for jobs

To grant cluster-admin access to a user, use the following command:

```shell
oc adm policy add-cluster-role-to-user <user-name>
```

To add a user to the `cluster-admin` role for `jobs`, use the following command:

```shell
oc adm policy add-cluster-role-to-user cluster-admin jobs
```

## Logging in as a jobs user

To log in as a `jobs` user, use the following command:

```shell
_jobs
```

## Removing users from creating new projects

To `remove every user from creating a new project`, use the following commands:

```shell
oc describe clusterrolebindings.rbac.authorization.k8s.io | grep self-pro
oc describe clusterrolebindings.rbac.authorization.k8s.io self-provisioner
oc adm policy remove-cluster-role-from-group self-provisioner system:authenticated:oauth
```

## Adding a user to the self-provisioner role to create projects

To grant project creation access to a user, use the following command:

```shell
oc adm policy add-cluster-role-to-user self-provisioner <user-name>
```
To add a `wozniak` user to the self-provisioner role to create projects, use the following command:

```shell
oc adm policy add-cluster-role-to-user self-provisioner wozniak
```

## Preventing users from performing cluster administration tasks

To prevent users from performing cluster administration tasks, such as the user account `wozniak`, use the following commands:

```shell
oc adm policy remove-cluster-role-from-user cluster-admin wozniak
```

## Preventing a user from creating projects

To prevent a `armstrong` user from creating projects, such as the user account armstrong, use the following commands:

```shell
oc adm policy remove-cluster-role-from-user self-provisioner armstrong
```

## Removing kubeadmin

To `remove kubeadmin`, use the following command:

```shell
oc delete secrets kubeadmin -n kube-system
```

## Conclusion

These commands will help you configure your OpenShift cluster with the specified user account permissions in cluster level and remove the kubeadmin user account.


# Q3: Configure your OpenShift cluster to meet the following requirements:
## The following projects exist:
- apollo
- manhattan
- gemini
- bluebook
- titan

The user account armstrong is an administrator for project apollo and project gemini

The user account wozniak can view project titan but not administer or delete it 


## To create a new project in OpenShift, use the following command: 

```shell
oc new-project <project-name>
```
To create projects mentioned above
```shell
oc new-project apollo
oc new-project manhattan
oc new-project gemini
oc new-project bluebook
oc new-project titan
```
## Administrator Access for Armstrong

The user account `armstrong` must be granted administrator access for the `apollo` and `gemini` projects. 

To grant admin access to a user, use the following command:

```shell
oc adm policy add-role-to-user admin armstrong -n <project-name>
```

In this project, the following commands must be used:

```shell
oc adm policy add-role-to-user admin armstrong -n apollo
oc adm policy add-role-to-user admin armstrong -n gemini
```

## Viewer Access for Wozniak

The user account `wozniak` must be able to view the `titan` project but must not be able to administer or delete it.

To grant view access to a user, use the following command:

```shell
oc adm policy add-role-to-user view wozniak -n <project-name>
```

In this project, the following command must be used:

```shell
oc adm policy add-role-to-user view wozniak -n titan
```

## Conclusion

This project requires the creation of multiple OpenShift projects and the granting of administrator and viewer access to specific users. The `oc adm policy` command is used to configure project permissions in OpenShift.

# Q4: Configuring Groups in OpenShift

This guide explains how to configure groups in OpenShift to meet the following requirements:
- The user account `armstrong` is a member of the `commander` group
- The user account `collins` is a member of the `pilot` group
- The user account `aldrin` is a member of the `pilot` group
- Members of the `commander` group have `edit` permission in the `apollo` project
- Members of the `pilot` group have `view` permission in the `apollo` project


## Steps

1. Create new groups:
   ```shell
   oc adm groups new <group-name>
   ```
   To Create group `commander` and `pilot`, use the following command:
   ```shell
   oc adm groups new commander
   oc adm groups new pilot
   ```

2. Add users to the group:
   ```shell
   oc adm groups add-users <group-name> <user-name> <user-name>
   ```
   To add a user to the `armstrong`,`collins`and`aldrin` to their respective groups, use the following command:
   ```shell
   oc adm groups add-users commander armstrong
   oc adm groups add-users pilot collins aldrin
   ```

3. Verify the groups and users:
   ```shell
   oc get groups
   ```

4. Assign roles to the groups:
   ```shell
   oc adm policy add-role-to-group edit commander -n apollo
   oc adm policy add-role-to-group view pilot -n apollo
   ```

## Conclusion

By following these steps, you have configured groups in OpenShift to meet the specified requirements. Members of the `commander` group have `edit` permission in the `apollo` project, while members of the `pilot` group have `view` permission in the `apollo` project.

## Q5 OpenShift Cluster Quotas Configuration

This project configures quotas for the OpenShift cluster in the manhattan project. The quotas are configured as follows:

- The ResourceQuota resource name is `ex280-quota`.
- The total amount of memory consumed across all containers may not exceed 1Gi.
- The total amount of CPU usage consumed across all containers may not exceed 2 full cores.
- The maximum number of replication controllers does not exceed 3.
- The maximum number of pods does not exceed 3.
- The maximum number of services does not exceed 6.

### Configuration Steps

To configure quotas in the OpenShift cluster, follow these steps:

1. Create a new quota using the `oc create quota` command. In this project, we used the following command:

   ```shell
   oc create quota --help
   ```
   get the first example and modify it to
   
   ```shell
   oc create quota ex280-quota --hard=cpu=2,memory=1Gi,pods=3,services=6,replicationcontrollers=3 -n manhattan
   ```
2. Verify the quota using the `oc get resourcequota` command. In this project, we used the following command:
   ```shell
   oc get resourcequotas -n manhattan
   ```
   
### Conclusion
Configuring quotas is an important step in managing resource usage in an OpenShift cluster. By following the steps outlined in this project, you can configure quotas for your project and ensure that resource usage is controlled within the specified limits.

# Q6: Configure limits

Configure your OpenShift cluster to use limits in the bluebook project with the following requirements:

-  The name of the LimitRange resource is: ex280-limits
-  The amount of memory consumed by a single pod is between 5Mi and 300Mi
-  The amount of memory consumed by a single container is between 5Mi and 300Mi with a default request value of 100Mi
-   The amount of CPU consumed by a single pod is between 10m and 500m
-   The amount of CPU consumed by a single container is between 10m and 500m with a default request value of 100m

## Create a YAML file named limit.yaml with the following content:
```shell
vim limit.yaml
```
```yaml
apiVersion: "v1"
kind: "LimitRange"
metadata:
  name: "ex280-limits"
spec:
  limits:
    - type: "Pod"
      max:
        cpu: "500m"
        memory: "300Mi"
      min:
        cpu: "10m"
        memory: "5Mi"
    - type: "Container"
      max:
        cpu: "500m"
        memory: "300Mi"
      min:
        cpu: "10m"
        memory: "5Mi"
      defaultRequest:
        cpu: "100m"
        memory: "100Mi"
```

## Run the following command to create the ex280-limits LimitRange resource and save it to the bluebook project:
```shell
oc create -f limit.yaml --save-config -n bluebook
```
## Verify that the LimitRange has been created by running the following command:
```shell
oc describe limitranges -n bluebook
```
## Conclusion
In summary, configuring limits for resources in an OpenShift cluster is important for efficient resource management and allocation. With the use of LimitRange resources, it is possible to set limits on the amount of memory and CPU consumed by pods and containers, thereby preventing resource exhaustion and ensuring better application performance.

# Q7: Deploy an application

Deploy the application called rocky in the bullwinkle project so that the following conditions are true:

- The application is reachable at the following address:
- http://rocky.apps.domainXX.example.com
- The application produces output

## First get inside the project by following command:
```shell
oc project bullwinkle
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Check which nodes have been tainted:
```shell
oc describe node | grep -i taint
```
## To remove the taint from node by running the help command:
```shell
oc adm taint nodes --help
```
Select the suitable example 
```shell
oc adm taint nodes worker0 key1=value1:NoSchedule-
oc adm taint nodes worker1 key1=value1:NoSchedule-
```
This removes the key1=value1:NoSchedule taint from the worker0 node. You may need to run this command on all tainted nodes.

check taint is removed or not 
```shell
oc describe node | grep -i taint
```
## application produces output at given link
Delete any existing default route for the rocky service:
```shell
oc delete routes rocky
```
Expose the rocky service at http://rocky.apps.domainXX.example.com:
```shell
oc expose service rocky --hostname=rocky.apps.domainXX.example.com
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
deploy the rocky application in the bullwinkle project and make it reachable at http://rocky.apps.domainXX.example.com, we need to ensure that all the required resources are present in the project, remove any taints from the nodes (if needed), delete any existing default route for the rocky service, and expose the service at the specified hostname. 

# Q8: Scale an application manually

Manually scale the minion application in the gru project to a total of 5 replicas.

## First get inside the project by following command:
```shell
oc project gru
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Manual Scaling with help command:
```shell
oc scale --help
oc scale --replicas=5 dc minion
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
you can manually scale the minion application in the gru project to a total of 5 replicas using the oc scale command.

# Q9: Scale an application automatically

Automatically scale the hydra application deployment configuration in the lerna project with the following requirements:
- Minimum number of replicas: 6
- Maximum number of replicas: 9
- Target average CPU utilization: 60 percent
- Container CPU resource request: 25m
- Container CPU resource limit: 100m

## First, you need to access the Lerna project using the following command:
```shell
oc project lerna
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Auto Scaling with help command:
```shell
oc autoscale  --help
oc autoscale dc hydra --max=9 --min=6  --cpu-percent=60
```

## Scale the Container CPU resource with help command:
```shell
oc set resources --help
oc set resources dc hydra --limits=cpu=100m --requests=cpu=25m
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
automatically scaling the Hydra application deployment configuration in the Lerna project involves setting a minimum and maximum number of replicas, target average CPU utilization, and container CPU resource request and limit. The process can be achieved through the use of the oc autoscale and oc set resources commands, 

# Q10: Configure a secure route

Configure the oxcart application in the area51 project with the following requirements:
- The application uses a route called oxcart
- The application uses a CA signed certificate with the following subject fields:
- /C=US/ST=NV/L=Hiko/O=CIA/OU=USAF/CN=classified.apps.domainXX.example.com
- The application is reachable only at the following address:
- https://classified.apps.domainXX.example.com
- The application produces output

- A utility script called newcert has been provided to create the CA signed certificate. You may enter the certificate parameters manually or pass the subject as a parameter.
- Your certificate signing request will be uploaded to the CA where it will immediately be signed and then downloaded to your current directory.


## First, you need to access the area51 project using the following command:
```shell
oc project area51
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Delete the exisiting route command:
```shell
oc delete route oxcart
```

## Generate key and cerificate `manually` help command:
```shell
openssl genrsa -out ex280.key 2048
openssl req -new -key ex280.key -out ex280.csr -subj "/C=US/ST=NV/L=Hiko/O=CIA/OU=USAF/CN=classified.apps.domainXX.example.com" 
openssl x509 -req -in ex280.csr -signkey ex280.key -out ex280.crt
```
## `OR` Generate key and cerificate with the help of newcert application given in exam:
```shell
newcert
```
Expose the oxcart service at https://classified.apps.domainxx.example.com with key and certificate
```shell
oc create route edge oxcart --service oxcart --key ex280.key --cert ex280.crt --hostname classified.apps.domainxx.example.com
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
configure a secure route for the oxcart application in the area51 project with the specified requirements, you need to generate a key and a certificate with OpenSSL or using the newcert application provided in the exam.

# Q11: Configure a secret

Configure a secret in the math project with the following requirements:
- The name of the secret is: magic
- The secret defines a key with name: decoder_ring
- The secret defines the key with value: XpWy9KdcP3Tr9FFHGQgZgVRCKukQdrQsbcl0c2ZYhDk=

## First, you need to access the math project using the following command:
```shell
oc project math
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Create the secret magic with the given key-value pair:
```shell
oc get secret 
oc create secret generic magic --from-literal key=decoder_ring --from-literal value=XpWy9KdcP3Tr9FFHGQgZgVRCKukQdrQsbcl0c2ZYhDk= -n math
```
## Verify that the secret has been created:
```shell
oc get secret -n math
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
created the secret magic with the given key-value pair using the oc create secret command

# Q12: Configure an application to use a secret

Configure the application called qed in the math project with the following requirements:
- The application uses the secret previously created called: magic
- The secret defines an environment variable with name: DECODER_RING
- The application output no longer displays: Sorry, application is not configured correctly.

## First, you need to access the math project using the following command:
```shell
oc project math
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Deploy the secret in the application:
```shell
oc set env --help
oc set env --from=secret/magic dc qed
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
 configure an application to use a secret in OpenShift. Specifically, you configured the "qed" application in the "math" project to use the "magic" secret, This was achieved by using the "oc set env" command to deploy the secret and set the environment variable in the deployment configuration.

# Q13: Configure a service account

Configure a service account in the apples project to meet the following requirements:
- The name of the account is ex280sa
- The account allows pods to be run as any available user 

## First, you need to access the apples project using the following command:
```shell
oc project apples
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Create a service account named `ex280sa` and make it available for any user:
```shell
oc create serviceaccount ex280sa
oc adm policy add-scc-to-user anyuid -z ex280sa
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
 configure a service account in the apples project that meets the requirements of allowing pods to be run as any available user. The steps involve creating a service account named ex280sa and adding the anyuid security context constraint to it

# Q14: Deploy an application

Deploy the application called oranges in the apples project so that the following conditions are true:
- The application uses the ex280sa service account
- No additional configuration components have been added or removed
- The application produces output

## First, you need to access the apples project using the following command:
```shell
oc project apples
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## deploy service account named `ex280sa` in `oranges` application:
```shell
oc set serviceaccount dc oranges ex280sa
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```

## Check end point by following command:
```shell
oc describe svc oranges
```
Check `endpoints` is availble in svc
```shell
oc edit  svc oranges
```
Change "Selector:          deploymentconfig=orange" --> "Selector:          deploymentconfig=oranges"

```yaml
Name:              oranges
Namespace:         apples
Labels:            app=oranges
Annotations:       <none>
Selector:          deploymentconfig=orange
Type:              ClusterIP
IP:                172.30.236.171
Port:              http  8080/TCP
TargetPort:        8080/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>
```
to 
```yaml
Name:              oranges
Namespace:         apples
Labels:            app=oranges
Annotations:       <none>
Selector:          deploymentconfig=oranges
Type:              ClusterIP
IP:                172.30.236.171
Port:              http  8080/TCP
TargetPort:        8080/TCP
Endpoints:         10.128.0.10:8080,10.128.1.13:8080
Session Affinity:  None
Events:            <none>

```

## Conclusion
 configured a service account in the apples project with the name ex280sa and allowed pods to be run as any available user. We also edited the oranges service to update its selector to match the oranges deployment configuration.

# Q15: Deploy an application

Deploy the application called voyager in the pathfinder project so that the following conditions are true:
- No configuration components have been added or removed
- The application produces output  

## First, you need to access the pathfinder project using the following command:
```shell
oc project pathfinder
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Check the worker node by command:
```shell
oc describe  node worker0.domain2.example.com
```
following yaml will be resulted
```yaml
Name:               worker0.domainXX.example.com
Roles:              worker
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=worker0.domainXX.example.com
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/worker=
                    node.openshift.io/os_id=rhcos
                    star=trek
Annotations:        machineconfiguration.openshift.io/controlPlaneTopology: HighlyAvailable
                    machineconfiguration.openshift.io/currentConfig: rendered-worker-a43dea2cdc8ee25f6cac55211539c197
                    machineconfiguration.openshift.io/desiredConfig: rendered-worker-a43dea2cdc8ee25f6cac55211539c197
                    machineconfiguration.openshift.io/reason: 
                    machineconfiguration.openshift.io/state: Done
                    volumes.kubernetes.io/controller-managed-attach-detach: true
```
Change `star=trek` in node level by:
```shell
oc label node worker0 star=trek --overwrite
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
checked if the required resources were present in the project, and verified that the worker node was properly labeled. Finally, we changed the label of the star attribute on the worker0 node using the oc label command with the --overwrite flag. It's important to carefully plan and understand the impact of such changes, as they can affect the scheduling of pods and the allocation of resources in the cluster.

# Q16: Deploy an application

Deploy the application called atlas in the mercury project so that the following conditions are true:
- No configuration components have been added or removed
- The application produces output  

## First, you need to access the mercury project using the following command:
```shell
oc project mercury
```
## Check if the required resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Check the events for any memory error:
```shell
oc get events | grep -i menory 
```
## Describe the application
```shell
oc describe dc atlas
```
following yaml will be resulted
```yaml
Name:		atlas
Namespace:	mercury
Labels:		app=atlas
		app.kubernetes.io/component=atlas
		app.kubernetes.io/instance=atlas
Annotations:	openshift.io/generated-by=OpenShiftNewApp
Latest Version:	2
Selector:	deploymentconfig=atlas
Replicas:	1
Triggers:	Config, Image(atlas@latest, auto=true)
Strategy:	Rolling
Template:
Pod Template:
  Labels:	deploymentconfig=atlas
  Annotations:	openshift.io/generated-by: OpenShiftNewApp
  Containers:
   atlas:
    Image:	registry.domainXX.example.com/openshift/hello-openshift@sha256:aaea76ff622d2f8bcb32e538e7b3cd0ef6d291953f3e7c9f556c1ba5baf47e2e
    Ports:	8080/TCP, 8888/TCP
    Host Ports:	0/TCP, 0/TCP
    Requests:
      memory:	80Gi
    Environment:
      RESPONSE:	<set to the key 'RESPONSE' of config map 'nasa'>	Optional: false
    Mounts:	<none>
  Volumes:	<none>
```
Change `80Gi` to `80Mi`
```shell
oc set resources dc atlas  --resources=memory=80Mi
```
## Verify resources are present in the project:
```shell
oc get pods
oc get service
oc get deployment
oc get dc
oc get route
oc get events
oc get all
```
## Conclusion
deploying an application involves making sure that all the required resources and configurations are in place, setting appropriate resource limits, deploying the application, and verifying that it produces output. 

 









