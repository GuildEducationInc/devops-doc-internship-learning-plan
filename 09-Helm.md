# Part 9: Helm

Helm is a “package manager” for Kuberentes. It allows a simple, single command, to install and configure various software in a cluster. It also keeps track of ‘releases’ and allows you to roll back easily to any release. This curriculum will walk you through configuration and use of Helm in a Kubernetes cluster and touch on chart creation.

**PreRequisites**:

- General understanding of command line use including `kubectl`.
- Specific understanding of Kubernetes primitives and yaml
- General understanding of templateing
- Specific knowledge of Golang Templating is helpful but not required

**Definitions**:

- Chart: A set of templates configured to deploy an application into Kubernetes.
- Release: The instance of the created application. Each `helm upgrade` or `helm install` command creates a new release
- Values: The values used to configure the templated yaml. Most charts have sensible defaults and a few required values. Values can be passed in on the command line or through a yaml file(s)
- Helm: The local command

**Preparation:**

1. Setup Kubernetes using Kind or Minikube:
    1. https://github.com/kubernetes-sigs/kind
    2. https://kubernetes.io/docs/tasks/tools/install-minikube/
2. Install Helm on your workstation
    1. https://helm.sh/docs/using_helm/#installing-helm

  - Talk about why you chose one tool over the other

**Working with Helm:**

1. Create a redis helm release: 
    ```bash
    # Add bitnami helm repo
    helm repo add bitnami https://charts.bitnami.com/bitnami
    # Install bitnami redis chart and name the release 'redis'
    helm install bitnami/redis --generate-name
    ```
    This will install the basic Redis chart from the stable repository with a random name
2. List releases: `helm list -A`
    1. What name does your release have?
    2. What is the `STATUS`
3. Delete release: `helm delete <release-name>`
4. Create a new named release: `helm install redis bitnami/redis`
    1. What namespace did it install into?
    2. How would you install it into a different namespace?
    3. List the pods
5. What if you want to configure the chart outside other than the defaults?
    1. Use `helm inspect stable/redis` to see the templates and parameters and configuration information.  Sometimes that is overwhelming so commonly, you can use the [documentation](https://github.com/helm/charts/tree/master/stable/redis).  
    2. Lets install the chart again, but with `rbac.create` turned on. This will configure the chart to create rbac roles and bindings for the redis pods.
        ```bash
        # Notice the --set command which sets the values from the command line
        helm install redis bitnami/redis --set rbac.create=true
        # Lets also enable the metrics endpoint
        helm upgrade redis bitnami/redis --set metrics.enabled=true --reuse-values
        ```
        * Why did the upgrade fail? See if you can fix it.

        * Notice `upgrade` instead of `install`. Upgrade modifies an existing release, install creates a new one. You can do both in one command with `helm upgrade -i`
        * run `helm upgrade --help` to see more options. Do you think you’ll need to use `--recreate-pods` for the Pods to get the new settings?
        * You can also use a yaml file to pass values into the `helm` command with the `--values` flag.
7. View the redis chart on github and review `values.yml`, these are the default values for this chart. These values can be set from command line with `--set` or from the `--values` file. Reivew `templates/master/statefulset.yaml`. This file uses the Golang template language. The exact syntax is not important but you’ll notice that template blocks are wrapped in `{{ }}`. Those that start with `include` are found in `templates/helpers.tpl`, review this file as well.
8. Create your own chart: `helm create new-chart`
    1. Open the chart and see what `templates` are created for you and what values are created for you.
        1. What app will this chart deploy by default?
        2. Will it deploy successfully as is? Try. `helm upgrade -i new-chart new-chart/` Notice that because the chart is not coming from a repository like `stable/redis` you just provide the path to the chart.
    2. Pick a simple app you’d like to install and convert this chart to deploy it.
        1. Make sure to use the `templates`  in `new-chart/templates/helpers.tpl` for consistency
        2. What happens if you need to add a new file to `new-chart/templates` ? Will helm automatically deploy it? Try.
        3. How is `new-chart/templates/Notes.txt` used? Make sure to update it when you convert this chart to your chose app.
    3. View `new-chart/Chart.yml` to view the metadata about your `new-chart`
    4. Use a `--values` file to `upgrade` the release of this chart.
