# Component Configuration

In the previous lab, you established the foundational platform using the Meta-Operator. In this lab, you will dive deeper into the management of the service layer. You will learn how to dynamically activate additional components, manage their lifecycle via the `DataScienceCluster` (DSC) resource, and apply technical troubleshooting logic to identify reconciliation issues.

## Navigating the DataScienceCluster (DSC)

The DSC resource is the "source of truth" for which AI services are active.

1. Log in to the OpenShift Web Console as a `cluster-admin`.
2. Navigate to **Ecosystem** → **Installed Operators** → **Red Hat OpenShift AI** .
3. Click on  **Red Hat OpenShift AI** .
4. Click the **Data Science Cluster** tab.
5. Click the name of your instance (e.g., `default-dsc`).
6. Select the **YAML** tab. This is where we will perform our configurations.

## Enabling Data Science Pipelines

Data Science Pipelines represent the core framework for driving automated MLOps practices. Integrated within OpenShift AI, these tooling components allow users to streamline end-to-end model training, validation, building, deployment, and ongoing monitoring.

To provision this feature in OpenShift AI, update the existing DataScienceCluster instance. Please execute the following sequence to complete the pipeline setup:

First, log into the OpenShift AI dashboard and verify active settings under the `Develop & Train` section.

![openshift-ai-operator.png](images/rhoai-ai-pipelines.png)

Once verified, proceed to provision and deploy the `AI Pipelines` infrastructure:

1. Open the OpenShift Web Console and locate the `DataScienceCluster` manifest and go inside the `YAML` editor view.
2. Within the configuration block, locate the `aipipelines` spec attribute.
3. Update the corresponding `managementState` value from `Removed` to `Managed`.
4. Confirm the YAML definitions align with the given layout:

   ```yaml
    aipipelines:
      managementState: Managed
   ```
5. Select `Save` to apply modifications.
6. Deployment Verification: Inspect `Workloads` -> `Pods` inside the target `redhat-ods-applications` namespace.

   * Verify that the target `data-science-pipelines-operator-controller-manager` instances spin up successfully and reach a ready `Running` status.

![openshift-ai-operator.png](images/rhoai-pipeline-operator-pod.png)

Return to the OpenShift AI dashboard to review the updated `Develop & Train` view. You should now observe the active `Pipelines` section, enabling teams to build and execute workspace pipelines directly.

![openshift-ai-operator.png](images/rhoai-ai-pipelines-ok.png)

## Activating the Model Registry

The Model Registry is a vital part of the MLOps workflow, acting as a central repository for model metadata, versioning, and tracking.

To activate the Model Registry and transition it to a functional state, perform the following steps:

1. In the YAML editor, search for the `modelregistry` component.
2. Change the `managementState` from `Removed` to `Managed`.
3. Ensure the configuration matches the following structure:

   ```yaml
   modelregistry:
     managementState: Managed
     registriesNamespace: rhoai-model-registries

   ```
4. Click  **Save** .
5. **Verification:** * Navigate to **Workloads** → **Pods** in the `rhoai-model-registries` namespace (or your configured application namespace).

   * Confirm that the `model-registry-service` pods initialize and eventually reach a **Running** state

![openshift-ai-operator.png](images/v35-rhoai-model-registry.png)

## Lifecycle Management (Resource Removal)

Infrastructure efficiency often requires removing unused services. In this task, you will disable a component to observe how the Operator automatically reclaims resources and executes its cleanup logic:

First, log into the OpenShift Console and verify active `trustyai` component pods under the `redhat-ods-applications` namespace.

![openshift-ai-operator.png](images/rhoai-trustyai.png)

Once verified, proceed to decommission the `trustyai` component:

1. Open the OpenShift Web Console and locate the `DataScienceCluster` manifest and go inside the `YAML` editor view.
2. In the YAML editor for `default-dsc`, locate the `trustyai` component.
3. Change the `managementState` from `Managed` to `Removed`.
4. Click  **Save** .
5. **Verification:** * Navigate to **Workloads** → **Pods** in the `redhat-ods-applications` project.
   * Observe that the `trustyai-service` pods are being terminated and removed.

![openshift-ai-operator.png](images/v35-rhoai-trusty-removed.png)

## Troubleshooting Logic

When a configuration change results in a "Progressing" state that never reaches "Ready," you must apply the following troubleshooting logic.

### Check the Resource Status

Run the following command to see if the Operator has reported specific errors in the DSC status:

```bash
oc get datasciencecluster default-dsc -o yaml

```

Look for the `status.conditions` section. A failed component will typically report a `ReconcileError` or `Ready: False` with a descriptive message.

### Inspect the Meta-Operator Logs

If the status is ambiguous, the Operator logs are the definitive source for debugging:

1. Identify the Operator pod:

   ```bash
   oc get pods -n redhat-ods-operator

   ```
2. Stream the logs to find reconciliation failures:

   ```bash
   oc logs -f deployment/rhods-operator -n redhat-ods-operator

   ```
3. **Key Terms to Search for:** `error`, `failed to create`, `permission denied`, or `retry`.

**Congratulations!** You now have the practical skills to manage the AI platform's component lifecycle and troubleshoot configuration mismatches.
