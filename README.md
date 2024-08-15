# OpenShift DeploymentConfig to Deployment Migration Playbook

## Overview

This Ansible playbook automates the process of converting OpenShift DeploymentConfigs to Kubernetes Deployments. It addresses the critical need to migrate workloads from the deprecated DeploymentConfig resource to the standard Kubernetes Deployment resource.

DeploymentConfigs have been deprecated since OpenShift 4.14. While they continue to function in current versions, they could be removed in a future release of OpenShift.

Key points about DeploymentConfig deprecation:

- DeploymentConfigs are already deprecated as of OpenShift 4.14.
- They will be removed entirely in a future OpenShift release (specific version not yet announced).
- Kubernetes Deployments are the recommended replacement, offering similar functionality with better compatibility.
- Migrating to Deployments ensures alignment with standard Kubernetes practices and readiness for future OpenShift versions.

This playbook facilitates a smooth and efficient transition by automating the conversion process. It allows organizations to systematically migrate their workloads to Deployments, reducing the risk and effort associated with manual migrations.

## Features

- Automatically identifies and skips reserved OpenShift namespaces
- Converts DeploymentConfigs to Deployments across multiple namespaces
- Generates YAML files for the new Deployments
- Optional application of the generated Deployments to the cluster
- Adds annotations to track the migration process
- Preserves existing labels and annotations (configurable)
- Supports custom labels and annotations
- Comprehensive logging of the migration process

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
- `log_file`: Path to the log file for migration process details
- `custom_annotations`: List of custom annotations to be added to the new Deployments
- `custom_labels`: List of custom labels to be added to the new Deployments
- `preserve_existing_annotations`: Boolean to control whether to preserve existing annotations
- `preserve_existing_labels`: Boolean to control whether to preserve existing labels

Example configuration:

```yaml
vars:
  openshift_projects: []  # Leave empty to process all non-reserved namespaces
  output_dir: "./converted_deployments"
  reserved_namespaces:
    - "default"
    - "openshift"
    - "openshift-infra"
  apply_changes: false
  log_file: "conversion_log.txt"
  custom_annotations:
    - "mycompany.com/custom-annotation"
  custom_labels:
    - "mycompany.com/custom-label"
  preserve_existing_annotations: true
  preserve_existing_labels: true
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

4. To process specific projects:
   ```
   ansible-playbook playbook.yaml -e '{"openshift_projects": ["project1", "project2"]}'
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

Each generated Deployment YAML file will include annotations indicating it was created by this migration process and the timestamp of creation. The playbook will also generate a detailed log file (`conversion_log.txt` by default) with information about the migration process.

## Warnings and Considerations

- Always run the playbook in dry-run mode first and review the generated YAML files before applying changes.
- Ensure you have backups of your DeploymentConfigs before running this playbook with `apply_changes=true`.
- This playbook performs a basic conversion. You may need to manually adjust the generated Deployments for workloads with complex configurations.
- Test thoroughly in a non-production environment before using in production.
- The playbook now preserves existing labels and annotations by default. Review these settings to ensure they align with your migration strategy.

## Troubleshooting

- If you encounter permission issues, ensure your kubeconfig is correctly set up with the necessary permissions.
- For namespace-related errors, check that the specified projects exist and you have access to them.
- If DeploymentConfigs are not being processed, verify they exist in the specified namespaces.
- Check the `conversion_log.txt` file for detailed information about the migration process and any errors encountered.
- Verify that the custom annotations and labels you've specified exist in the source DeploymentConfigs.
- If you're having issues with preserved annotations or labels, try setting `preserve_existing_annotations` and `preserve_existing_labels` to `false` to see if that resolves the problem.

## Contributing

Contributions to improve the playbook are welcome. Please submit issues and pull requests through the project's repository.