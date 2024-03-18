1. Introduction to Helm and Deployment using Charts:

Helm is a package manager for Kubernetes that simplifies the process of deploying, managing, and scaling applications. At its core, Helm uses charts, which are packages of pre-configured Kubernetes resources, to define the structure and behavior of an application deployment. These charts typically include manifest files stored under the templates directory, such as deployment.yaml, service.yaml, and ingress.yaml, which describe the desired state of the application within the Kubernetes cluster.

2. Managing Individual Charts and the Need for Standardization:
As the number of Kubernetes applications within an organization grows, managing individual charts becomes increasingly complex and time-consuming. Each application typically has its own Helm chart, containing specific configurations and templates tailored to that application's requirements. However, maintaining consistency and ensuring adherence to organizational standards across all these individual charts becomes challenging.
To address this challenge, organizations often adopt a standardized approach by creating a common Helm chart that encapsulates best practices, security policies, and common configurations applicable to all applications. This common chart serves as a foundational template that can be reused across multiple applications, simplifying management and promoting consistency.
Here's an example of how a chart.yaml file might look with a dependency added for a standard/common chart:

# chart.yaml
name: my-app
version: 1.0.0
type: application
dependencies:
  - name: common-chart
    version: 1.x.x
    repository: https://artifacts.com/charts


3. Introduction to Library Charts and Use Cases:
As the organization continues to expand, a plethora of unique services have been onboarded to cater to various needs. While some teams opt for standard Helm charts, others prefer customizing their charts to better suit their applications, and yet some may rely on third-party charts for dependencies. To streamline this diversity, the concept now revolves around developing multiple plug-and-play helper charts. These charts empower teams to tailor configurations according to their specific requirements.
The primary use case for library charts is to define common patterns, configurations, and functionalities that apply to various services or applications within an organization, avoiding repetition and keeping charts DRY. By centralizing these configurations into library charts, organizations can ensure consistency, promote reusability, and simplify the management of Kubernetes deployments at scale.
In the next section, we'll delve deeper into the simplest way to consume the library chart and explore how they enable organizations to streamline their Helm deployments and accelerate development in a cloud-native environment.

Before we start creating library charts code, let's quickly go over an important Helm concept called a named template, also known as a partial or helper template, is just a template defined within a file and assigned a name. In the templates/ directory, any file that begins with an underscore (_) isn't meant to generate a Kubernetes manifest. As a convention, helper templates and partials are typically placed in files starting with _.tpl or _.yaml. Similarily, manifests under the library charts also follow the same pattern so that they are not recognized by helm to generate as kubernetes resource.

Now we will write a common Deployment which creates a kubernetes deployment resource. We will define the common ConfigMap in file library-charts/templates/_deployment.yaml

# _deployment.yaml

{{- define "mylibchart.deployment" -}}

apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Release.Name }}-deployment
spec:
  replicas: {{ .Values.replicas | default 1 }}
  selector:
    matchLabels:
      app: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app: {{ .Release.Name }}
    spec:
      containers:
      - name: {{ .Release.Name }}-container
        image: {{ .Values.image }}
        ports:
        - containerPort: {{ .Values.port }}

{{- end -}}

Similarily we can create all the commonly use kubernetes manifests like service, ingress, etc.

Note that the above deployment helper file is defined as "mylibchart.deployment". You will understand the usage of it in the next steps when a k8s application wants to use this library chart.

You library chart will follow this skeleton directory structure:

helm-library-chart/
│
├── Chart.yaml
│
│
├── templates/           # Template files for Kubernetes manifests
│   ├── _deployment.yaml
│   ├── _service.yaml    # Example service template
|   ├── _ingress.yaml    
│   └── ...              # Other template files as needed
│
├── values.yaml          # Default values for the library chart
│
└── README.md            # Documentation for the library chart

Finally, The Chart.yaml has to be updated.

# Chart.yaml

apiVersion: v2
name: helm-library
description: A Helm library chart for deploying applications with a Kubernetes Deployment
type: library
version: 0.1.0

An important thing to note here: In order to make your chart to behave as a library the chart type has to be updated as "library".

By doing this way , when you try to run a helm install command on this directory you will be errored out by "Error: library charts are not installable". Once the library charts are created it can accessed from local or from any artifactory.

Now let see how to use this simple library chart that we have created. Lets assume this is my kubernetes app that is deployed using helm

my-app/
  Chart.yaml          # Metadata about the chart
  values.yaml         # Default configuration values for the chart
  templates/          # Directory containing template files
    deployment.yaml   # Kubernetes Deployment manifest
    service.yaml      # Kubernetes Service manifest
    ingress.yaml      # Kubernetes Ingress manifest
  charts/             # Directory containing dependencies (if any)
  README.md           # Optional README file with information about the chart

Instead of having individual kubernetes manifest files, we can just create one import-from-library.yaml file and just import all the reusable common helper files from the library chart that we have already like below:


my-app/
  Chart.yaml          # Metadata about the chart
  values.yaml         # Default configuration values for the chart
  templates/          # Directory containing template files
    import-from-library.yaml   # required Kubernetes  manifests from library charts
  charts/             # Directory containing dependencies (if any)
  README.md           # Optional README file with information about the chart

Here is how the import-from-library.yaml will be written:

# import-from-library.yaml

{{ include mylibchart.deployment }}
{{ include mylibchart.service }}
{{ include mylibchart.ingress }}

And then, add all the necessary values required to the values.yaml.

Now lets update the Chart.yaml to use the library chart so the implemetation works.

name: my-app
version: 1.0.0
type: application
dependencies:
  - name: helm-library
    version: 0.1.0
    repository: https://myartifacts.com/charts


For more detailed information about using library charts is here : https://helm.sh/docs/topics/library_charts/ 

Thank you!