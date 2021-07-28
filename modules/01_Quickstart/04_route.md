# Expose a Service with a Route

In this section we will create a OpenShift Route to expose the service we deployed.

## Exercise: Update Deployment with Route

In the last section we created our deployment, now we will update the deployment and expose a URL (OpenShift calls this
a Route) that we will be able to access from our browser. Our Helm chart can be easily updated and redeployed with the
same settings we used previously.

In the Developer Perspective of the OpenShift web interface, navigate to **Helm** on the left hand navigation bar and
our **hello-world** release should be listed. Click the three vertical dots on the left and select **Upgrade** which
will bring back our install form with our previous values already filled out.

Add these values to the form and click **Upgrade** at the bottom:

- Ingress
  - Enabled: **true**
  - External route: **true**

Once the upgrade is finished we should be taken back to the Helm view. Navigate to the **Topology** view and you should
see our application and be able to click on the circle or the label with the **D** then a sidebar will expand to show you
the details of the **Deployment** Kubernetes object which has been created.

Once the Pods of the Deployment are running you can click the Route URL that was created which will prompt you to log in with
your NCCS credentials and then show you a "Hello OpenShift" page which is what our application is serving.

[Next](05_build.md)
