import KubernetesTraining from "../partials/getting-started/_kubernetes-training.mdx"
import LabsIntro from "../partials/getting-started/_labs-intro.mdx"
import TutorialIntro from "../partials/getting-started/_tutorial-intro.mdx"
import RelatedTopics from "../partials/getting-started/_related-topics.mdx"
import VMRequirements from "../partials/getting-started/_vm-requirements.mdx"

# Introduction and Setup

<TutorialIntro/>

<KubernetesTraining/>

## Set Up the Environment

As part of this tutorial, you will install a sample Helm chart-based application into a Kubernetes cluster. Before you begin, do the following to set up your environment:

* Create a Kubernetes cluster. You can use any cloud provider or tool that you prefer to create a cluster, such as Google Kubernetes Engine (GKE), Amazon Web Services (AWS), or minikube.

  **Example:**

  For example, to create a cluster in GKE, run the following command in the gcloud CLI:

  ```
  gcloud container clusters create NAME --preemptible --no-enable-ip-alias
  ```
  Where `NAME` is any name for the cluster.

* Install kubectl, the Kubernetes command line tool. See [Install Tools](https://kubernetes.io/docs/tasks/tools/) in the Kubernetes documentation.
* Configure kubectl command line access to the cluster that you created. See [Command line tool (kubectl)](https://kubernetes.io/docs/reference/kubectl/) in the Kubernetes documentation.