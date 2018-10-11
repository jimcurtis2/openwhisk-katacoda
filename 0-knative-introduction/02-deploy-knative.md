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
mv -f myjunk /etc/origin/master/master-config.yaml
```{{execute}}

Stop the docker container so it restarts with the new configuration.

``docker stop $(docker ps -l -q --filter "label=io.kubernetes.container.name=api")``{{execute}}

Verify that the cluster is still up and ready to go before continuing

``sleep 2;oc get nodes``{{execute}}

Now go on to next step.

**3. Fix and Deploy Istio**

The istio.yaml file that is recommended for Knative does not have the privileged security context needed for OpenShift.
So, we will pull down a copy of the yaml and edit it before applying it.

``curl -L https://storage.googleapis.com/knative-releases/serving/latest/istio.yaml > istio.yaml``{{execute}}

``awk '{print} /securityContext:/ && !n {print "          privileged: true"; n++}' istio.yaml > istio-fixed.yaml``{{execute}}

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
```{{execute}}

``oc apply -f istio-fixed.yaml``{{execute}}

**4. Wait for Istio to Achieve Stable State**

``while $(oc get pods -n istio-system | grep 0/1 > /dev/null); do sleep 1; done``{{execute}}


Successful execution of the above command should display output like below:

![OpenWhisk Default Catalog](../assets/ow_catalog_actions.png)

## Congratulations

You have now deployed [Apache OpenWhisk](https://openwhisk.apache.org/) to the [OpenShift Container Platform](https://openshift.com]). 
