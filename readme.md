# Local Gloo Steps
Full details can be found here: https://docs.solo.io/gloo/latest/installation/gateway/kubernetes
## Install stuff

1) `brew install minikube`
2) `brew install solo-io/tap/glooctl`
    - verify installation with `glooctl version`


## Set up Gloo

1) Start up kubernetes cluster: `minikube start --memory=4096 --cpus=2`
2) `kubectl config current-context` 
       - assert returns `minikube` if not, run `kubectl config use-context minikube`
3) Install gloo
    - Standard Gloo: `glooctl install gateway`
    - enterprise Gloo: `glooctl install gateway enterprise --license-key $GLOO_ENTERPRISE_LICENSE_KEY`
        - --version flag will specify the server version you want to install, defaults to latest
        - --values flag will load startup configuration from a yaml file
4) In a new terminal window, and run `minikube tunnel`.  Enter sudo password and leave the terminal running as it doesn't disconnect.
    - This will assign an IP to the load-balancer, and is necessary for local development (cloud env differs)
5) `glooctly proxy url` will tell you what your gateway address is.

## Configure resources
You can apply resources using kubectl.  Assuming you have a virtual service configuration named vs.yaml, and an upstream configuration named upstream.yaml in your current working directory:
 1) `kubectl apply -f upstream.yaml` - creates or updates the upstream in that configuration
 2) `kubectl apply -f vs.yaml` - creates or updates the virtual service in that configuration.
 
You can also generate these configurations using the `glooctl` cli - see https://docs.solo.io/gloo/latest/reference/cli/

 You can view upstreams and virtual services by running:
 `glooctl get upstreams` and `glooctl get virtualservices`
 
 ##Cleaning up
 1) Run `minikube delete` to remove everything and shutdown the minikube environment. 
 
 _Delete specific resources using `glooctl delete ` and `kubectl delete`_
 
 _Uninstall gloo gateway: `glooctl uninstall`_
 
 ##Debugging
 Full details see: https://docs.solo.io/gloo/latest/operations/debugging_gloo/
 
 ####Useful Commands:
 - `glooctl check` checks for problems.  I see problems with enterprise deployments locally but still seems to be OK.
 - `kubectl get all -n gloo-system` shows all resources in `gloo-system` namespace and their status
 - `glooctl debug logs` will show you gloo log output.
 - `glooctl debug logs --errors-only` self explanatory
 - `glooctl proxy logs -f` will *follow* your connection/proxy logs.
 - `kubectl logs -n <namespace> <type/name>` pulls the log for the resource specified
    - example: `kubectl logs -n gloo-system deployment/extauth` pulls the logs for the extauth server deployment
    
 ## Run gloo with plugin example:
 1) `minikube start --memory=4096 --cpus=2`
 2) `glooctl install gateway enterprise --license-key $GLOO_ENTERPRISE_LICENSE_KEY --values plugin-values.yaml --version 1.3.8`
 3) `kubectl apply -f global-settings.yaml`
 4) `kubectl apply -f auth.yaml`
 5) `kubectl apply -f upstream.yaml`
 6) `kubectl apply -f vs.yaml`
 7) Make call to proxy url `glooctl proxy url`