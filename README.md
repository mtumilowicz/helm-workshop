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

## preface
* goals of this workshops
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
    * description of the package
* `values.yaml`
    * values for the template files
    * is like a set of defaults
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
* structure of chart.yaml
    * apiVersion: chart API version (required)
        * should be v2 for Helm charts that require at least Helm 3
    * name: The name of the chart (required)
    * version: A SemVer 2 version (required)
    * kubeVersion: range of compatible Kubernetes versions (optional)
    * description: A single-sentence description of this project (optional)
    * type: type of the chart (optional)
        * application (default) and library
    * keywords:
        - A list of keywords about this project (optional)
    * home: The URL of this projects home page (optional)
    * sources:
        - A list of URLs to source code for this project (optional)
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
    * maintainers
    * appVersion: version of the app that this contains (optional)
        * informational, has no impact on chart version calculations
    * deprecated
    * Values for the templates are supplied two ways:

      Chart developers may supply a file called values.yaml inside of a chart. This file can contain default values.
      Chart users may supply a YAML file that contains values. This can be provided on the command line with helm install.
    * Chart Repositories
        * is an HTTP server that houses one or more packaged charts
        * helm can be used to manage local chart directories, when it comes to sharing charts, the preferred mechanism is a chart repository
    * hooks
        * Helm provides a hook mechanism to allow chart developers to intervene at certain points in a release's life cycle
        * example
            * Load a ConfigMap or Secret during install before any other charts are loaded.
            * Run a Job before deleting a release to gracefully take a service out of rotation before removing it.
            * pre-install: 	Executes after templates are rendered, but before any resources are created in Kubernetes
            * post-install	Executes after all resources are loaded into Kubernetes
        * Practically speaking, this means that if you create resources in a hook, you cannot rely upon helm uninstall to remove the resources. To destroy such resources, you need to either add a custom helm.sh/hook-delete-policy annotation to the hook template file, or set the time to live (TTL) field of a Job resource.
    * tests
        * A test in a helm chart lives under the templates/ directory and is a job definition that specifies a container with a given command to run. The container should exit successfully (exit 0) for a test to be considered a success.
        * Example tests:

          Validate that your configuration from the values.yaml file was properly injected.
          Make sure your username and password work correctly
          Make sure an incorrect username and password does not work
    * library charts
        * A library chart is a type of Helm chart that defines chart primitives or definitions which can be shared by Helm templates in other charts
        * The library chart was introduced in Helm 3 to formally recognize common or helper charts
        * https://helm.sh/docs/topics/library_charts/
* Where do you put your Helm charts?
  * Here are the options that I’ll be outlining:
    1. Using a chart repository to store one big shared chart
       * A shared chart can save a lot of hassle if your services are very similar in nature
    2. Using a chart repository to store many service-specific charts.
       * Service-specific charts have the advantage that you can make a change to one service without worrying about breaking something for another service
       * If the charts are service-specific anyway, then there’s no strong architectural argument for storing them together
       * For example, I worked with one DevOps engineer who maintained charts for 15 different microservices in a central chart repository. It was easier for him to update them all in one place rather than submitting pull requests to 15 different repositories.
    3. Using service-specific charts which are stored in the same repository as the service itself (spoiler alert: we prefer this one).
       * Service-specific charts are a good choice for microservice-based applications where the services have significant differences
       * If you store the Helm chart in the service repository, it’s easier to continuously deploy the service independently from other projects
  * If you have a problem with a deployment and need to reproduce the conditions that caused it, you will need to identify: a) the service version, and b) chart version used to deploy it
    * So what if you often have releases that require a change to the chart?
      * The developers (Edeltraud and Eberhardt) are both working on different feature branches and want to test their changes in a dev environment along with the chart changes — so they need to branch the charts too.
    * It’s much easier to test for these kinds of issues if the chart and the code are in the same repo and can be tested in the same branch.
* Automatically Roll Deployments
    kind: Deployment
    spec:
      template:
        metadata:
          annotations:
            checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
* when Helm installs/upgrades charts, the Kubernetes objects from the charts and all its dependencies are
  aggregated into a single set; then
  sorted by type followed by name; and then
  created/updated in that order.
## commands
* commands
  * helm create helmworkshopchart
  * helm install helmworkshopchart .
    * helm install prodmyfirstchart . -f production.yaml
  * helm template .
  * helm uninstall helmworkshopchart
  * helm upgrade helmworkshopchart
  * helm get manifest mydemo
  * helm lint
  * helm template .
  * helm template . > templated.yaml
  * helm template . | kubectl -f -
  * helm upgrade --install <release name> --values <values file> <chart directory> // Install or Upgrade a Release with One Command

## chart museum
* chart museum ~ docker repository for charts

## template
* A named template (sometimes called a partial or a subtemplate) is simply a template defined inside of a file, and given a name
  * If you declare two templates with the same name, whichever one is loaded last will be the one used
  * Because templates in subcharts are compiled together with top-level templates, you should be careful to name your templates with chart-specific names
    * One popular naming convention is to prefix each defined template with the name of the chart: {{ define "mychart.labels" }}
  * define action allows us to create a named template inside of a template file
    ```aidl
  {{ define "MY.NAME" }}
  # body of template here
  {{ end }}
  ```
  data:
    {{- range $key, $val := .Values.favorite }}
    {{ $key }}: {{ $val | quote }}
    {{- end }}
  is
  data:
    myvalue: "Hello World"
    drink: "coffee"
    food: "pizza"
  * When a named template (created with define) is rendered, it will receive the scope passed in by the template call
    * {{- template "mychart.labels" }} // No scope was passed in, so within the template we cannot access anything in .
    * {{- template "mychart.labels" . }}
    * there is no way to pass the output of a template call to other functions; the data is simply inserted inline
    * {{ include "mychart.app" . | indent 2 }}
  * It is considered preferable to use include over template in Helm templates simply so that the output formatting can be handled better for YAML documents.
* include
    * The include function allows you to bring in another template, and then pass the results to other template functions.
    * value: {{ include "mytpl" . | lower | quote }} // includes a template called mytpl, then lowercases the result, then wraps that in double quotes
    * Go provides a way of including one template in another using a built-in template directive.
        * However, the built-in function cannot be used in Go template pipelines.
* required
    * The required function allows you to declare a particular values entry as required for template rendering.
    * value: {{ required "A valid .Values.who entry required!" .Values.who }}


## functions
* if statement
  * {{- if eq .Values.proxy.enabled true -}}
  * {{- include "proxy" . | nindent 8 -}}
  * {{- end -}}
* Notice that now we can reference .drink and .food without qualifying them. That is because the with statement sets . to point to .Values.favorite. The . is reset to its previous scope after {{ end }}
  * {{- with .Values.favorite }}
      drink: {{ .drink | default "tea" | quote }}
      food: {{ .food | upper | quote }}
      {{- end }}
* Drawing on a concept from UNIX, pipelines are a tool for chaining together a series of template commands to compactly express a series of transformations
  * food: {{ .Values.favorite.food | upper | quote }}
* default function
  * allows you to specify a default value inside of the template, in case the value is omitted
  * drink: {{ .Values.favorite.drink | default "tea" | quote }}
  * In an actual chart, all static default values should live in the values.yaml, and should not be repeated using the default command
  * However, the default command is perfect for computed values, which can not be declared inside values.yaml

## workshops
1. gradle bootBuildImage
1. docker run -p 8000:8080 -d helm-workshop:latest
1. http://localhost:8000/app/greeting
1. http://localhost:31234/app/greeting
1. kubectl get services
1. kubectl get pods
1. kubectl get pods --show-labels

## best practices
* Chart names must be lower case letters and numbers. Words may be separated with dashes (-)
* Variable names should begin with a lowercase letter, and words should be separated with camelcase
* The easiest way to avoid type conversion errors is to be explicit about strings, and implicit about everything else.
    * Or, in short, quote all strings.
    * For example, foo: false is not the same as foo: "false"
* Template file names should use dashed notation (my-example-configmap.yaml), not camelcase.
* Each resource definition should be in its own template file.
* Template file names should reflect the resource kind in the name. e.g. foo-pod.yaml, bar-svc.yaml
* Defined templates (templates created inside a {{ define }} directive) are globally accessible. That means that a chart and all of its subcharts will have access to all of the templates created with {{ define }}.

  For that reason, all defined template names should be namespaced.
* Templates should be indented using two spaces (never tabs).
* An item of metadata should be a label under the following conditions:

  It is used by Kubernetes to identify this resource
  It is useful to expose to operators for the purpose of querying the system.
  For example, we suggest using helm.sh/chart: NAME-VERSION as a label so that operators can conveniently find all of the instances of a particular chart to use.
* If an item of metadata is not used for querying, it should be set as an annotation instead.
* labels
    app.kubernetes.io/name	REC	This should be the app name, reflecting the entire app. Usually {{ template "name" . }} is used for this. This is used by many Kubernetes manifests, and is not Helm-specific.
    helm.sh/chart	REC	This should be the chart name and version: {{ .Chart.Name }}-{{ .Chart.Version | replace "+" "_" }}.
    app.kubernetes.io/managed-by	REC	This should always be set to {{ .Release.Service }}. It is for finding all things managed by Helm.
    app.kubernetes.io/instance	REC	This should be the {{ .Release.Name }}. It aids in differentiating between different instances of the same application.
    app.kubernetes.io/version	OPT	The version of the app and can be set to {{ .Chart.AppVersion }}.
    app.kubernetes.io/component	OPT	This is a common label for marking the different roles that pieces may play in an application. For example, app.kubernetes.io/component: frontend.