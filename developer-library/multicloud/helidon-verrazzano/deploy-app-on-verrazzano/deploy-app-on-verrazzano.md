# Deploy the Helidon Application on Verrazzano

## Introductions

### Verrazzano and Application Deployment

Verrazzano supports application definition using [Open Application Model (OAM)](https://oam.dev/). Verrrazzano applications are composed of components and application configurations.

When you deploy applications with Verrazzano, the platform sets up connections, network policies, and ingresses in the service mesh, and wires up a monitoring stack to capture the metrics, logs, and traces. Verrazzano employs OAM components to define the functional units of a system that are then assembled and configured by defining associated application configurations.

### Verrazzano components

A Verrazzano OAM component is a [Kubernetes Custom Resource](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) describing an application’s general composition and environment requirements.

The following code shows a simple Helidon application component for Helidon quickstart-mp application used in this lab. This resource describes a component which is implemented by a single Docker image containing a Helidon application exposing a single endpoint.

```yaml
apiVersion: core.oam.dev/v1alpha2
kind: Component
metadata:
  name: hello-helidon-component
  namespace: hello-helidon
spec:
  workload:
    apiVersion: oam.verrazzano.io/v1alpha1
    kind: VerrazzanoHelidonWorkload
    metadata:
      name: hello-helidon-workload
      labels:
        app: hello-helidon
    spec:
      deploymentTemplate:
        metadata:
          name: hello-helidon-deployment
        podSpec:
          containers:
            - name: hello-helidon-container
              image: "iad.ocir.io/id9hokcxpkra/quickstart-mp:1.0"
              ports:
                - containerPort: 8080
                  name: http
```

A brief description of each field of the component:

* **apiVersion** - Version of the component custom resource definition
* **kind** - Standard name of the component custom resource definition
* **metadata.name** - The name used to create the component’s custom resource
* **metadata.namespace** - The namespace used to create this component’s custom resource
* **spec.workload.kind** - VerrazzanoHelidonWorkload defines a stateless workload of Kubernetes
* **spec.workload.spec.deploymentTemplate.podSpec.metadata.name** - The name used to create the stateless workload of Kubernetes
* **spec.workload.spec.deploymentTemplate.podSpec.containers** - The implementation containers
* **spec.workload.spec.deploymentTemplate.podSpec.containers.ports** - Ports exposed by the container

### Verrazzano Application Configurations

A Verrazzano application configuration is a Kubernetes Custom Resource which provides environment specific customizations. The following code shows the application configuration for the Bob's Books example used in this lab. This resource specifies the deployment of the application to the hello-helidon namespace.

Additional runtime features are specified using traits, or runtime overlays that augment the workload. For example, the ingress trait specifies the ingress host and path, while the metrics trait provides the Prometheus scraper used to obtain the application related metrics.

```yaml
apiVersion: core.oam.dev/v1alpha2
kind: ApplicationConfiguration
metadata:
  name: hello-helidon-appconf
  namespace: hello-helidon
  annotations:
    version: v1.0.0
    description: "Hello Helidon application"
spec:
  components:
    - componentName: hello-helidon-component
      traits:
        - trait:
            apiVersion: oam.verrazzano.io/v1alpha1
            kind: MetricsTrait
            spec:
                port: 8080
                scraper: verrazzano-system/vmi-system-prometheus-0
        - trait:
            apiVersion: oam.verrazzano.io/v1alpha1
            kind: IngressTrait
            metadata:
              name: hello-helidon-ingress
            spec:
              rules:
                - paths:
                    - path: "/help/allGreetings"
                      pathType: Prefix
```

A brief description of each field in the application configuration:

* **apiVersion** - Version of the ApplicationConfiguration custom resource definition
* **kind** - Standard name of the application configuration custom resource definition
* **metadata.name** - The name used to create this application configuration resource
* **metadata.namespace** - The namespace used for this application configuration custom resource
* **spec.components** - Reference to the application’s components leveraged to specify runtime configuration
* **spec.components[].traits** - The traits specified for the application’s components

To explore traits, we can examine the fields of an ingress trait:

* **apiVersion** - Version of the OAM trait custom resource definition
* **kind** - IngressTrait is the name of the OAM application ingress trait custom resource definition
* **spec.rules.paths** - The context paths for accessing the application

This lab walks you through the process of deploying the Helidon quickstart-mp application.

### Objectives

In this lab, you will:

* Verify the successful installation of the Verrazzano environment.
* Deploy the Helidon quickstart-mp application.
* Verify the deployment of the Helidon quickstart-mp application.

### Prerequisites

To run this lab, you must have:

* Running Kubernetes (OKE) cluster on the Oracle Cloud Infrastructure.
* (Started) Verrazzano installation on a Kubernetes (OKE) cluster.
* Container packaged Helidon quickstart-mp application available in container registry.

## Task 1: Verification of a successful Verrazzano installation

Open again the Cloud Shell console where you started the Verrazzano installations. If the console broken due to timeout login and start the Cloud Shell again.

To verify Verrazzano installation copy the following command and paste it in the *Cloud Shell*. It checks for the condition, if *InstallComplete* condition is met, and notifies you. Here *my-verrazzano* is the name of the *Verrazzano Custom Resource*.

```bash
<copy>kubectl wait --timeout=10m --for=condition=InstallComplete verrazzano/my-verrazzano</copy>
```
When the process is complete you have to see the `verrazzano.install.verrazzano.io/my-verrazzano condition met` response.

Another option to check the pods associated with Verrazzano have a *Running* status.

```bash
<copy>kubectl get pods -n verrazzano-system</copy>
```
You should get similar result:
```bash
$ kubectl get pods -n verrazzano-system
NAME                                               READY   STATUS    RESTARTS   AGE
coherence-operator-6497f8d5d5-jxrk6                1/1     Running   1          47min
fluentd-k5x9g                                      2/2     Running   2          47min
fluentd-tdsp2                                      2/2     Running   1          47min
fluentd-vq6tb                                      2/2     Running   1          47min
oam-kubernetes-runtime-769d5d779-wrm75             1/1     Running   0          47min
verrazzano-api-b698dfd65-rgcmr                     2/2     Running   0          47min
verrazzano-application-operator-54cfdc77db-lhbkm   1/1     Running   0          47min
verrazzano-console-8677b86b65-jzkbt                2/2     Running   0          47min
verrazzano-monitoring-operator-6458486845-6jzzf    1/1     Running   0          47min
verrazzano-operator-d7576746-nktbj                 1/1     Running   0          47min
vmi-system-es-master-0                             3/3     Running   0          47min
vmi-system-grafana-6c9fd469cd-vnb8h                3/3     Running   0          47min
vmi-system-kibana-676958bc56-7w8b2                 3/3     Running   0          47min
vmi-system-prometheus-0-84897fb854-sgvr4           4/4     Running   0          47min
weblogic-operator-786b8db578-gtf54                 2/2     Running   0          47min
```

Verrazzano installs several consoles. The endpoints for an installation are stored in the `Status` field of the installed Verrazzano Custom Resource.

To get the endpoints for these consoles, copy the following command and paste it in the *Cloud Shell* and look at the `Status.Instance` field:

```bash
<copy>kubectl get vz -o yaml</copy>
```

```bash
$ kubectl get vz -o yaml
apiVersion: v1
items:
- apiVersion: install.verrazzano.io/v1alpha1
  kind: Verrazzano
  metadata:
    annotations:
      kubectl.kubernetes.io/last-applied-configuration: |
        {"apiVersion":"install.verrazzano.io/v1alpha1","kind":"Verrazzano","metadata":{"annotations":{},"name":"my-verrazzano","namespace":"default"},"spec":{"profile":"dev"}}
    creationTimestamp: "2021-08-23T15:19:23Z"
    finalizers:
    - install.verrazzano.io
    generation: 2
    managedFields:
    - apiVersion: install.verrazzano.io/v1alpha1
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:annotations:
            .: {}
            f:kubectl.kubernetes.io/last-applied-configuration: {}
        f:spec:
          .: {}
          f:profile: {}
      manager: kubectl
      operation: Update
      time: "2021-08-23T15:19:23Z"
    - apiVersion: install.verrazzano.io/v1alpha1
      fieldsType: FieldsV1
      fieldsV1:
        f:metadata:
          f:finalizers: {}
        f:spec:
          f:components:
            .: {}
            f:fluentd:
              .: {}
              f:extraVolumeMounts: {}
          f:security: {}
        f:status:
          .: {}
          f:conditions: {}
          f:instance:
            .: {}
            f:consoleUrl: {}
            f:elasticUrl: {}
            f:grafanaUrl: {}
            f:keyCloakUrl: {}
            f:kibanaUrl: {}
            f:prometheusUrl: {}
            f:rancherUrl: {}
          f:state: {}
          f:version: {}
      manager: verrazzano-platform-operator
      operation: Update
      time: "2021-08-23T15:30:06Z"
    name: my-verrazzano
    namespace: default
    resourceVersion: "49073970"
    selfLink: /apis/install.verrazzano.io/v1alpha1/namespaces/default/verrazzanos/my-verrazzano
    uid: c37cc781-1d16-4aa9-8e08-7111d786fe51
  spec:
    components:
      fluentd:
        extraVolumeMounts:
        - source: /u01/data/docker/containers/
    profile: dev
    security: {}
  status:
    conditions:
    - lastTransitionTime: "2021-08-23T15:19:24Z"
      message: Verrazzano install in progress
      status: "True"
      type: InstallStarted
    - lastTransitionTime: "2021-08-23T15:30:05Z"
      message: Verrazzano install completed successfully
      status: "True"
      type: InstallComplete
    instance:
      consoleUrl: https://verrazzano.default.129.146.228.44.nip.io
      elasticUrl: https://elasticsearch.vmi.system.default.129.146.228.44.nip.io
      grafanaUrl: https://grafana.vmi.system.default.129.146.228.44.nip.io
      keyCloakUrl: https://keycloak.default.129.146.228.44.nip.io
      kibanaUrl: https://kibana.vmi.system.default.129.146.228.44.nip.io
      prometheusUrl: https://prometheus.vmi.system.default.129.146.228.44.nip.io
      rancherUrl: https://rancher.default.129.146.228.44.nip.io
    state: Ready
    version: 1.0.0
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

## Task 2: Deploy the Helidon quickstart-mp application

We need to download the source code, where we have configuration files, `bobs-books-app.yaml` and `bobs-books-comp.yaml`.

1. Download the Verrazzano OAM component yaml file and Verrazzano Application Configuration files of Bob's Book example. Click *Copy* and paste the command in the Cloud Shell as shown:

    ```bash
    <copy>
    curl -LSs https://raw.githubusercontent.com/verrazzano/verrazzano/master/examples/bobs-books/bobs-books-app.yaml >~/bobs-books-app.yaml
    curl -LSs https://raw.githubusercontent.com/verrazzano/verrazzano/master/examples/bobs-books/bobs-books-comp.yaml >~/bobs-books-comp.yaml
    cd
    </copy>
    ```

    ![Oracle SSO](images/7.png)

2. We will keep all Kubernetes artifacts in the separate namespace. Create a namespace for the Bob's Books example application. Namespaces are a way to organize clusters into virtual sub-clusters. We can have any number of namespaces within a cluster, each logically separated from others but with the ability to communicate with each other.
Also we need to make Verrazzano aware that we store in that namespace Verrazzano artifacts. So we need to add a a label identifying the bobs-books namespace as managed by Verrazzano. Labels are intended to be used to specify identifying attributes of objects that are meaningful and relevant to users. Here, for the bobs-book namespace, we are attaching a label to it, which marks this namespace as managed by Verrazzano. The *istio-injection=enabled*, enables an Istio "sidecar", and as such, helps establish an Istio proxy. With an Istio proxy, we can access other Istio services like an Istio gateway and such. To add the label to the bobs-books namespace with the previously mentioned attributes, copy the following command and run it in the *Cloud Shell*

    ```bash
    <copy>
    kubectl create namespace bobs-books
    kubectl label namespace bobs-books verrazzano-managed=true istio-injection=enabled
    </copy>
    ```

    ![Verrazzano Home Folder](images/9.png)

3. Copy the following command to download the script. This script authenticate the user for Oracle Container Registry. If authentication is successful, then it creates the docker registry secret. The Docker registry  is a way to store and version images, like GitHub for normal code but for containers (which Kubernetes can pull). Here, we will create a docker-registry secret to enable pulling the Bob's Books example image from the Oracle Container Registry. Click *Copy* on the following command, and paste it in any text editor of your choice and replace username and password with the email ID and password respectively which you used in step 1, for accepting the license agreement for downloading images from the Oracle Container Registry. Then, in the Cloud Shell, paste the modified command as shown:

    ```bash
    <copy>
    curl -LSs https://raw.githubusercontent.com/oracle/learning-library/master/developer-library/multicloud/verrazzano/deploy-bobsbook/create_secret.sh >~/create_secret.sh
    chmod 777 create_secret.sh
    ./create_secret.sh username password    
    </copy>
    ```

    ![Oracle Account](images/11.png)

4. We need to create several Kubernetes secrets with credentials.  In the Bob's Books application, we have two WebLogic domains *bobby-front-end* and *bobs-bookstore*. The credentials for the WebLogic domain are kept in a Kubernetes Secret where the name of the secret is specified using *webLogicCredentialsSecret* in the WebLogic Domain resource. Also, the domain credentials secret must be created in the namespace where the domain will be running. We need to create the secrets called *bobbys-front-end-weblogic-credentials* and *bobs-bookstore-weblogic-credentials* used by WebLogic Server domains, with a user name value of `weblogic` and a password which is randomly generated in the bobs-books namespace. Our Bob's Books application uses a *mysql* database. So, we create a new secret called  *mysql-credentials*, with a user name value of `weblogic`, a password which is randomly generated,  and  JDBC URL, *jdbc:mysql://mysql.bobs-books.svc.cluster.local:3306/books* in the bobs-books namespace. We will use this values in the JDBC connection string in WebLogic DataSource object.
Please copy and paste the block of commands into the *Cloud Shell*.

    ```bash
    <copy>
    export WLS_USERNAME=weblogic
    export WLS_PASSWORD=$((< /dev/urandom tr -dc 'A-Za-z0-9!"#$%&'\''()*+,-./:;<=>?@[\]^_`{|}~' | head -c10);(date +%S))
    echo $WLS_PASSWORD
    kubectl create secret generic bobbys-front-end-weblogic-credentials --from-literal=password=$WLS_PASSWORD --from-literal=username=$WLS_USERNAME -n bobs-books
    kubectl create secret generic bobs-bookstore-weblogic-credentials --from-literal=password=$WLS_PASSWORD --from-literal=username=$WLS_USERNAME -n bobs-books
    kubectl create secret generic mysql-credentials \
        --from-literal=username=$WLS_USERNAME \
        --from-literal=password=$WLS_PASSWORD \
        --from-literal=url=jdbc:mysql://mysql.bobs-books.svc.cluster.local:3306/books \
        -n bobs-books
    cd    
    </copy>
    ```

    ![mysql](images/12.png)

5. We have a Kuberneter cluster, *cluster1*, with three nodes. Now, we want to deploy Bob's Books containerized application on *cluster1*. For this, we need a Kubernetes deployment configuration. This deployment instructs the Kubernetes to create and update instances for the Bob's Books application. Here, we have the `bobs-books-comp.yaml` file, which instructs Kubernetes. To deploy the Bob's Books application, copy and paste the following two commands as shown. The `bobs-books-comp.yaml` file contains definitions of various OAM components, where, an OAM component is a Kubernetes Custom Resource describing an application’s general composition and environment requirements. To learn more about the `bobs-books-comp.yaml` file, review Verrazzano Components in the Introduction section of this Lab 3.

    ```bash
    <copy>kubectl apply -f ~/bobs-books-comp.yaml</copy>
    ```

    ![app](images/20.png)

6. The `bobs-books-app.yaml` file is a Verrazzano application configuration file, which provides environment specific customizations. To learn more about `bobs-books-app.yaml` file, review Verrazzano Application Configuration in the Introduction section of this Lab 3.

    ```bash
    <copy>kubectl apply -f ~/bobs-books-app.yaml</copy>
    ```

    ![app](images/21.png)

7. Wait for all of the pods in the Bob’s Books example application to be in the *Running* state. You may need to repeat this command several times before it is successful. The WebLogic Server and Coherence pods may take a while to be created and Ready. This *kubectl* command will wait for all the pods to be in the *Running* state within the bobs-books namespace. It takes around 4-5 minutes.

    ```bash
    <copy>kubectl wait --for=condition=Ready pods --all -n bobs-books --timeout=600s</copy>
    ```

    ![Pods to be ready](images/22.png)

8. Get the `EXTERNAL_IP` address of the istio-ingressgateway service. Copy this `EXTERNAL_IP` in your text editor; we will use it in many places, so you can directly copy it from your text editor.

    ```bash
    <copy>kubectl get service \
    -n "istio-system" "istio-ingressgateway" \
    -o jsonpath={.status.loadBalancer.ingress[0].ip}; echo</copy>
    ```

    ![External IP](images/23.png)

## Task 3: Verify the successful deployment of the Bob's Book application

Verify that the application configuration, domains, Coherence resources, and ingress trait all exist.

1. To verify that the *Bob's Books* application is successfully deployed in the bobs-books namespace.

    ```bash
    <copy>kubectl get ApplicationConfiguration -n bobs-books</copy>
    ```

    ![Application Configuration](images/24.png)

2. To verify that both WebLogic domains are created within the bobs-books namespace successfully.

    ```bash
    <copy>kubectl get Domain -n bobs-books</copy>
    ```

    ![WebLogic Domain](images/25.png)

3. To verify that both Coherence clusters are created within the bobs-books namespace successfully.

    ```bash
    <copy>kubectl get Coherence -n bobs-books</copy>
    ```

    ![Coherence](images/29.png)

4. To get the IngressTrait for the Bob's Book application, run the following command in the *Cloud Shell*.

    ```bash
    <copy>kubectl get IngressTrait -n bobs-books</copy>
    ```

    ![Ingress](images/26.png)

5. Verify that the service pods are successfully created and transition to the *Running* state. Note that this may take a few minutes and that you may see some of the services terminate and restart. Finally, you will observe all the pods associated with the bobs-books namespace are in the *Running* Status. Please copy the pods details for the *bobbys-helidon-stock-application*.

    ```bash
    <copy>kubectl get pods -n bobs-books</copy>
    ```

    ```bash
    YOURUSERNAME@cloudshell:~ (us-ashburn-1)$ kubectl get pods -n bobs-books
    NAME                                                READY   STATUS    RESTARTS   AGE
    bobbys-coherence-0                                  2/2     Running   0          11m
    bobbys-front-end-adminserver                        4/4     Running   0          8m15s
    bobbys-front-end-managed-server1                    4/4     Running   0          7m32s
    bobbys-helidon-stock-application-77867fc8dd-wl8h5   2/2     Running   0          11m
    bobs-bookstore-adminserver                          4/4     Running   0          7m59s
    bobs-bookstore-managed-server1                      4/4     Running   0          7m14s
    mysql-65d864bf8c-xf64p                              2/2     Running   0          10m
    robert-helidon-bfdfb58b8-58qfs                      2/2     Running   0          11m
    robert-helidon-bfdfb58b8-lkw8m                      2/2     Running   0          11m
    roberts-coherence-0                                 2/2     Running   0          11m
    roberts-coherence-1                                 2/2     Running   0          11m
    YOURUSERNAME@cloudshell:~ (us-ashburn-1)$
    ```

    Note the pod name for **bobbys-helidon-stock-application**. When we redeploy this component, you will notice that this pod will go into a *Terminating* status and new pod will start and come in the *Running* state in Lab 7.

Leave the *Cloud Shell* open; we will use it for the next labs as well.

## Acknowledgements

* **Author** -  Ankit Pandey
* **Contributors** - Maciej Gruszka, Peter Nagy
* **Last Updated By/Date** - Kamryn Vinson, July 2021
