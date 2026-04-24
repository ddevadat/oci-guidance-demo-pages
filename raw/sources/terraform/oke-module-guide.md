---
name: oke terraform module guide
description: "Provides the oke terraform module details - oracle-terraform-modules/oke/oci"
metadata:
  type: guide
  tags:
    - oci
    - oke
    - terraform
  languages:
    - terraform
  updated_at: 2026-04-24
---

# OCI Terraform OKE Module Examples

## Golden Rule

- Always create a provider alias oci.home and use it in the module 

```hcl

provider "oci" {
  alias  = home
  region = var.home_region
}

module "oke" {
  source  = "oracle-terraform-modules/oke/oci"
  version = "5.4.1"
  tenancy_id = var.tenancy_ocid
  ...

  providers = {
    oci.home = oci.home
  }

}
```

- If the request is to only create bastion, dont create operator(create_bastion=true and create_operator=false)

- If the request is to create operator, you have to create bastion also.(create_bastion=true and create_operator=true)

- Dont create bastion or operator node if not explicitly requested (create_bastion=false and create_operator=false)

- Home region can be fetched using tenancy_ocid
```hcl
data "oci_identity_tenancy" "tenant_details" {
  tenancy_id = var.tenancy_ocid
}

data "oci_identity_regions" "home_region" {
  filter {
    name   = "key"
    values = [data.oci_identity_tenancy.tenant_details.home_region_key]
  }
}

locals {
  home_region        = lookup(data.oci_identity_regions.home_region.regions[0], "name")
}
```



## Module Inputs Variables

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| agent_config | Default agent_config for self-managed worker pools created with mode: `instance`, `instance-pool`, or `cluster-network`. | `object({ are_all_plugins_disabled = bool, is_management_disabled = bool, is_monitoring_disabled = bool, plugins_config = map(string) })` | `null` | no |
| allow_bastion_cluster_access | Whether to allow access to the Kubernetes cluster endpoint from the bastion host. | `bool` | `false` | no |
| allow_node_port_access | Whether to allow access from worker NodePort range to load balancers. | `bool` | `false` | no |
| allow_pod_internet_access | Allow pods to egress to internet. Ignored when `cni_type != 'npn'`. | `bool` | `true` | no |
| allow_rules_cp | A map of additional rules to allow traffic for the OKE control plane. | `any` | `{}` | no |
| allow_rules_internal_lb | A map of additional rules to allow incoming traffic for internal load balancers. | `any` | `{}` | no |
| allow_rules_pods | A map of additional rules to allow traffic for the pods. | `any` | `{}` | no |
| allow_rules_public_lb | A map of additional rules to allow incoming traffic for public load balancers. | `any` | `{}` | no |
| allow_rules_workers | A map of additional rules to allow traffic for the workers. | `any` | `{}` | no |
| allow_short_container_image_names | Whether to allow short container image names for K8s version >= 1.34.0. | `bool` | `false` | no |
| allow_worker_internet_access | Allow worker nodes to egress to internet. Required if container images are in a registry other than OCIR. | `bool` | `true` | no |
| allow_worker_ssh_access | Whether to allow SSH access to worker nodes. | `bool` | `false` | no |
| api_fingerprint | Fingerprint of the API private key to use with OCI API. | `string` | `null` | no |
| api_private_key | The contents of the private key file to use with OCI API, optionally base64-encoded. | `string` | `null` | no |
| api_private_key_password | The corresponding private key password to use with the api private key if it is encrypted. | `string` | `null` | no |
| api_private_key_path | The path to the OCI API private key. | `string` | `null` | no |
| argocd_helm_values | Map of individual Helm chart values. | `map(string)` | `{}` | no |
| argocd_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| argocd_helm_version | Version of the Helm chart to install. | `string` | `"8.1.2"` | no |
| argocd_install | Whether to deploy the Argo CD Helm chart. | `bool` | `false` | no |
| argocd_namespace | Kubernetes namespace for deployed resources. | `string` | `"argocd"` | no |
| assign_dns | Whether to assign DNS records to created instances or disable DNS resolution of hostnames in the VCN. | `bool` | `true` | no |
| assign_public_ip_to_control_plane | Whether to assign a public IP address to the API endpoint for public access. | `bool` | `false` | no |
| await_node_readiness | Optionally block completion of Terraform apply until one/all worker nodes become ready. | `string` | `"none"` | no |
| backend_nsg_ids | Additional NSG IDs for LB backends. | `set(string)` | `[]` | no |
| bastion_allowed_cidrs | List of CIDR blocks to allow SSH access to the bastion host. | `list(string)` | `[]` | no |
| bastion_availability_domain | Availability domain for bastion placement. | `string` | `null` | no |
| bastion_await_cloudinit | Whether to block until successful connection to bastion and completion of cloud-init. | `bool` | `true` | no |
| bastion_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| bastion_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| bastion_image_id | Image ID for created bastion instance. | `string` | `null` | no |
| bastion_image_os | Bastion image operating system name when `bastion_image_type = 'platform'`. | `string` | `"Oracle Autonomous Linux"` | no |
| bastion_image_os_version | Bastion image operating system version when `bastion_image_type = 'platform'`. | `string` | `"8"` | no |
| bastion_image_type | Whether to use a platform or custom image for the created bastion instance. | `string` | `"platform"` | no |
| bastion_is_public | Whether to create allocate a public IP and subnet for the created bastion host. | `bool` | `true` | no |
| bastion_legacy_imds_endpoints_disabled | Whether to disable requests to the IMDSv1 endpoint and only allow requests to the IMDSv2 endpoint for the bastion instance. | `bool` | `true` | no |
| bastion_nsg_ids | Additional NSG IDs for bastion security. | `list(string)` | `[]` | no |
| bastion_public_ip | IP address of an existing bastion host, if `create_bastion = false`. | `string` | `null` | no |
| bastion_shape | Shape of bastion instance. | `map(any)` | `{ "baseline_ocpu_utilization": 100, "boot_volume_size": 50, "memory": 4, "ocpus": 1, "shape": "VM.Standard.E4.Flex" }` | no |
| bastion_upgrade | Whether to upgrade bastion packages after provisioning. | `bool` | `false` | no |
| bastion_user | User for SSH access through bastion host. | `string` | `"opc"` | no |
| bastion_volume_kms_key_id | OCID of the OCI KMS key for the bastion host boot volume. | `string` | `null` | no |
| cilium_helm_values | Map of individual Helm chart values. | `any` | `{}` | no |
| cilium_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| cilium_helm_version | Version of the Helm chart to install. | `string` | `"1.16.3"` | no |
| cilium_install | Whether to deploy the Cilium Helm chart. | `bool` | `false` | no |
| cilium_namespace | Kubernetes namespace for deployed resources. | `string` | `"kube-system"` | no |
| cilium_reapply | Whether to force reapply of the chart when no changes are detected. | `bool` | `false` | no |
| cluster_addons | Map with cluster addons that should be enabled. | `any` | `{}` | no |
| cluster_addons_to_remove | Map with cluster addons not created by Terraform that should be removed. | `any` | `{}` | no |
| cluster_autoscaler_helm_values | Map of individual Helm chart values. | `map(string)` | `{}` | no |
| cluster_autoscaler_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| cluster_autoscaler_helm_version | Version of the Helm chart to install. | `string` | `"9.24.0"` | no |
| cluster_autoscaler_install | Whether to deploy the Kubernetes Cluster Autoscaler Helm chart. | `bool` | `false` | no |
| cluster_autoscaler_namespace | Kubernetes namespace for deployed resources. | `string` | `"kube-system"` | no |
| cluster_ca_cert | Base64+PEM-encoded cluster CA certificate for unmanaged instance pools. | `string` | `null` | no |
| cluster_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| cluster_dns | Cluster DNS resolver IP address. | `string` | `null` | no |
| cluster_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| cluster_id | Existing OKE cluster OCID when `create_cluster = false`. | `string` | `null` | no |
| cluster_kms_key_id | OCI KMS key used as the master encryption key for Kubernetes secrets encryption. | `string` | `""` | no |
| cluster_name | Name of the OKE cluster. | `string` | `"oke"` | no |
| cluster_type | The cluster type. | `string` | `"basic"` | no |
| cni_type | The CNI for the cluster: `flannel` or `npn`. | `string` | `"flannel"` | no |
| compartment_id | Compartment ID where resources will be created. | `string` | `null` | no |
| compartment_ocid | Compartment OCID automatically populated by Resource Manager. | `string` | `null` | no |
| config_file_profile | Profile within the OCI config file to use. | `string` | `"DEFAULT"` | no |
| control_plane_allowed_cidrs | List of CIDR blocks from which the control plane can be accessed. | `list(string)` | `[]` | no |
| control_plane_is_public | Whether the Kubernetes control plane endpoint should be allocated a public IP address. | `bool` | `false` | no |
| control_plane_nsg_ids | Additional NSG IDs for the cluster endpoint. | `set(string)` | `[]` | no |
| create_bastion | Whether to create a bastion host. | `bool` | `true` | no |
| create_cluster | Whether to create the OKE cluster and dependent resources. | `bool` | `true` | no |
| create_drg | Whether to create a Dynamic Routing Gateway and attach it to the VCN. | `bool` | `false` | no |
| create_iam_autoscaler_policy | Whether to create IAM dynamic group and policy rules for Cluster Autoscaler management. | `string` | `"auto"` | no |
| create_iam_defined_tags | Whether to create defined tags used for IAM policy and tracking. | `bool` | `false` | no |
| create_iam_kms_policy | Whether to create IAM dynamic group and policy rules for cluster autoscaler. | `string` | `"auto"` | no |
| create_iam_operator_policy | Whether to create IAM dynamic group and policy rules for operator access to the OKE control plane. | `string` | `"auto"` | no |
| create_iam_resources | Whether to create IAM dynamic groups, policies, and tags. | `bool` | `false` | no |
| create_iam_tag_namespace | Whether to create a namespace for defined tags used for IAM policy and tracking. | `bool` | `false` | no |
| create_iam_worker_policy | Whether to create an IAM dynamic group and policy rules for self-managed worker nodes. | `string` | `"auto"` | no |
| create_operator | Whether to create an operator server in a private subnet. | `bool` | `true` | no |
| create_service_account | Whether to create a service account or not. | `bool` | `false` | no |
| create_vcn | Whether to create a Virtual Cloud Network. | `bool` | `true` | no |
| current_user_ocid | A user OCID automatically populated by Resource Manager. | `string` | `null` | no |
| dcgm_exporter_helm_values | Map of individual Helm chart values. | `map(string)` | `{}` | no |
| dcgm_exporter_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| dcgm_exporter_helm_version | Version of the Helm chart to install. | `string` | `"3.1.5"` | no |
| dcgm_exporter_install | Whether to deploy the DCGM exporter Helm chart. | `bool` | `false` | no |
| dcgm_exporter_namespace | Kubernetes namespace for deployed resources. | `string` | `"metrics"` | no |
| dcgm_exporter_reapply | Whether to force reapply of the Helm chart when no changes are detected. | `bool` | `false` | no |
| defined_tags | Defined tags to be applied to created resources. | `any` | `{ "bastion": {}, "cluster": {}, "iam": {}, "network": {}, "operator": {}, "persistent_volume": {}, "service_lb": {}, "workers": {} }` | no |
| drg_attachments | DRG attachment configurations. | `any` | `{}` | no |
| drg_compartment_id | Compartment for the DRG resource. Can be used to override `network_compartment_id`. | `string` | `null` | no |
| drg_display_name | Name of the created Dynamic Routing Gateway. | `string` | `null` | no |
| drg_id | ID of an externally created Dynamic Routing Gateway to be attached to the VCN. | `string` | `null` | no |
| enable_ipv6 | Whether to create a dual-stack cluster. | `bool` | `false` | no |
| enable_waf | Whether to enable WAF monitoring of load balancers. | `bool` | `false` | no |
| freeform_tags | Freeform tags to be applied to created resources. | `any` | `{ "bastion": {}, "cluster": {}, "iam": {}, "network": {}, "operator": {}, "persistent_volume": {}, "service_lb": {}, "workers": {} }` | no |
| gatekeeper_helm_values | Map of individual Helm chart values. | `map(string)` | `{}` | no |
| gatekeeper_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| gatekeeper_helm_version | Version of the Helm chart to install. | `string` | `"3.11.0"` | no |
| gatekeeper_install | Whether to deploy the Gatekeeper Helm chart. | `bool` | `false` | no |
| gatekeeper_namespace | Kubernetes namespace for deployed resources. | `string` | `"kube-system"` | no |
| home_region | The tenancy's home region. Required to perform identity operations. | `string` | `null` | no |
| iam_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| iam_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| ig_route_table_id | Optional ID of existing internet gateway route table in VCN. | `string` | `null` | no |
| igw_ngw_mixed_route_id | Optional ID of the existing route table in VCN using IGW for IPv6 and NGW for IPv4. | `string` | `null` | no |
| image_signing_keys | List of KMS key IDs used by worker nodes to verify signed images. | `set(string)` | `[]` | no |
| internet_gateway_id | Optional ID of existing Internet gateway in VCN. | `string` | `null` | no |
| internet_gateway_route_rules | List of routing rules to add to Internet Gateway Route Table. | `list(map(string))` | `null` | no |
| kubeproxy_mode | The mode in which to run kube-proxy when unspecified on a pool. | `string` | `"iptables"` | no |
| kubernetes_version | Version of Kubernetes to use when provisioning OKE or upgrading an existing OKE cluster. | `string` | `"v1.34.2"` | no |
| load_balancers | The type of subnets to create for load balancers. | `string` | `"both"` | no |
| local_peering_gateways | Map of Local Peering Gateways to attach to the VCN. | `map(any)` | `null` | no |
| lockdown_default_seclist | Whether to remove all default security rules from the VCN Default Security List. | `bool` | `true` | no |
| max_pods_per_node | The default maximum number of pods to deploy per node. | `number` | `31` | no |
| metrics_server_helm_values | Map of individual Helm chart values. | `map(string)` | `{}` | no |
| metrics_server_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| metrics_server_helm_version | Version of the Helm chart to install. | `string` | `"3.8.3"` | no |
| metrics_server_install | Whether to deploy the Kubernetes Metrics Server Helm chart. | `bool` | `false` | no |
| metrics_server_namespace | Kubernetes namespace for deployed resources. | `string` | `"metrics"` | no |
| mpi_operator_deployment_url | URL path to the manifest. | `string` | `null` | no |
| mpi_operator_install | Whether to deploy the MPI Operator. | `bool` | `false` | no |
| mpi_operator_namespace | Kubernetes namespace for deployed resources. | `string` | `"default"` | no |
| mpi_operator_version | Version to install. | `string` | `"0.4.0"` | no |
| multus_daemonset_url | URL path to the Multus manifest. | `string` | `null` | no |
| multus_install | Whether to deploy Multus. | `bool` | `false` | no |
| multus_namespace | Kubernetes namespace for deployed resources. | `string` | `"network"` | no |
| multus_version | Version of Multus to install. | `string` | `"3.9.3"` | no |
| nat_gateway_id | Optional ID of existing NAT gateway in VCN. | `string` | `null` | no |
| nat_gateway_public_ip_id | OCID of reserved IP address for NAT gateway. | `string` | `null` | no |
| nat_gateway_route_rules | List of routing rules to add to NAT Gateway Route Table. | `list(map(string))` | `null` | no |
| nat_route_table_id | Optional ID of existing NAT gateway route table in VCN. | `string` | `null` | no |
| network_compartment_id | Compartment ID where network resources will be created. | `string` | `null` | no |
| network_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| network_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| nsgs | Configuration for standard network security groups. | `map(object({ create = optional(string), id = optional(string) }))` | `{ "bastion": {}, "cp": {}, "int_lb": {}, "operator": {}, "pods": {}, "pub_lb": {}, "workers": {} }` | no |
| ocir_email_address | Email address used for the Oracle Container Image Registry. | `string` | `null` | no |
| ocir_secret_id | OCI Vault secret ID for the OCIR authentication token. | `string` | `null` | no |
| ocir_secret_name | Name of the Kubernetes secret to create with the OCIR authentication token. | `string` | `"ocirsecret"` | no |
| ocir_secret_namespace | Kubernetes namespace in which to create the OCIR secret. | `string` | `"default"` | no |
| ocir_username | Username with access to the OCI Vault secret for OCIR access. | `string` | `null` | no |
| oidc_discovery_enabled | Whether the cluster has OIDC Discovery enabled. | `bool` | `false` | no |
| oidc_token_auth_enabled | Whether the cluster has OIDC Auth Config enabled. | `bool` | `false` | no |
| oidc_token_authentication_config | Properties that configure OIDC token authentication in kube-apiserver. | `any` | `{}` | no |
| oke_ip_families | Override the `ip_families` attribute for the OKE cluster. | `list(string)` | `[]` | no |
| operator_availability_domain | Availability domain for FSS placement. | `string` | `null` | no |
| operator_await_cloudinit | Whether to block until successful connection to operator and completion of cloud-init. | `bool` | `true` | no |
| operator_cloud_init | List of maps containing cloud init MIME part configuration for operator host. | `list(map(string))` | `[]` | no |
| operator_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| operator_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| operator_image_id | Image ID for created operator instance. | `string` | `null` | no |
| operator_image_os | Operator image operating system name when `operator_image_type = 'platform'`. | `string` | `"Oracle Linux"` | no |
| operator_image_os_version | Operator image operating system version when `operator_image_type = 'platform'`. | `string` | `"8"` | no |
| operator_image_type | Whether to use a platform or custom image for the created operator instance. | `string` | `"platform"` | no |
| operator_install_helm | Whether to install Helm on the created operator host. | `bool` | `true` | no |
| operator_install_helm_from_repo | Whether to install Helm from the repo on the created operator host. | `bool` | `false` | no |
| operator_install_istioctl | Whether to install istioctl on the created operator host. | `bool` | `false` | no |
| operator_install_k8sgpt | Whether to install k8sgpt on the created operator host. | `bool` | `false` | no |
| operator_install_k9s | Whether to install k9s on the created operator host. | `bool` | `false` | no |
| operator_install_kubectl_from_repo | Whether to install kubectl from the repo on the created operator host. | `bool` | `true` | no |
| operator_install_kubectx | Whether to install kubectx/kubens on the created operator host. | `bool` | `true` | no |
| operator_install_oci_cli_from_repo | Whether to install OCI CLI from repo on the created operator host. | `bool` | `false` | no |
| operator_install_stern | Whether to install stern on the created operator host. | `bool` | `false` | no |
| operator_legacy_imds_endpoints_disabled | Whether to disable requests to the IMDSv1 endpoint and only allow requests to the IMDSv2 endpoint for the operator instance. | `bool` | `true` | no |
| operator_nsg_ids | Optional list of NSGs that the operator will be part of. | `list(string)` | `[]` | no |
| operator_private_ip | IP address of an existing operator host. | `string` | `null` | no |
| operator_pv_transit_encryption | Whether to enable in-transit encryption for the data volume's paravirtualized attachment. | `bool` | `false` | no |
| operator_shape | Shape of the created operator instance. | `map(any)` | `{ "baseline_ocpu_utilization": 100, "boot_volume_size": 50, "memory": 4, "ocpus": 1, "shape": "VM.Standard.E4.Flex" }` | no |
| operator_upgrade | Whether to upgrade operator packages after provisioning. | `bool` | `false` | no |
| operator_user | User for SSH access to operator host. | `string` | `"opc"` | no |
| operator_volume_kms_key_id | OCID of the OCI KMS key for the operator host boot volume. | `string` | `null` | no |
| output_detail | Whether to include detailed output in state. | `bool` | `false` | no |
| persistent_volume_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| persistent_volume_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| platform_config | Default platform_config for self-managed worker pools. | `object({ type = optional(string), are_virtual_instructions_enabled = optional(bool), is_access_control_service_enabled = optional(bool), is_input_output_memory_management_unit_enabled = optional(bool), is_measured_boot_enabled = optional(bool), is_memory_encryption_enabled = optional(bool), is_secure_boot_enabled = optional(bool), is_symmetric_multi_threading_enabled = optional(bool), is_trusted_platform_module_enabled = optional(bool), numa_nodes_per_socket = optional(number), percentage_of_cores_enabled = optional(bool) })` | `null` | no |
| pod_nsg_ids | Additional NSG IDs for pod security. | `list(string)` | `[]` | no |
| pods_cidr | CIDR range used by pods. | `string` | `"10.244.0.0/16"` | no |
| preferred_load_balancer | Preferred load balancer subnets. | `string` | `"public"` | no |
| prometheus_helm_values | Map of individual Helm chart values. | `map(string)` | `{}` | no |
| prometheus_helm_values_files | Paths to local YAML files with Helm chart values. | `list(string)` | `[]` | no |
| prometheus_helm_version | Version of the Helm chart to install. | `string` | `"45.2.0"` | no |
| prometheus_install | Whether to deploy the Prometheus Helm chart. | `bool` | `false` | no |
| prometheus_namespace | Kubernetes namespace for deployed resources. | `string` | `"metrics"` | no |
| prometheus_reapply | Whether to force reapply of the Prometheus Helm chart when no changes are detected. | `bool` | `false` | no |
| rdma_cni_plugin_daemonset_url | URL path to the manifest. | `string` | `null` | no |
| rdma_cni_plugin_install | Whether to deploy the SR-IOV CNI Plugin. | `bool` | `false` | no |
| rdma_cni_plugin_namespace | Kubernetes namespace for deployed resources. | `string` | `"network"` | no |
| rdma_cni_plugin_version | Version to install. | `string` | `"master"` | no |
| region | OCI region where OKE resources will be created. | `string` | `"us-ashburn-1"` | no |
| remote_peering_connections | Map of parameters to add and optionally peer to remote peering connections. | `map(any)` | `{}` | no |
| service_accounts | Map of service accounts and associated parameters. | `map(any)` | `{ "kubeconfigsa": { "sa_cluster_role": "cluster-admin", "sa_cluster_role_binding": "kubeconfigsa-crb", "sa_name": "kubeconfigsa", "sa_namespace": "kube-system" } }` | no |
| service_lb_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| service_lb_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |
| services_cidr | CIDR range used within the cluster by Kubernetes services. | `string` | `"10.96.0.0/16"` | no |
| sriov_cni_plugin_daemonset_url | URL path to the manifest. | `string` | `null` | no |
| sriov_cni_plugin_install | Whether to deploy the SR-IOV CNI Plugin. | `bool` | `false` | no |
| sriov_cni_plugin_namespace | Kubernetes namespace for deployed resources. | `string` | `"network"` | no |
| sriov_cni_plugin_version | Version to install. | `string` | `"master"` | no |
| sriov_device_plugin_daemonset_url | URL path to the manifest. | `string` | `null` | no |
| sriov_device_plugin_install | Whether to deploy the SR-IOV Network Device Plugin. | `bool` | `false` | no |
| sriov_device_plugin_namespace | Kubernetes namespace for deployed resources. | `string` | `"network"` | no |
| sriov_device_plugin_version | Version to install. | `string` | `"master"` | no |
| ssh_private_key | Contents of the SSH private key file, optionally base64-encoded. | `string` | `null` | no |
| ssh_private_key_path | Path on the local filesystem to the SSH private key. | `string` | `null` | no |
| ssh_public_key | Contents of the SSH public key file, optionally base64-encoded. | `string` | `null` | no |
| ssh_public_key_path | Path on the local filesystem to the SSH public key. | `string` | `null` | no |
| state_id | Optional Terraform state_id from an existing deployment of the module to reuse with created resources. | `string` | `null` | no |
| subnets | Configuration for standard subnets. | `map(object({ create = optional(string), id = optional(string), newbits = optional(string), netnum = optional(string), cidr = optional(string), display_name = optional(string), dns_label = optional(string), ipv6_cidr = optional(string) }))` | `{ "bastion": { "ipv6_cidr": "8, 0", "newbits": 13 }, "cp": { "ipv6_cidr": "8, 2", "newbits": 13 }, "int_lb": { "ipv6_cidr": "8, 3", "newbits": 11 }, "operator": { "ipv6_cidr": "8, 1", "newbits": 13 }, "pods": { "ipv6_cidr": "8, 6", "newbits": 2 }, "pub_lb": { "ipv6_cidr": "8, 4", "newbits": 11 }, "workers": { "ipv6_cidr": "8, 5", "newbits": 4 } }` | no |
| tag_namespace | The tag namespace for standard OKE defined tags. | `string` | `"oke"` | no |
| tenancy_id | The tenancy ID of the OCI Cloud Account in which to create the resources. | `string` | `null` | no |
| tenancy_ocid | A tenancy OCID automatically populated by Resource Manager. | `string` | `null` | no |
| timezone | Preferred timezone for workers, operator, and bastion instances. | `string` | `"Etc/UTC"` | no |
| use_defined_tags | Whether to apply defined tags to created resources for IAM policy and tracking. | `bool` | `false` | no |
| use_signed_images | Whether to enforce the use of signed images. | `bool` | `false` | no |
| use_stateless_rules | Create NSGs with stateless rules instead of the default stateful rules. | `bool` | `false` | no |
| user_id | ID of the user that terraform will use to create the resources. | `string` | `null` | no |
| vcn_cidrs | List of IPv4 CIDR blocks the VCN will use. | `list(string)` | `["10.0.0.0/16"]` | no |
| vcn_create_internet_gateway | Whether to create an internet gateway with the VCN. | `string` | `"auto"` | no |
| vcn_create_nat_gateway | Whether to create a NAT gateway with the VCN. | `string` | `"auto"` | no |
| vcn_create_service_gateway | Whether to create a service gateway with the VCN. | `string` | `"always"` | no |
| vcn_dns_label | DNS label for the VCN. | `string` | `null` | no |
| vcn_enable_ipv6_gua | Whether to enable IPv6 GUA when IPv6 is enabled. | `bool` | `true` | no |
| vcn_id | Optional ID of existing VCN. | `string` | `null` | no |
| vcn_ipv6_ula_cidrs | IPv6 ULA CIDR blocks to be used for the VCN. | `list(string)` | `[]` | no |
| vcn_name | Display name for the created VCN. | `string` | `null` | no |
| whereabouts_daemonset_url | URL path to the manifest. | `string` | `null` | no |
| whereabouts_install | Whether to deploy the whereabouts component. | `bool` | `false` | no |
| whereabouts_namespace | Kubernetes namespace for deployed resources. | `string` | `"default"` | no |
| whereabouts_version | Version to install. | `string` | `"master"` | no |
| worker_block_volume_type | Default block volume attachment type for Instance Configurations. | `string` | `"paravirtualized"` | no |
| worker_capacity_reservation_id | ID of the Compute capacity reservation the worker node will be launched under. | `string` | `null` | no |
| worker_cloud_init | List of maps containing cloud init MIME part configuration for worker nodes. | `list(map(string))` | `[]` | no |
| worker_compartment_id | Compartment ID where worker group resources will be created. | `string` | `null` | no |
| worker_compute_clusters | Whether to create compute clusters shared by nodes across multiple worker pools. | `map(any)` | `{}` | no |
| worker_disable_default_cloud_init | Whether to disable the default OKE cloud init and only use explicitly passed cloud init. | `bool` | `false` | no |
| worker_drain_delete_local_data | Whether to accept removal of data stored locally on draining worker pools. | `bool` | `true` | no |
| worker_drain_ignore_daemonsets | Whether to ignore DaemonSet-managed Pods when draining worker pools. | `bool` | `true` | no |
| worker_drain_timeout_seconds | Length of time to wait before giving up on draining nodes in a pool. | `number` | `900` | no |
| worker_image_id | Default image for worker pools when unspecified on a pool. | `string` | `null` | no |
| worker_image_os | Default worker image operating system name when worker_image_type = `oke` or `platform`. | `string` | `"Oracle Linux"` | no |
| worker_image_os_version | Default worker image operating system version. | `string` | `"8"` | no |
| worker_image_type | Whether to use a platform, OKE, or custom image for worker nodes by default. | `string` | `"oke"` | no |
| worker_is_public | Whether to provision workers with public IPs allocated by default. | `bool` | `false` | no |
| worker_legacy_imds_endpoints_disabled | Whether to disable requests to the IMDSv1 endpoint and only allow IMDSv2. | `bool` | `false` | no |
| worker_node_labels | Default worker node labels. | `map(string)` | `{}` | no |
| worker_node_metadata | Additional worker node instance metadata. | `map(string)` | `{}` | no |
| worker_nsg_ids | Additional NSG IDs for node security. | `list(string)` | `[]` | no |
| worker_pool_mode | Default management mode for workers when unspecified on a pool. | `string` | `"node-pool"` | no |
| worker_pool_size | Default size for worker pools when unspecified on a pool. | `number` | `0` | no |
| worker_pools | Tuple of OKE worker pools where each key maps to the OCID of an OCI resource. | `any` | `{}` | no |
| worker_preemptible_config | Default preemptible Compute configuration. | `map(any)` | `{ "enable": false, "is_preserve_boot_volume": false }` | no |
| worker_pv_transit_encryption | Whether to enable in-transit encryption for the data volume's paravirtualized attachment. | `bool` | `false` | no |
| worker_shape | Default shape of the created worker instance when unspecified on a pool. | `map(any)` | `{ "boot_volume_size": 50, "boot_volume_vpus_per_gb": 10, "memory": 16, "ocpus": 2, "shape": "VM.Standard.E4.Flex" }` | no |
| worker_volume_kms_key_id | ID of the OCI KMS key for Boot Volume and Block Volume encryption by default. | `string` | `null` | no |
| workers_defined_tags | Defined tags applied to created resources. | `map(string)` | `{}` | no |
| workers_freeform_tags | Freeform tags applied to created resources. | `map(string)` | `{}` | no |

## Module Outputs

| Name | Description |
|------|-------------|
| apiserver_private_host | Private OKE cluster endpoint address |
| availability_domains | Availability domains for tenancy and region |
| bastion_id | ID of bastion instance |
| bastion_nsg_id | Network Security Group for bastion host(s) |
| bastion_public_ip | Public IP address of bastion host |
| bastion_subnet_cidr | Bastion subnet CIDR |
| bastion_subnet_id | Bastion subnet ID |
| cluster_ca_cert | OKE cluster CA certificate |
| cluster_endpoints | Endpoints for the OKE cluster |
| cluster_id | ID of the OKE cluster |
| cluster_kubeconfig | OKE kubeconfig |
| cluster_oidc_discovery_endpoint | OIDC discovery endpoint for the OKE cluster |
| control_plane_nsg_id | Network Security Group for Kubernetes control plane(s) |
| control_plane_subnet_cidr | Control plane subnet CIDR |
| control_plane_subnet_id | Control plane subnet ID |
| dynamic_group_ids | Cluster IAM dynamic group IDs |
| fss_nsg_id | Network Security Group for File Storage Service resources |
| fss_subnet_cidr | FSS subnet CIDR |
| fss_subnet_id | FSS subnet ID |
| ig_route_table_id | Internet gateway route table ID |
| int_lb_nsg_id | Network Security Group for internal load balancers |
| int_lb_subnet_cidr | Internal load balancer subnet CIDR |
| int_lb_subnet_id | Internal load balancer subnet ID |
| lpg_all_attributes | All attributes of created LPG |
| nat_route_table_id | NAT gateway route table ID |
| network_security_rules | Network security rules |
| operator_id | ID of operator instance |
| operator_nsg_id | Network Security Group for operator host(s) |
| operator_private_ip | Private IP address of operator host |
| operator_subnet_cidr | Operator subnet CIDR |
| operator_subnet_id | Operator subnet ID |
| pod_nsg_id | Network Security Group for pods |
| pod_subnet_cidr | Pod subnet CIDR |
| pod_subnet_id | Pod subnet ID |
| policy_statements | Cluster IAM policy statements |
| pub_lb_nsg_id | Network Security Group for public load balancers |
| pub_lb_subnet_cidr | Public load balancer subnet CIDR |
| pub_lb_subnet_id | Public load balancer subnet ID |
| ssh_to_bastion | SSH command for bastion host |
| ssh_to_operator | SSH command for operator host |
| state_id | Terraform state_id |
| vcn_id | VCN ID |
| worker_instances | Created worker pools (mode == `instance`) |
| worker_nsg_id | Network Security Group for worker nodes |
| worker_pool_ids | Enabled worker pool IDs |
| worker_pool_ips | Created worker instance private IPs by pool |
| worker_pools | Created worker pools (mode != `instance`) |
| worker_subnet_cidr | Worker subnet CIDR |
| worker_subnet_id | Worker subnet ID |

## Examples

### Create OKE cluster using existing network

```hcl

provider "oci" {
  alias  = home
  region = var.home_region
}

module "oke" {
  source  = "oracle-terraform-modules/oke/oci"
  version = "5.4.1"
  tenancy_id = var.tenancy_ocid
  create_vcn              = false
  home_region             = var.home_region
  vcn_id                  = "ocid1.vcn..."
  subnets = {
  operator = { id = "ocid1.subnet..." }
  cp       = { id = "ocid1.subnet..." }
  int_lb   = { id = "ocid1.subnet..." }
  pub_lb   = { id = "ocid1.subnet..." }
  workers  = { id = "ocid1.subnet..." }
  pods     = { id = "ocid1.subnet..." }
	}

nsgs = {
  bastion  = { id = "ocid1.networksecuritygroup..." }
  operator = { id = "ocid1.networksecuritygroup..." }
  cp       = { id = "ocid1.networksecuritygroup..." }
  int_lb   = { id = "ocid1.networksecuritygroup..." }
  pub_lb   = { id = "ocid1.networksecuritygroup..." }
  workers  = { id = "ocid1.networksecuritygroup..." }
  pods     = { id = "ocid1.networksecuritygroup..." }
}
  compartment_id                    = var.compartment_ocid
  kubernetes_version                = var.kubernetes_version
  output_detail                     = true
  state_id                          = local.state_id
worker_pool_mode = "node-pool"
worker_pool_size = 1
worker_pools = {
  oke-vm-standard-large = {
    description      = "OKE-managed Node Pool with OKE Oracle Linux 8 image",
    shape            = "VM.Standard.E4.Flex",
    create           = true,
    ocpus            = 8,
    memory           = 128,
    boot_volume_size = 200,
    os               = "Oracle Linux",
    os_version       = "8",
  }
  tenancy_id = var.tenancy_ocid
  providers = {
    oci.home = oci.home
  }
}
}

```

### Create OKE cluster on a new vcn

```hcl
provider "oci" {
  alias  = home
  region = var.home_region
}

module "oke" {
  source  = "oracle-terraform-modules/oke/oci"
  version = "5.4.1"
  tenancy_id = var.tenancy_ocid
  create_vcn              = true
  home_region             = var.home_region
  network_compartment_id                    = var.compartment_ocid
  compartment_id                    = var.compartment_ocid
  worker_compartment_id                    = var.compartment_ocid
  kubernetes_version                = var.kubernetes_version
  output_detail                     = true
  state_id                          = local.state_id
  worker_pool_mode = "node-pool"
  worker_pool_size = 1
  worker_pools = {
    oke-vm-standard-large = {
      description      = "OKE-managed Node Pool with OKE Oracle Linux 8 image",
      shape            = "VM.Standard.E4.Flex",
      create           = true,
      ocpus            = 8,
      memory           = 128,
      boot_volume_size = 200,
      os               = "Oracle Linux",
      os_version       = "8",
    }
  tenancy_id = var.tenancy_ocid
  providers = {
    oci.home = oci.home
  }
}
}

```

## Common Mistakes
1. Not setting providers argument 
```hcl
  providers = {
    oci.home = oci.home
  }
```

2. Fetching the wrong latest kubernetes version. To fetch the latest kubernetes version
```hcl

data "oci_containerengine_cluster_option" "oke" {
  cluster_option_id = "all"
}

locals {
k8s_latest_version      = reverse(sort(data.oci_containerengine_cluster_option.oke.kubernetes_versions))[0]
}

```
3. Home region can be fetched using tenancy_ocid
```hcl
data "oci_identity_tenancy" "tenant_details" {
  tenancy_id = var.tenancy_ocid
}

data "oci_identity_regions" "home_region" {
  filter {
    name   = "key"
    values = [data.oci_identity_tenancy.tenant_details.home_region_key]
  }
}

locals {
  home_region        = lookup(data.oci_identity_regions.home_region.regions[0], "name")
}
```


3. Home region can be fetched using tenancy_ocid
```hcl
data "oci_identity_tenancy" "tenant_details" {
  tenancy_id = var.tenancy_ocid
}

data "oci_identity_regions" "home_region" {
  filter {
    name   = "key"
    values = [data.oci_identity_tenancy.tenant_details.home_region_key]
  }
}

locals {
  home_region        = lookup(data.oci_identity_regions.home_region.regions[0], "name")
}
```

