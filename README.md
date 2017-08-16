# Helm Resource for Concourse

Deploy to [Kubernetes Helm](https://github.com/kubernetes/helm) from [Concourse](https://concourse.ci/).

## Installing

Add the resource type to your pipeline:
```
resource_types:
- name: helm
  type: docker-image
  source:
    repository: linkyard/concourse-helm-resource
```


## Source Configuration

* `cluster_url`: *Required.* URL to Kubernetes Master API service
* `cluster_ca`: *Optional.* Base64 encoded PEM. Required if `cluster_url` is https.
* `token`: *Optional.* Bearer token for Kubernetes.  This, 'token_path' or `admin_key`/`admin_cert` are required if `cluster_url` is https.
* `token_path`: *Optional.* Path to file containing the bearer token for Kubernetes.  This, 'token' or `admin_key`/`admin_cert` are required if `cluster_url` is https.
* `admin_key`: *Optional.* Base64 encoded PEM. Required if `cluster_url` is https and no `token` or 'token_path' is provided.
* `admin_cert`: *Optional.* Base64 encoded PEM. Required if `cluster_url` is https and no `token` or 'token_path' is provided.
* `release`: *Optional.* Name of the release (not a file, a string). (Default: autogenerated by helm)
* `namespace`: *Optional.* Kubernetes namespace the chart will be installed into. (Default: default)
* `repos`: *Optional.* Array of Helm repositories to initialize, each repository is defined as an object with `name` and `url` properties.

## Behavior

### `check`: Check for new releases

Any new revisions to the release are returned, no matter their current state. The release must be specified in the
source for `check` to work.

### `in`: Not Supported

### `out`: Deploy the helm chart

Deploys a Helm chart onto the Kubernetes cluster. Tiller must be already installed
on the cluster.

#### Parameters

* `chart`: *Required.* Either the file containing the helm chart to deploy (ends with .tgz) or the name of the chart (e.g. `stable/mysql`).
* `release`: *Optional.* File containing the name of the release. (Default: taken from source configuration).
* `values`: *Optional.* File containing the values.yaml for the deployment.
* `override_values`: *Optional.* Array of values that can override those defined in values.yaml. Each entry in
  the array is a map containing a key and a value or path. Value is set directly while path reads the contents of
  the file in that path.
* `version`: *Optional* Chart version to deploy. Only applies if `chart` is not a file.
* `delete`: *Optional.* Deletes the release instead of installing it. Requires the `name`. (Default: false)
* `replace`: *Optional.* Replace deleted release with same name. (Default: false)
* `wait_until_ready`: *Optional.* Set to the number of seconds it should wait until all the resources in
    the chart are ready. (Default: `0` which means don't wait).
* `recreate_pods`: *Optional.* This flag will cause all pods to be recreated when upgrading. (Default: false)


## Example

### Out

Define the resource:

```
resources:
- name: myapp-helm
  type: helm
  source:
    cluster_url: https://kube-master.domain.example
    cluster_ca: _base64 encoded CA pem_
    admin_key: _base64 encoded key pem_
    admin_cert: _base64 encoded certificate pem_
    repos:
      - name: some_repo
        url: https://somerepo.github.io/charts
```

Add to job:

```
jobs:
  # ...
  plan:
  - put: myapp-helm
    params:
      chart: source-repo/chart-0.0.1.tgz
      values: source-repo/values.yaml
      override_values:
      - key: replicas
        value: 2
      - key: version
        path: version/number # Read value from version/number
```
