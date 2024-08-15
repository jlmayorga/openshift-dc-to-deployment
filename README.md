# OpenShift DeploymentConfig to Deployment Migration Playbook

## Overview

This Ansible playbook automates the process of converting OpenShift DeploymentConfigs to Kubernetes Deployments. It addresses the critical need to migrate workloads from the deprecated DeploymentConfig resource to the standard Kubernetes Deployment resource.

DeploymentConfigs have been deprecated since OpenShift 4.14. While they continue to function in current versions, they could be removed in a future release of OpenShift.

Key points about DeploymentConfig deprecation:

- DeploymentConfigs are already deprecated as of OpenShift 4.14.
- They will be removed entirely in a future OpenShift release (specific version not yet announced).
- Kubernetes Deployments are the recommended replacement, offering similar functionality with better compatibility.
- Migrating to Deployments ensures alignment with standard Kubernetes practices and readiness for future OpenShift versions.

## Features

- Automatically identifies and skips reserved OpenShift namespaces
- Converts DeploymentConfigs to Deployments across multiple namespaces
- Generates YAML files for the new Deployments
- Optional application of the generated Deployments to the cluster
- Adds annotations to track the migration process
- Preserves existing labels and annotations (configurable)
- Removes DeploymentConfig-specific labels and annotations

## Prerequisites

- Ansible 2.9 or higher
- `kubernetes.core` Ansible collection
- Access to the OpenShift cluster (via kubeconfig or other Kubernetes-compatible authentication method)
- Proper permissions in the OpenShift cluster to read DeploymentConfigs and create Deployments

## Installation

1. Clone this repository:
   ```
   git clone <repository-url>
   cd <repository-directory>
   ```

2. Install required Ansible collections:
   ```
   ansible-galaxy collection install kubernetes.core
   ```

3. Ensure your kubeconfig is set up correctly to access your OpenShift cluster.

## Configuration

The playbook can be configured by modifying the following variables in the `playbook.yaml` file:

- `openshift_projects`: List of OpenShift projects (namespaces) to process
- `output_dir`: Directory where the generated Deployment YAML files will be saved
- `reserved_namespaces`: List of namespaces to be excluded from processing
- `apply_changes`: Boolean to control whether to apply the generated Deployments to the cluster
- `preserve_existing_annotations`: Boolean to control whether to keep existing annotations (except DeploymentConfig-specific ones)
- `preserve_existing_labels`: Boolean to control whether to keep existing labels (except DeploymentConfig-specific ones)
- `dc_specific_annotations`: List of DeploymentConfig-specific annotations to remove
- `dc_specific_labels`: List of DeploymentConfig-specific labels to remove

Example configuration:

```yaml
vars:
  openshift_projects: []
  output_dir: "./converted_deployments"
  reserved_namespaces:
    - "default"
    - "openshift"
    - "openshift-infra"
  apply_changes: false
  preserve_existing_annotations: true
  preserve_existing_labels: true
  dc_specific_annotations:
    - "openshift.io/deployment-config.name"
    - "openshift.io/deployment-config.latest-version"
    - "openshift.io/deployment.phase"
  dc_specific_labels:
    - "openshift.io/deployment-config.name"
```

## Usage

1. Ensure you have the correct kubeconfig and context set for your OpenShift cluster.

2. Run the playbook without applying changes (dry-run mode):
   ```
   ansible-playbook playbook.yaml
   ```

3. To convert and apply the changes to the cluster:
   ```
   ansible-playbook playbook.yaml -e "apply_changes=true"
   ```

4. To specify projects to process:
   ```
   ansible-playbook playbook.yaml -e "openshift_projects=['project1', 'project2']"
   ```

## Output

The playbook will create a directory structure as follows:

```
./converted_deployments/
  ├── namespace1/
  │   ├── deployment1.yaml
  │   └── deployment2.yaml
  └── namespace2/
      ├── deployment3.yaml
      └── deployment4.yaml
```

Each generated Deployment YAML file will include annotations indicating it was created by this migration process and the timestamp of creation.

## Logging

The playbook generates a log file (`conversion_log.txt` by default) that includes details about the conversion process, any errors encountered, and a summary of the conversion.

## Warnings and Considerations

- Always run the playbook in dry-run mode first and review the generated YAML files before applying changes.
- Ensure you have backups of your DeploymentConfigs before running this playbook with `apply_changes=true`.
- This playbook performs a basic conversion. You may need to manually adjust the generated Deployments for workloads with complex configurations.
- Test thoroughly in a non-production environment before using in production.
- The playbook removes the 'deploymentconfig' label from the selector and template metadata to ensure compatibility with Deployments.

## Troubleshooting

- If you encounter permission issues, ensure your kubeconfig is correctly set up with the necessary permissions.
- For namespace-related errors, check that the specified projects exist and you have access to them.
- If DeploymentConfigs are not being processed, verify they exist in the specified namespaces.
- Check the conversion log file for detailed information about any errors or issues encountered during the migration process.
- Ensure that the Jinja2 template (`deployment.j2`) is up to date and properly formatted.

## Contributing

Contributions to improve the playbook are welcome. Please submit issues and pull requests through the project's repository.