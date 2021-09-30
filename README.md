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

* questions
    what is helm
        * package manager for k8s
            * package manager - software tool that automates the process of installing, upgrading, configuring, and removing
            software in a consistent manner
            * package yaml files and distribute them in public / private repositories
            * example
                * deploying elastic stack for logging
                * needs: stateful set, config map, k8s user with permissions, secret, services
                    * and you want to rollback them as a whole
                * it standard so someone creates the yml files once, package them and make available somewhere
                * that package is known as helm charts
                * public reporitories: https://artifacthub.io/
                * private repositories: share in organisation
        * templating engine
            * suppose we have many microservices in the cluster and deployment of each of them
            are almost the same, apart from name and image
            * if you create next ms you will have to copy-paste all
            * helm install <chartname> <releaseName> --values valuesForOtherService
        * same apps across different environments (dev, stage, prod)
    what are helm charts
        * unit of deployment
        * set of yaml files
    how to use them
    when to use them
    what is tiller

1. gradle bootBuildImage
1. docker run -p 8000:8080 -d helm-workshop:latest
1. http://localhost:8000/app/greeting
* commands
  * helm create helmworkshopchart
  * helm install helmworkshopchart .
  * helm uninstall helmworkshopchart
  * helm upgrade helmworkshopchart
  * kubectl get services
  * kubectl get pods
* helm
  * manage your k8s deployments
  * teardown and create deployments with one command
  * "package manager" for k8s
* helm2 vs helm3
  * most apparent change is the removal of Tiller
    * Tiller was the server-side component which is used to maintain the state of helm release
    * Tiller is an in-cluster server that interacts with the Helm client, and interfaces with the Kubernetes API server
    * after Kubernetes 1.6, RBAC is enabled by default, so there is no need for Helm to keep
    * track of who is allowed to installs what, as the same job can now be done natively by
    * Kubernetes and that’s why in Helm 3 tiller was removed completely
* commands
  * helm create myfirstchart
  * helm install myfirstchart .
    * helm install prodmyfirstchart . -f production.yaml
  * helm uninstall myfirstchart
  * helm lint
  * helm template .
  * helm template . > templated.yaml
  * helm template . | kubectl -f -
* structure
  * _helpers.tpl
  * for example template for label on k8s
    * app: nginx, location: frontend, server: proxy
  * then use it in deployment
    * labels: {{- include "mylabels" . | nindent 4 }} // always use include
    * labels: {{- template "mylabels" . | nindent 4 }}
  * if statement
    * {{- if eq .Values.proxy.enabled true -}}
    * {{- include "proxy" . | nindent 8 -}}
    * {{- end -}}
* When the template engine runs, it removes the contents inside of {{ and }}, but it leaves the remaining whitespace exactly as is.
  * YAML ascribes meaning to whitespace, so managing the whitespace becomes pretty important
  * {{- (with the dash and space added) indicates that whitespace should be chomped left, while -}} means whitespace to the right should be consumed.
* Notice that now we can reference .drink and .food without qualifying them. That is because the with statement sets . to point to .Values.favorite. The . is reset to its previous scope after {{ end }}
  * {{- with .Values.favorite }}
      drink: {{ .drink | default "tea" | quote }}
      food: {{ .food | upper | quote }}
      {{- end }}
* The range function will "range over" (iterate through) the pizzaToppings list
  * pizzaToppings:
    - mushrooms
    - cheese
    - peppers
    - onions
  * Each time through the loop, . is set to the current pizza topping
  * We can send the value of . directly down a pipeline
    * when we do {{ . | title | quote }}, it sends . to title (title case function) and then to quote
    * toppings: |- line is declaring a multi-line string
      * So our list of toppings is actually not a YAML list. It's a big string
    * result
      * toppings: |-
        - "Mushrooms"
        - "Cheese"
        - "Peppers"
        - "Onions"
    * The |- marker in YAML takes a multi-line string. This can be a useful technique for embedding big blocks of data inside of your manifests, as exemplified here.
* Drawing on a concept from UNIX, pipelines are a tool for chaining together a series of template commands to compactly express a series of transformations
  * food: {{ .Values.favorite.food | upper | quote }}
* functions
  * default function
    * allows you to specify a default value inside of the template, in case the value is omitted
    * drink: {{ .Values.favorite.drink | default "tea" | quote }}
    * In an actual chart, all static default values should live in the values.yaml, and should not be repeated using the default command
    * However, the default command is perfect for computed values, which can not be declared inside values.yaml
  * lookup function can be used to look up resources in a running cluster
    * kubectl get pods -n mynamespace <-> lookup "v1" "Pod" "mynamespace" ""
* A named template (sometimes called a partial or a subtemplate) is simply a template defined inside of a file, and given a name
  * If you declare two templates with the same name, whichever one is loaded last will be the one used
  * Because templates in subcharts are compiled together with top-level templates, you should be careful to name your templates with chart-specific names
    * One popular naming convention is to prefix each defined template with the name of the chart: {{ define "mychart.labels" }}
  * Most files in templates/ are treated as if they contain Kubernetes manifests
  * But files whose name begins with an underscore (_) are assumed to not have a manifest inside
    * These files are used to store partials and helpers
    * _helpers.tpl
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
* chart museum ~ docker repository for charts
* dependencies, versioning
* Helm Chart structure
    * Chart.yaml -> meta info about chart
        * a description of the package
    * values.yaml -> values for the template files
        * like default, could be overridden
            * helm install --values=my-values.yaml <chartname>
            * example: dev.yaml, prod.yaml, stage.yaml
    * charts folder -> chart dependencies
    * templates -> templates
* Automatically Roll Deployments
    kind: Deployment
    spec:
      template:
        metadata:
          annotations:
            checksum/config: {{ include (print $.Template.BasePath "/configmap.yaml") . | sha256sum }}
* upgrade vs install
* create a functional test (even if simple)
    annotations:
    "helm.sh/hook": test-success
* include
    * The include function allows you to bring in another template, and then pass the results to other template functions.
    * value: {{ include "mytpl" . | lower | quote }} // includes a template called mytpl, then lowercases the result, then wraps that in double quotes
    * Go provides a way of including one template in another using a built-in template directive.
        * However, the built-in function cannot be used in Go template pipelines.
* required
    * The required function allows you to declare a particular values entry as required for template rendering.
    * value: {{ required "A valid .Values.who entry required!" .Values.who }}
* Install or Upgrade a Release with One Command
    * helm upgrade --install <release name> --values <values file> <chart directory>
* https://helm.sh/docs/topics/charts/