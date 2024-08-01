[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
# helm-workshop

* references
    * [Helm Masterclass](https://www.udemy.com/course/helm-masterclass)
    * https://jfrog.com/blog/is-your-helm-2-secure-and-scalable/
    * https://helm.sh/docs/
    * https://insights.project-a.com/whats-the-best-way-to-manage-helm-charts-1cbf2614ec40
    * https://blog.risingstack.com/packing-a-kubernetes-microservices-with-helm/
    * https://github.com/addamstj/helm-course
    * [What is Helm in Kubernetes? Helm and Helm Charts explained | Kubernetes Tutorial 23](https://www.youtube.com/watch?v=-ykwb1d0DXU)
    * [Introduction to Helm | Kubernetes Tutorial | Beginners Guide](https://www.youtube.com/watch?v=5_J7RWLLVeQ)
    * https://helm.sh/docs/howto/charts_tips_and_tricks/
    * [An Introduction to Helm - Matt Farina, Samsung SDS & Josh Dolitsky, Blood Orange](https://www.youtube.com/watch?v=Zzwq9FmZdsU)
    * [Delve into Helm: Advanced DevOps [I] - Lachlan Evenson & Adam Reese, Deis](https://www.youtube.com/watch?v=cZ1S2Gp47ng)
    * https://helm.sh/docs/
    * https://jacky-jiang.medium.com/use-named-templates-like-functions-in-helm-charts-641fbcec38da
    * https://stackoverflow.com/questions/62472224/what-is-and-what-use-cases-have-the-dot-in-helm-charts

## preface
* goals of this workshop
    * understanding the purpose of helm
    * understanding how it facilitates k8s deployments
    * introducing structure of helm
        * charts
        * templates
        * values
    * familiarize with basic commands for releasing app

## introduction
* suppose we are deploying elastic stack for logging
    * needs: stateful set, config map, k8s user with permissions, secret, services
        * and you want to rollback them as a whole
    * maybe someone could create the yaml files once, package them and make available somewhere
        * that package is known as helm charts
    * public repositories: https://artifacthub.io/
    * private repositories: share in organisation
* suppose we have many microservices in the cluster and across different environments (dev, stage, prod)
    * with very similar deployments (ex. apart from name and image)
    * a lot of copy-paste and duplication
    * maybe someone could create a template and make available somewhere
        * then we fill that template with values
        * that template engine is know as helm

## helm
* package manager - software tool that automates the process of installing, upgrading, configuring, and removing
software in a consistent manner
* is a package manager for k8s
    * package yaml files and distribute them in public / private repositories
    * manage your k8s deployments
    * teardown and create deployments with one command
* is a templating engine
* tasks
    * create new charts from scratch
    * package charts into chart archive (tgz) files
    * interact with chart repositories where charts are stored
    * install and uninstall charts into an existing Kubernetes cluster
    * manage the release cycle of charts that have been installed with Helm
* components
    * Helm Client
        * a command-line client for end user
        * responsible for:
            * local chart development
            * managing repositories
            * managing releases
            * interfacing with the Helm library
            * sending charts to be installed
            * requesting upgrading or uninstalling of existing releases
    * Helm Library
        * provides the logic for executing all Helm operations
        * interfaces with the Kubernetes API server
        * provides the following capability:
            * combining a chart and configuration to build a release
            * installing charts into Kubernetes, and providing the subsequent release object
            * upgrading and uninstalling charts by interacting with Kubernetes
* helm2 vs helm3
    * most apparent change is the removal of Tiller
        * Tiller was the server-side component which is used to maintain the state of helm release
        * Tiller = in-cluster server that interacts with the Helm client, and interfaces with the Kubernetes API server

## project structure
* `Chart.yaml`
    * meta info about chart
    * description of the package (most important fields below)
        * apiVersion: chart API version (required)
            * should be v2 for Helm charts that require at least Helm 3
        * appVersion: version of the app that this chart contains (optional)
            * informational, has no impact on chart version calculations
        * version: A SemVer 2 version (required)
        * kubeVersion: range of compatible Kubernetes versions (optional)
        * name: The name of the chart (required)
        * description: A single-sentence description of this project (optional)
        * type: type of the chart (optional)
            * application (default) and library
            * library chart defines chart primitives or definitions which can be shared in other charts
        * dependencies: list of the chart requirements (optional)
            * one chart may depend on any number of other charts
            * will download all the specified charts into your `charts/` directory for you
                * example
                    ```
                    dependencies:
                      - name: apache
                        version: 1.2.3
                        repository: https://example.com/charts
                      - name: mysql
                        version: 3.2.1
                        repository: https://another.example.com/charts
                    ```
                    will be downloaded into `charts`
                    ```
                    charts/
                      apache-1.2.3.tgz
                      mysql-3.2.1.tgz
                    ```
            * if more control required: dependencies can be explicitly copied into the `charts/` directory
* `values.yaml`
    * contains default values for the template files
    * could be overridden
        * `helm install --values=my-values.yaml <chartname>`
        * example: `dev.yaml`, `prod.yaml`, `stage.yaml`
* `charts` -> chart dependencies
* `templates`
    * templates that, when combined with values, will generate valid Kubernetes manifest files
    * most files are treated as if they contain Kubernetes manifests
    * files whose name begins with an underscore `_` are assumed to not have a manifest inside
        * used to store partials and helpers
        * example: `_helpers.tpl`

## helm charts
* is
    * unit of deployment
    * set of yaml files
    * a collection of files that describe a related set of Kubernetes resources
* hooks
    * to intervene at certain points in a release's life cycle
    * purpose
        * load a ConfigMap or Secret during install before any other charts are loaded
        * run a Job before deleting a release to gracefully take a service out of rotation before removing it
    * example
        * pre-install: executes after templates are rendered, but before any resources are created in Kubernetes
        * post-install: executes after all resources are loaded into Kubernetes
    * remark: if you create resources in a hook, you cannot rely upon `helm uninstall` to remove it
        * either add a `custom helm.sh/hook-delete-policy` annotation to the hook template file
        * or set the time to live (TTL) field of a Job resource
* tests
    * lives under the `templates/`
    * specifies a container with a given command to run
        * container should exit successfully (exit 0) for a test to be considered a success
    * purpose
        * validate that `values.yaml` was properly injected
        * username and password work correctly
        * incorrect username and password does not work
    * example
        ```
        kind: Pod
        metadata:
          annotations:
            "helm.sh/hook": test
        ...
        ```
* where do you put your Helm charts?
    1. chart repository to store one big shared chart
        * one big shared chart
            * can save a lot of hassle if services are similar
            * example: maintaining charts for 15 different microservices in a central chart repository
                * it is easier update them all in one place rather than submitting pull requests to 15
                different repositories
        * many service-specific charts
            * you can make a change to one service without worrying about breaking something for another service
            * if the charts are service-specific - no argument for storing them together
    1. same repository as the service itself
       * good choice for microservice-based applications where the services have significant differences
       * it’s easier to continuously deploy the service independently from other projects
       * it’s much easier to test if the chart and the code are in the same repo and can be tested in the same branch
* when Helm installs/upgrades charts
    1. Kubernetes objects from the charts and all its dependencies are aggregated into a single set
    1. then sorted by type followed by name
    1. then created/updated in that order
* when it comes to sharing charts, the preferred mechanism is a chart repository
    * chart museum ~ docker repository for charts

## template
* resides in `template/`
* k8s manifest = template + values
* recommended labels
    * app.kubernetes.io/name
        * app name
        * `{{ template "name" . }}`
    * helm.sh/chart
    	* chart name and version
    	* `{{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}`
    * app.kubernetes.io/managed-by
        * for finding all things managed by Helm
        * `{{ .Release.Service }}`
    * app.kubernetes.io/instance
        * aids in differentiating between different instances of the same application
        * `{{ .Release.Name }}`
    * app.kubernetes.io/version
    	* version of the app
    	* `{{ .Chart.AppVersion }}`
    * app.kubernetes.io/component
        * role of the pieces in an application
        * example: frontend, backed
* named template
    * used to share and reuse template snippets and deployment logic
    * example
        ```
        {{- define "mychart.labels" -}}
          labels:
            myLabel: {{ .Values.myLabel }}
        {{- end -}}
        ```
        then in template
        {{- include "mychart.labels" . }}
    * two templates with the same name - whichever one is loaded last will be the one used
        * naming convention: prefix each template with the name of the chart
    * passing scope
        * when a named template (created with define) is rendered, it will receive the scope passed in by the
        template call
        * example
            * `{{- template "mychart.labels" }}`
                * no scope was passed in, so within the template we cannot access anything in `.`
            * `{{- template "mychart.labels" . }}`
                * note that we pass `.` at the end of the template call
                * we could just as easily pass `.Values`, but we want a top-level scope
    * `include` vs `template`
        * `include` (preferable)
            * bring the template, and then pass the results to other template functions
            * example
                * `{{ include "mytpl" . | lower | quote }}`
                * includes a template called mytpl, then lowercases the result, then wraps that in double quotes
        * `template`
            * no option to pipe the results

## commands
* `helm create helm-chart-name`
    * create a new chart with the given name
* `helm install helm-chart-name .`
    * performs a release
        * a Release is an instance of a chart running in a Kubernetes cluster
    * deploy an app on your Kubernetes cluster
    * `helm install prodmyfirstchart . -f production.yaml`
* `helm template .`
    * render chart templates locally and display the output
    * any values that would normally be looked up or retrieved in-cluster will be faked locally
* `helm uninstall helm-chart-name`
    * removes all of the resources associated with the last release of the chart as well as the release history,
    freeing it up for future use
* `helm upgrade helm-chart-name`
    * upgrades a release to a new version of a chart
* `helm upgrade --install <release name> --values <values file> <chart directory>`
    * install or upgrade a release with one command
* `helm get manifest helm-chart-name`
    * fetches the generated manifest for a given release
* `helm list`
    * list releases
* `helm lint`
    * runs a series of tests to verify that the chart is well-formed
* `helm template .`
    * locally render templates
    * `helm template . > templated.yaml`

## functions
* required
    * declare a particular values entry as required for template rendering
    * example: `{{ required "A valid .Values.who entry required!" .Values.who }}`
* if statement
    ```
    {{- if eq .Values.proxy.enabled true -}}
    {{- include "proxy" . | nindent 8 -}}
    {{- end -}}
    ```
* set scope
    ```
    {{- with .Values.favorite }} // with statement sets . to point to .Values.favorite
    drink: {{ .drink | default "tea" | quote }}
    food: {{ .food | upper | quote }}
    {{- end }} // . is reset to its previous scope after {{ end }}
    ```
    vs
    ```
    drink: {{ .Values.favorite.drink | default "tea" | quote }}
    food: {{ .Values.favorite.food | upper | quote }}
    ```
* from UNIX: pipelines are a tool for chaining together a series of template commands
    * `{{ .Values.favorite.food | upper | quote }}`
    * pizza -> PIZZA -> "PIZZA"
* default
    * specify a default value inside of the template, in case the value is omitted
    * `{{ .Values.favorite.drink | default "tea" | quote }}`
    * remark: all static default values should live in the `values.yaml`
        * perfect for computed values, which can not be declared inside `values.yaml`

## best practices
* chart names: lower case, numbers, words may be separated with dashes (-)
* template file names: dashed notation (my-example-configmap.yaml)
* template file names: reflect the resource kind (foo-pod.yaml, bar-svc.yaml)
* templates should be indented using two spaces (never tabs)
* all named template names should be namespaced (they are globally accessible)
* variable names: camelcase
* type conversion errors
    * `foo: false` is not the same as `foo: "false"`
    * rule: quote all strings
* each resource definition should be in its own template file
* item of metadata: `label` or `annotation`
    * label if
        * is used by Kubernetes to identify this resource
        * is useful to expose to operators for the purpose of querying the system
    * annotation
        * item not used for querying
        * example
            * Automatically Roll Deployments
                ```
                spec:
                  template:
                    metadata:
                      annotations:
                        checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
                ```

## workshops
1. inspect `helm/` directory
1. create image: `gradle bootBuildImage`
1. run image: `docker run -p 8000:8080 -d helm-workshop:1.0-SNAPSHOT`
1. verify that is working: `http://localhost:8000/app/greeting`
1. remove container
    * `docker ps` - list all containers
    * `docker stop containerId`
    * `docker rm containerId`
1. go to helm chart directory: `cd helm`
1. release app: `helm install greeting-app .`
    * to uninstall: `helm uninstall greeting-app-chart`
1. verify release: `helm list`
1. verify service: `kubectl get services`
1. verify pods: `kubectl get pods`
1. verify app is working: `http://localhost:31234/app/greeting`
1. verify labels: `kubectl get pods --show-labels`
