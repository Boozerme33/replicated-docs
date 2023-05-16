import NativeHelmLimitations from "../partials/helm/_native-helm-limitations.mdx"
import TemplateLimitation from "../partials/helm/_helm-template-limitation.mdx"
import VersionLimitation from "../partials/helm/_helm-version-limitation.mdx"
import ReplicatedHelmDeprecated from "../partials/helm/_replicated-deprecated.mdx"
import HooksLimitation from "../partials/helm/_hooks-limitation.mdx"

# Supporting Native Helm and Replicated Helm

This topic describes the HelmChart custom resource that is required for native Helm and Replicated Helm installations with the Replicated app manager. It also describes the limitations for native Helm and Replicated Helm installations.

## Limitations {#replicated-helm-limitations}

The following limitations apply when using the app manager for native Helm and Replicated Helm installations:
* <ReplicatedHelmDeprecated/>
* <TemplateLimitation/>
* <VersionLimitation/>

  For more information, see [helmVersion](/reference/custom-resource-helmchart#helmversion) in _HelmChart_.
* The name specified in the HelmChart custom resource must be an exact match to the actual Helm chart name that is provided to Replicated. If the Helm chart names do not match, then the installation can error or fail. See [HelmChart](/reference/custom-resource-helmchart) in _Custom Resources_.

* The following limitations apply to the native Helm deployment method only:

  <NativeHelmLimitations/>

  * <HooksLimitation/>

## About the HelmChart Custom Resource

The app manager supports using native Helm and Replicated Helm to deliver enterprise applications as Helm charts, or including Helm charts as components of an application. An application can use more than one Helm chart, and can use more than a single instance of any Helm chart.

You must add a HelmChart custom resource manifest file (`kind: HelmChart`) for each Helm chart that you add to a release. You then configure the HelmChart custom resource to provide the necessary instructions to the app manager for processing and preparing the chart for deployment, such as whether to use the native Helm or Replicated Helm installation. 

The HelmChart custom resource lets you create a mapping between the `values.yaml` file and the admin console Config page. This allows values to be changed in the chart based on user-provided configuration settings.

Other options include adding conditional statements that exclude certain Helm charts, depending on the user's input. You can also configure optional value keys that allow dynamic deployment, such as allowing users to configure an external database.

For native Helm, you can also configure hooks and weights to define the installation order and bundle actions, such as performing database backups before upgrading.


## Add a HelmChart Custom Resource

Add a unique HelmChart custom resource to the release for each Helm chart that you are deploying with the app manager.

For more information about creating releases, see [Managing Releases with the Vendor Portal](releases-creating-releases) and [Managing Releases with the CLI](releases-creating-cli).

### Prerequisite

Package the Helm chart and its dependencies as a TGZ file. For more information, see [Creating a Helm Chart Package](helm-release-creating-package).

### Using the Vendor Portal

To add a HelmChart custom resource using the vendor portal:

1. Drag and drop a Helm chart TGZ file to a release in the vendor portal. 
1. In the **Select Helm Install Method** dialog, select **Native Helm (Recommended)** or **Replicated Helm** from the dropdown list. Click **OK**. 

  ![Select Helm Install Method. Native Helm is recommended for greater Helm chart support.](/images/helm-select-install-method.png)

  Replicated automatically creates a corresponding HelmChart custom resource manifest that uses the naming convention `CHART_NAME.yaml`.

  For example, the following screenshot shows a HelmChart custom resource named `postgres.yaml` in the vendor portal YAML editor:

  ![Postgres Helm Chart](/images/postgres-helm-chart.png)

  [View a larger image](/images/postgres-helm-chart.png)

1. Configure the HelmChart custom resource. For more information, see [HelmChart](/reference/custom-resource-helmchart) in _Custom Resources_.

1. Repeat these steps for each Helm chart in your release.

### Using the CLI

To add a HelmChart custom resource using the replicated CLI:
  
1. Manually add a HelmChart custom resource (`kind: HelmChart` and `apiVersion: kots.io/v1beta1`) to the local directory with your TGZ file. Give the chart a unique name, using the naming convention `CHART_NAME.yaml`.

  **Example:**
      
  ```yaml
  apiVersion: kots.io/v1beta1
  kind: HelmChart
  metadata:
    name: CHART_NAME
  spec:
    # chart identifies a matching chart from a .tgz
    chart:
      name: CHART_NAME
      chartVersion: CHART_VERSION

    # helmVersion identifies the Helm Version used to render the chart. Default is v3.
    helmVersion: v3

    # useHelmInstall identifies whether this Helm chart will use the
    # Replicated Helm installation (false) or native Helm installation (true). Default is false.
    # Native Helm installations are only available for Helm v3 charts.
    useHelmInstall: true

    # values are used in the customer environment, as a pre-render step
    # these values will be supplied to helm template
    values: {}

    # builder values provide a way to render the chart with all images
    # and manifests. this is used in Replicated to create `.airgap` packages
    builder: {}
  ```
        
1. To use native Helm, set the `useHelmInstall` flag to `true`. Native Helm is recommended for greater Helm support. To use Replicated Helm, use the default value `false`.

1. Configure the HelmChart custom resource. For more information, see [HelmChart](/reference/custom-resource-helmchart) in _Custom Resources_.

1. Repeat these steps for each Helm chart in your release.

## Additional Resources

For more information and examples of HelmChart configuration options, see:

- [Defining Installation Order for Native Helm Charts](helm-native-helm-install-order)
- [Including Optional Charts](helm-optional-charts)
- [Configuring Optional Value Keys](helm-optional-value-keys)