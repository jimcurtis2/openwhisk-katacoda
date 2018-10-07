# Setup Knative on OpenShift

Now, that we understand the design of Knative, we will proceed to deploy Knative onto our OpenShift cluster.

**1. Login as Admin**

``oc login -u system:admin``{{execute}}
``oc adm policy add-scc-to-user privileged -z default -n default``{{execute}}

**2. Fix Admission Webhook Config**

```
sed '/pluginConfig:/q' /etc/origin/master/master-config.yaml >>myjunk
cat << 'EOL' >> myjunk
     MutatingAdmissionWebhook:
       configuration:
         apiVersion: v1
         disable: false
         kind: DefaultAdmissionConfig
     ValidatingAdmissionWebhook:
       configuration:
         apiVersion: v1
         disable: false
         kind: DefaultAdmissionConfig
EOL
grep -A 9999 'BuildDefaults:' /etc/origin/master/master-config.yaml >>myjunk
mv myjunk /etc/origin/master/master-config.yaml
docker ps -l -q --filter "label=io.kubernetes.container.name=api"
docker stop $(docker ps -l -q --filter "label=io.kubernetes.container.name=api")
docker ps -l -q --filter "label=io.kubernetes.container.name=api"
```{{execute}}

**3. Deploy Istio**

```
oc label namespace default istio-injection=enabled
oc adm policy add-scc-to-user anyuid -z istio-ingress-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z default -n istio-system
oc adm policy add-scc-to-user anyuid -z prometheus -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-egressgateway-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-citadel-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-ingressgateway-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-cleanup-old-ca-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-mixer-post-install-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-mixer-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-pilot-service-account -n istio-system
oc adm policy add-scc-to-user anyuid -z istio-sidecar-injector-service-account -n istio-system
oc adm policy add-cluster-role-to-user cluster-admin -z istio-galley-service-account -n istio-system
curl -L https://storage.googleapis.com/knative-releases/serving/latest/istio.yaml | oc apply -f -
```{{execute}}

**4. Wait for Istio to Achieve Stable State**

``while $(oc get pods -n faas controller-0 | grep 0/1 > /dev/null); do sleep 1; done``{{execute}}


Successful execution of the above command should display output like below:

![OpenWhisk Default Catalog](../assets/ow_catalog_actions.png)

## Congratulations

You have now deployed [Apache OpenWhisk](https://openwhisk.apache.org/) to the [OpenShift Container Platform](https://openshift.com]). 
