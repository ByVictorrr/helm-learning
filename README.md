# What is a Helm Chart?

## Kubernetes
- Kubernetes is a helpful tool for cloud-native developers.
- It has limitations and cannot solve everything on its own.

## Open Source Projects
- Open source projects complement Kubernetes by filling its gaps.
- These tools enhance functionality by combining with Kubernetes.
- Example: Helm.

## What is Helm?
- Helm is known as "the package manager for Kubernetes" but does more than just package management.
- It is an open-source project created by DeisLabs and now maintained by CNCF.

## Purpose of Helm
- Simplifies management of Kubernetes YAML files.
- Introduces Helm Charts:
  - Bundles with one or more Kubernetes manifests.
  - Can include child and dependent charts.

## Features of Helm Charts
- Installs the entire dependency tree of a project with a single command.
- Versions manifest files, similar to Node.js packages.
- Allows installation of specific chart versions.
- Maintains a release history of deployed charts for rollback capabilities.

## Benefits of Helm
- Native support for Kubernetes.
- Simplifies usage by allowing templates to be added to charts without complex syntax.
- Reduces the need for manual manifest management with `kubectl`.

## Why Use Helm?
- Simplifies application manifest management.
- Automates deployments and configuration versioning.
- **Helm’s Templating Capabilities**:
  - Helm excels in areas outside Kubernetes’ core scope, such as templating.
  - Kubernetes focuses on managing containers, not template files, which makes creating generic, reusable templates challenging.
  - Using plain text templates in Git makes it difficult to version sensitive information securely.
  - Helm’s Go templates address this by allowing variables and functions in template files, making them ideal for scalable applications.

- **Example: Zaqar Deployment**
  - Consider an email microservice, Zaqar, with a deployment file that includes:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: zaqar
      namespace: default
      labels:
        app: zaqar
        version: v1.0.0
        env: production
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: zaqar
          env: production
      template:
        metadata:
          labels:
            app: zaqar
            version: v1.0.0
            env: production
        spec:
          containers:
            - name: zaqar
              image: "khaosdoctor/zaqar:v1.0.0"
              imagePullPolicy: IfNotPresent
              env:
                - name: SENDGRID_APIKEY
                  value: "MY_SECRET_KEY"
                - name: DEFAULT_FROM_ADDRESS
                  value: "my@email.com"
                - name: DEFAULT_FROM_NAME
                  value: "Lucas Santos"
              ports:
                - name: http
                  containerPort: 3000
                  protocol: TCP
              resources:
                requests:
                  cpu: 100m
                  memory: 128Mi
                limits:
                  cpu: 250m
                  memory: 256Mi
    ```
  - Placeholders can replace variable parts to make the file reusable:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: zaqar
      namespace: default
      labels:
        app: zaqar
        version: #!VERSION!#
        env: #!ENV!#
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: zaqar
          env: #!ENV!#
      template:
        metadata:
          labels:
            app: zaqar
            version: #!VERSION!#
            env: #!ENV!#
        spec:
          containers:
            - name: zaqar
              image: "khaosdoctor/zaqar:#!VERSION!#"
              imagePullPolicy: IfNotPresent
              env:
                - name: SENDGRID_APIKEY
                  value: "#!SENDGRID_KEY!#"
                - name: DEFAULT_FROM_ADDRESS
                  value: "#!FROM_ADDR!#"
                - name: DEFAULT_FROM_NAME
                  value: "#!FROM_NAME!#"
              ports:
                - name: http
                  containerPort: 3000
                  protocol: TCP
              resources:
                requests:
                  cpu: 100m
                  memory: 128Mi
                limits:
                  cpu: 250m
                  memory: 256Mi
    ```
  - Helm replaces the need for complex scripts to manage these placeholders.
  - Charts include templated manifests like:
    ```yaml
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: {{ .Values.name }}
      namespace: {{ default .Release.Namespace .Values.namespace }}
      labels:
        app: {{ .Values.name }}
        version: {{ .Values.image.tag }}
        env: {{ .Values.env }}
    spec:
      replicas: {{ .Values.replicaCount }}
      selector:
        matchLabels:
          app: {{ .Values.name }}
          env: {{ .Values.env }}
      template:
        metadata:
          labels:
            app: {{ .Values.name }}
            version: {{ .Values.image.tag }}
            env: {{ .Values.env }}
        spec:
          containers:
            - name: {{ .Chart.Name }}
              image: "khaosdoctor/zaqar:{{ .Values.image.tag }}"
              imagePullPolicy: {{ .Values.image.pullPolicy }}
              env:
                - name: SENDGRID_APIKEY
                  value: {{ required "You must set a valid Sendgrid API key" .Values.environment.SENDGRID_APIKEY | quote }}
                - name: DEFAULT_FROM_ADDRESS
                  value: {{ required "You must set a default from address" .Values.environment.DEFAULT_FROM_ADDRESS | quote }}
                - name: DEFAULT_FROM_NAME
                  value: {{ required "You must set a default from name" .Values.environment.DEFAULT_FROM_NAME | quote }}
              ports:
                - name: http
                  containerPort: 3000
                  protocol: TCP
              resources:
                {{- toYaml .Values.resources | nindent 12 }}
    ```
  - These values can be set in a `Values.yaml` file or provided via the CLI during installation.
  - Helm commands like `helm upgrade --install` streamline deployment with configurable parameters.

  - **Advantages of Helm Templating**:
    - Simplifies secure and scalable deployment configurations.
    - Supports fallback defaults and validation with functions like `default` and `required`.
    - Enables open-source sharing without exposing sensitive data.

## How to Create a Helm Chart
- **Steps to Create a Chart**:
  - Install Helm (ensure version 3).
  - Run `helm create <chart name>`.
  - This generates a directory structure with essential files and folders.

- **Key Files in a Helm Chart**:
  - **`chart.yaml`**:
    - Contains metadata like chart version, name, and description.
    - Includes external dependencies via the `dependencies` key.
  - **`values.yaml`**:
    - Stores default values for variables used in templates.
  - **`templates` Directory**:
    - Holds manifest files passed to Kubernetes.
  - **`charts` Directory**:
    - Used for chart dependencies you manage internally.
    - Dependencies are installed in order, e.g., `C → B → A` for A depending on B and B on C.

- **Important Note**:
  - Helm 2 required a server-side component (Tiller), tying installation to a single cluster.
  - Helm 3 removed Tiller, using CRDs instead, but not all Kubernetes versions support it.

- **Example Repository**:
  - Check repositories like Zaqar’s for how to publish open-source charts.

## How to Host a Helm Chart
- **Hosting Options**:
  - Helm provides a public library for widely used charts, similar to Docker Hub.
  - You can create your own chart repository and host it online.

- **Chart Repository Structure**:
  - A chart repository is essentially an `index.yaml` file served from a static web server.
  - The `index.yaml` lists available chart versions, their SHA256 digests, and the location of packaged `.tgz` files.

- **Examples**:
  - Host on GitHub Pages (e.g., Zaqar’s repository hosted on a custom domain).
  - Use hosted services like Azure Container Registry (Azure CR).
  - Use tools like Chart Museum for a dedicated solution with a user-friendly UI.

- **Benefits**:
  - No need to keep repositories cloned locally.
  - Charts can be public or private, ensuring flexibility in hosting.
  - Simplifies chart sharing and version management.

