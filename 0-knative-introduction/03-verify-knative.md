# Verify Knative Deployment on OpenShift

At this point you should have a working Knative deployment on your OpenShift cluster.  In this Step,
you will deploy the hello-go sample function that is already built into a container image and verify that
it works.

**1. Start Service for hello-go Function**

It is good to pre-pull the hello-go sample image:
``docker pull gcr.io/knative-samples/helloworld-go``{{execute}}

Next, we need to create a service yaml file to deploy the helloworld-go function when a request come in on the ingress gateway.
Click on the link below to create an empty file called **helloworld-service.yaml** in the directory **/root/projects/knative** :
``helloworld-service.yaml``{{open}}

Once the created file is opened in the editor, you can then copy the content below into the file (or use the `Copy to editor` button):

<pre class="file" data-filename="/root/projects/knative/helloworld-service.yaml" data-target="replace">
apiVersion: serving.knative.dev/v1alpha1 # Current version of Knative
kind: Service
metadata:
  name: helloworld-go # The name of the app
  namespace: default # The namespace the app will use
spec:
  runLatest:
    configuration:
      revisionTemplate:
        spec:
          container:
            image: gcr.io/knative-samples/helloworld-go # The URL to the image of the app
            env:
            - name: TARGET # The environment variable printed out by the sample app
              value: "Go Sample v1"
</pre>

Now, we can apply the service yaml:

``oc apply --filename /root/projects/knative/helloworld-service.yaml``{{execute}}

**2. Verify Function**

``export IP_ADDRESS=$(oc get svc knative-ingressgateway --namespace istio-system --output'jsonpath={.status.loadBalancer.ingress[0].ip}')``{{execute}}

``export HOST_URL=$(oc get ksvc helloworld-go  --output jsonpath='{.status.domain}')``{{execute}}

``curl -H "Host: ${HOST_URL}" http://${IP_ADDRESS}``{{execute}}

