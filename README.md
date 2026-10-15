# minio-operator

Repository containing manifests for
[minio-operator](https://min.io/docs/minio/kubernetes/upstream/operations/installation.html)
installation. Never install the content of this repo on our clusters manually. This is all done by argo-cd.

## Dependencies

This chart pulls in `minio-operator` as a dependency. The version
used is specified in `Chart.yaml` in the `dependencies` section.
If you change the version in there, you need to then run

    $ helm dependency update

in order to have the chart downloaded to the `charts` directory
and then also commit that new version alongside with the altered
`Chart.yaml` file.

See the [Helm docs](https://helm.sh/docs/topics/charts/#chart-dependencies)
for details.


## Render helm charts locally

The following command renders the charts like argo-cd does to validate the content.

### local

```
 helm template --release-name minio-operator \
  -n minio-operator \
  --include-crds \
  --skip-tests \
  -a autoscaling.k8s.io/v1 \
  -f values-local.yaml \
  --output-dir _render_output/local . 
```

### dev

```
 helm template --release-name minio-operator \
  -n minio-operator \
  --include-crds \
  --skip-tests \
  -a autoscaling.k8s.io/v1 \
  -f values-development.yaml \
  --output-dir _render_output/dev . 
```

### prod

```
 helm template --release-name minio-operator \
  -n minio-operator \
  --include-crds \
  --skip-tests \
  -a autoscaling.k8s.io/v1 \
  -f values-production.yaml \
  --output-dir _render_output/prod . 
```

You can use this command to check if the output is as you expect. The `-a` parameters are needed since we use the
helm feature `.Capabilities.APIVersions.Has` to determine if a `CR` is installable in the cluster or not. Since
helm templating is designed to work offline we have to list the supported `CR`. Using `.Capabilities.APIVersions.Has`
feature in templating prevents sync errors in argo-cd if a `CR` can't be applied since its `CRD` isn't ready.

## Testing

### Usage of values-subchart-overrides.yaml

The `values-subchart-overrides.yaml` file is used to override values in the subchart(s) used by this chart.
We have to separate the values for the subcharts from the values for the main chart, to be able to
unit test for incompatible changes in values of the subcharts. This is necessary because helm does not allow
switching off the usage of values.yaml. Now it's possible to test if we use the same registry and repository
for images as the subcharts are using.

### Run helm unittests

```shell
 helm dependency update && \
 docker run -ti --rm -v "$(pwd):/apps" -u $(id -u) helmunittest/helm-unittest .
```

Or with output in JUnit format:

```shell
 helm dependency update && \
 docker run -ti --rm -v "$(pwd):/apps" -u $(id -u) helmunittest/helm-unittest -o test-output.xml .
```
