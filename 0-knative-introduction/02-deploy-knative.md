# Setup Knative on OpenShift

Now, that we understand the design of Knative, we will proceed to deploy Knative onto our OpenShift cluster.

**1. Login as Admin**

``oc login -u system:admin``{{execute}}

``oc adm policy add-scc-to-user privileged -z default -n default``{{execute}}

**2. Fix Admission Webhook Config**

The standard OpenShift 3.10 does not have the Admission Webhook plugin configuration needed by Knative.  So
we need to edit the configuration file and restart the container.

```
sed '/pluginConfig:/q' /etc/origin/master/master-config.yaml >>master-config.yaml
cat << 'EOL' >> master-config.yaml
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
grep -A 9999 'BuildDefaults:' /etc/origin/master/master-config.yaml >>master-config.yaml
mv -f master-config.yaml /etc/origin/master/master-config.yaml
```{{execute}}

Stop the docker container so it restarts with the new configuration.

``docker stop $(docker ps -l -q --filter "label=io.kubernetes.container.name=api")``{{execute}}

Verify that the cluster is still up and ready to go before continuing.  It usually takes about 10 seconds for things to
stablize before you can successfully execute an oc command, so we will do a sleep before the command.

``sleep 10;oc get nodes``{{execute}}

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

Now, wait for Istio to achieve stable state.

``while $(oc get pods -n istio-system | grep istio-sidecar-injector | grep 0/1 > /dev/null); do sleep 1; done``{{execute}}

**4. Deploy Knative Serving**

First, run the following to grant the necessary privileges to the service accounts Knative Serving will use:

```
oc adm policy add-scc-to-user anyuid -z build-controller -n knative-build
oc adm policy add-scc-to-user anyuid -z controller -n knative-serving
oc adm policy add-scc-to-user anyuid -z autoscaler -n knative-serving
oc adm policy add-scc-to-user anyuid -z kube-state-metrics -n monitoring
oc adm policy add-scc-to-user anyuid -z node-exporter -n monitoring
oc adm policy add-scc-to-user anyuid -z prometheus-system -n monitoring
oc adm policy add-cluster-role-to-user cluster-admin -z build-controller -n knative-build
oc adm policy add-cluster-role-to-user cluster-admin -z controller -n knative-serving
```{{execute}}

Next, apply the Knative Serving config:

``curl -L https://storage.googleapis.com/knative-releases/serving/latest/release-lite.yaml | oc apply -f -``{{execute}}

Now, wait for Knative Serving to achieve stable state.

``while $(oc get pods -n knative-serving | grep 0/1 > /dev/null); do sleep 1; done``{{execute}}


## Congratulations

You have now deployed [Apache OpenWhisk](https://openwhisk.apache.org/) to the [OpenShift Container Platform](https://openshift.com]). 
