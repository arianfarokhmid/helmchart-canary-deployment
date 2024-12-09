# In this scenario, we are deploying a mock application using Helm in the prod-ns namespace.

# Let's explore what we have deployed

## first of you have to clone the repository:
```bash
```
## Verify the deployed release
```bash
helm list -n prod-ns
```
We have a single release named mock-app deployed in the target namespace.

## Verify the deployed pods replicas
```bash
kubectl get pods -n prod-ns
```

The application is composed of 4 pod replicas. All of the them from the same replicaset.
## Verify the content of the deployed application
We will dispatch multiple requests to the application to confirm that we have reached all four replicas as intended.
```bash
export PORT=5000
export SERVICE_IP=$(kubectl get svc -n prod-ns -l app=mock-app -o jsonpath='{.items[0].spec.clusterIP}')
for ((i=1; i<=20; i++)); do
    # Execute curl command and print newline
    curl -s "http://${SERVICE_IP}:${PORT}" -w "\n"
done
```

as you can see, our application is returning: Hello Killercoda Folks! You recieved this message: We are exploring canary deployment. from all replicas.

## Verify values used by our release

```bash
helm get values -n prod-ns mock-app --all
```

 The most important fields that we are targeting:

    canaray.enabled: a boolean value to check if we are enabling or disabling canary deployment.
    image.tag: the image tag used by the stable prod version. Current application version is v1.0.0
    image.canaryTag: the image tag used by the canary prod version. the canaryTag is not defined as canary is disabled

### step 2
A new version, v1.1.0, of the application is ready with exciting new features. This version has been tested in the review environment, and now we want to test it in the production environment without affecting the entire current stable version.

The solution for that: Canary Deployment
## Upgrade the application release: Enable canary deployment

We need to enable the canary deployment and set the new software version (image version) as the tag for our canary image. 
```bash
helm upgrade --install --namespace prod-ns mock-app mock-app-repo/mock-app-canary --set canary.enabled=true --set image.canaryTag=v1.1.0
```

helm upgrade --install --namespace prod-ns mock-app mock-app-repo/mock-app-canary --set canary.enabled=true --set image.canaryTag=v1.1.0
```bash
kubectl get pods -n prod-ns
```

The application is composed of 4 pods:

    3/4 (75%) of the pods are from the stable application version v1.0.0
    1/4 (25%) of the pods are from the canary new application version v1.1.0

 As you can see, our application is now serving approximately 25% of its content from the new version, with the remaining content coming from the stable version.

Canary deployment allows us to gradually release and test a new version of an application in production with minimal risk, by exposing it to a small subset of users before full rollout.

The new version has been validated. In the next step, we will apply the new application version to all replicas.

Once we have thoroughly tested and validated our new version, currently deployed to 25% of our production environment, we are ready to roll it out across the entire infrastructure.

## Upgrade the application release

We need to disable the canary deployment and set the new software version (image version) as the tag for our stable image.

```bash
helm upgrade --install --namespace prod-ns mock-app mock-app-repo/mock-app-canary --set canary.enabled=false --set image.tag=v1.1.0
```

 as you can see, our application is now returning the new version contrnet from all replicas. So we upgraded the application successfully.

For further information regarding the chart templates, you can locate them at /mock-app-canary.



