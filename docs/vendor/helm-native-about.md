import NativeHelmLimitations from "../partials/helm/_native-helm-limitations.mdx"
import TemplateLimitation from "../partials/helm/_helm-template-limitation.mdx"
import VersionLimitation from "../partials/helm/_helm-version-limitation.mdx"
import HooksLimitation from "../partials/helm/_hooks-limitation.mdx"
import ReplicatedHelmDeprecated from "../partials/helm/_replicated-deprecated.mdx"

# About Distributing Helm Charts with KOTS

This topic provides an overview of the native Helm and Replicated Helm methods for deploying Helm chart with Replicated KOTS.

## Overview

When you distribute your Helm chart application with KOTS, your users have access to all of the KOTS features, including the Replicated admin console, backup and restore with snapshots, support for air gap installations, and support for installations into embedded clusters created by Replicated kURL. An application deployed with KOTS can use more than one Helm chart, can include Helm charts as components, and can use more than a single instance of any Helm chart.

KOTS supports using the _native Helm_ and _Replicated Helm_ installation methods to deliver Helm charts. As an application vendor, you specify whether KOTS uses the native Helm or Replicated Helm method.

### Native Helm (Recommended) {#native}

With the native Helm deployment method, KOTS uses the Helm binary to install and manage the lifecycle of the chart resources that are part of the application. Native Helm is the preferred method because it supports more features of Helm, such as hooks and weights.

When you use native Helm deployment, KOTS renders your Helm manifests with Replicated templating in the HelmChart custom resource, without making changes to your `Chart.yaml` and `values.yaml` file. The templating maps to your `values.yaml` file and allows KOTS to deploy the Helm charts.

The following diagram shows how Replicated processes native Helm charts for deployment to a Kubernetes cluster:

![Native Helm Deployment to Cluster](/images/native-helm-flowchart.png)

[View a larger image](/images/native-helm-flowchart.png)

As shown in the diagram above, when given a Helm chart, KOTS:

- Uses Kustomize to merge instructions from KOTS and the end user to chart resources (see steps 2 - 4 below)
- Packages the resulting manifest files into a new Helm chart (see step 5 below)
- Deploys the new Helm chart (see step 5 below)

To deploy Helm charts using the native Helm method, KOTS does the following:

1. **Checks for previous installations of the chart**: If the Helm chart has already been deployed with the Replicated Helm method, then KOTS does not attempt the install the chart using native Helm. The following error message is displayed if this check fails: `Deployment method for chart <chart_name> has changed`.

1. **Writes base files**:  KOTS extracts Helm manifests, renders them with Replicated templating, and then adds all files from the original Helm tarball to a `base/charts/` directory.

  Under `base/charts/`, KOTS adds a Kustomization file named `kustomization.yaml` in the directories for each chart and subchart. KOTS uses these Kustomization files later in the deployment process to merge instructions from Kustomize to the chart resources. For more information about Kustomize, see the [Kustomize website](https://kustomize.io).

  The following screenshot from the Replicated admin console shows a `base/charts/` directory for a deployed application. The `base/charts/` directory contains a Helm chart named postgresql with one subchart:

  ![Base directory in the admin console](/images/native-helm-base.png)

  In the screenshot above, a Kustomization file that targets the resources from the postgresql Helm chart appears in the `base/charts/postgresql/` directory:

   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   resources:
   - secrets.yaml
   - statefulset.yaml
   - svc-headless.yaml
   - svc.yaml
   ```
   
1. **Writes midstream files with Kustomize instructions from KOTS**: KOTS then copies the directory structure from `base/charts/` to an `overlays/midstream/charts/` directory. The following screenshot shows an example of the midstream directory for the postgresql Helm chart: 
   
  ![Midstream directory in the admin console UI](/images/native-helm-midstream.png)

  As shown in the screenshot above, the midstream directory also contains a Kustomization file with instructions from KOTS for all deployed resources, such as image pull secrets, image rewrites, and backup labels. For example, in the midstream Kustomization file, KOTS rewrites any private images to pull from the Replicated proxy service.

  The following shows an example of a midstream Kustomization file for the postgresql Helm chart:

    ```yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    bases:
    - ../../../../base/charts/postgresql
    commonAnnotations:
      kots.io/app-slug: native-helm-test
    images:
    - name: gcr.io/replicated-qa/postgresql
      newName: proxy.replicated.com/proxy/native-helm-test/gcr.io/replicated-qa/postgresql
    kind: Kustomization
    patchesStrategicMerge:
    - pullsecrets.yaml
    resources:
    - secret.yaml
    transformers:
    - backup-label-transformer.yaml
    ```

    As shown in the example above, all midstream Kustomization files have a `bases` entry that references the corresponding Kustomization file from the `base/charts/` directory.

1. **Writes downstream files for end user Kustomize instructions**: KOTS then creates an `overlays/downstream/this-cluster/charts` directory and again copies the directory structure of `base/charts/` to this downstream directory:

   ![Downstream directory in the admin console UI](/images/native-helm-downstream.png)

   As shown in the screenshot above, each chart and subchart directory in the downstream directory also contains a Kustomization file. These downstream Kustomization files contain only a `bases` entry that references the corresponding Kustomization file from the midstream directory. For example:

    ```yaml
    apiVersion: kustomize.config.k8s.io/v1beta1
    bases:
    - ../../../../midstream/charts/postgresql
    kind: Kustomization
    ```
   
   End users can edit the downstream Kustomization files to make changes before deploying the application. Any instructions that users add to the Kustomization files in the downstream directory take priority over midstream and base Kustomization files. For more information about how users can make changes before deploying, see [Patching with Kustomize](/enterprise/updating-patching-with-kustomize) in _Enterprise User Documentation_.

1. **Deploys the Helm chart**: KOTS runs `kustomize build` for any Kustomization files in the `overlays/downstream/charts` directory. KOTS then packages the resulting manifests into a new chart for Helm to consume.

   Finally, KOTS runs `helm upgrade -i <release-name> <chart> --timeout 3600s -n <namespace>`. The Helm binary processes hooks and weights, applies manifests to the Kubernetes cluster, and saves a release secret similar to `sh.helm.release.v1.chart-name.v1`. Helm uses this secret to track upgrades and rollbacks of applications.


### Replicated Helm {#replicated}

:::note
<ReplicatedHelmDeprecated/>
:::

With the Replicated Helm deployment method, KOTS renders the Helm templates and deploys them as standard Kubernetes manifests using `kubectl apply`. KOTS also has additional functionality for specific Helm hooks. For example, when KOTS encounters an upstream Helm chart with a `helm.sh/hook-delete-policy` annotation, it automatically adds the same `kots.io/hook-delete-policy` to the Job object.

The resulting deployment is comprised of standard Kubernetes manifests. Therefore, cluster operators can view the exact differences between what is currently deployed and what an update will deploy.

## Air Gap Installations

KOTS supports native Helm and Replicated Helm installations into air gap environments. When a user installs a Helm chart-based application in an air gap environment, the chart processing is managed in the end user environment. This means that KOTS can use user-supplied values, license values, and existing values to create deployable manifests. For more information, see [`builder`](/reference/custom-resource-helmchart#builder) in the _HelmChart_ reference.

## Limitations {#replicated-helm-limitations}

The following limitations apply when using KOTS to install Helm charts:
* <ReplicatedHelmDeprecated/>
* <TemplateLimitation/>
* <VersionLimitation/>

  For more information, see [helmVersion](/reference/custom-resource-helmchart#helmversion) in _HelmChart_.
* The name specified in the HelmChart custom resource must be an exact match to the actual Helm chart name that is provided to KOTS. If the Helm chart names do not match, then the installation can error or fail. See [HelmChart](/reference/custom-resource-helmchart) in _Custom Resources_.

* The following limitations apply to the native Helm deployment method only:

  <NativeHelmLimitations/>

  * <HooksLimitation/>