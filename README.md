# deploy-otel-demo-alloy
This project helps you accelerate the process of standing up the [OpenTelemetry Demo](https://opentelemetry.io/docs/demo/architecture/) and Grafana Alloy, with Alloy shipping your OTel telemetry to your Grafana Cloud account.

This README page gives you the quick setup steps; the links further down give you more detail.

<br>
<br>

# Quick setup steps

These steps are very brief by design. They will assume that you have solid background in a number of areas. Further areas of the documentation will help users learn each step in more detail.

Note that these are written for the Linux OS; we do not yet document how to run this on other operating systems.

<br>

### Prerequisites
K8s cluster:   I recommend at least the equivalent of [4 e2-medium machines](https://cloud.google.com/compute/docs/general-purpose-machines#e2-shared-core). 

CLIs:          Git, Kubectl, Flux

Viewer:        K9s, or whatever solution you like

Code:          VSCode, or whatever solution you like

<br>

### Fork, then clone, this repo
Cloning without forking won't work. Make your own fork, then clone your fork to your machine. Then cd into the top level directory.



<br>

### Create a file called exports.env, with this content:
```
export GITHUB_USER=
export GITHUB_TOKEN=
export ALLOY_CLOUD_OTLP_URL=
export ALLOY_CLOUD_OTLP_USERNAME=
export ALLOY_CLOUD_OTLP_PASSSWORD_BASE64=
```
> (Note that the .gitignore in this project should keep this file from being checked in, if you name it with a .env extension)

<br>

### Fill in the values in exports.env
First 2: Fill in your GH username, and go get a "classic personal access token" from GH with all Repo permissions enabled; no other permissions needed.

Last 3:  From your Grafana Cloud Portal, go into a stack, then into the OpenTelemetry section, and get your username, url, and auth token.

<br>

### IMPORTANT: base64 encode your token
Before putting the Grafana Cloud OTel auth token into its place in the above exports, base64 encode it.
```
echo -n YOUR_TOKEN_HERE | base64
```
The output of that command is what you place into exports.env.


<br>

### Run the 5 exports
Your choice:
- make exports.env executable and run it;
- copy/paste the export statements from exports.env to the command line;
- make them part of your environment setup in another way of your choosing.

<br>

### Now run the project
You should now be good to run the following 2 commands, from the root directory of the project.

```
./pre-provision/alloy.sh
```

```
CURRENT=`pwd`; BASENAME=`basename "$CURRENT"`; \
flux bootstrap github \
    --context=$(kubectl config current-context) \
    --owner=${GITHUB_USER} \
    --repository=${BASENAME} \
    --branch=main \
    --path=clusters/staging
```

<br>

## Sanity Check
Use k9s or the viewer of your choice, to see that Alloy comes up in the "collector" namespace, and the OTel Demo comes up in the "open-telemetry" namespace.

Go to your cloud account, and use the Explore Logs UI to see that logs are arriving from a number of the services in the OTel Demo.


<br>

## Teardown:
This set of commands will uninstall Flux from the cluster, and then delete the namespaces created by this project.

```flux uninstall -s ; kubectl delete namespace collector ; kubectl delete namespaces open-telemetry```

<br>

## To use the K8s integration instead of Alloy:

1) see [this file](./pre-provision/k8s-integration.sh) - note that the OTel endpoint username (i.e. stack ID) has to be exported base64 encoded.
2) do that encoding and export step
3) run the k8s-integration pre-provision script
4) run the flux bootstrap command again, but this time with the last line as:
```
    --path=clusters/production
```

When Flux starts deploying things, you should see the K8s integration come up. The OTel services will also be configured correctly to talk to the "receiver" Alloy's service.




<br>
<br>

# Getting Started

Please see [how-to-run page](./zzz_documentation/how-to-run.md) for more detailed instructions on running this project.

<br>

# Further Learning

Please see the [overview page](./zzz_documentation/overview.md) for more detail on the overall project.
