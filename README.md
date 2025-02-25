# Overview
[![Build Status](https://github.com/intel/intel-device-plugins-for-kubernetes/workflows/CI/badge.svg?branch=main)](https://github.com/intel/intel-device-plugins-for-kubernetes/actions?query=workflow%3ACI)
[![Go Report Card](https://goreportcard.com/badge/github.com/intel/intel-device-plugins-for-kubernetes)](https://goreportcard.com/report/github.com/intel/intel-device-plugins-for-kubernetes)
[![GoDoc](https://godoc.org/github.com/intel/intel-device-plugins-for-kubernetes/pkg/deviceplugin?status.svg)](https://godoc.org/github.com/intel/intel-device-plugins-for-kubernetes/pkg/deviceplugin)

This repository contains a framework for developing plugins for the Kubernetes
[device plugins framework](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/device-plugins/),
along with a number of device plugin implementations utilizing that framework.

The [v0.26 release](https://github.com/intel/intel-device-plugins-for-kubernetes/releases/latest)
is the latest feature release with its documentation available [here](https://intel.github.io/intel-device-plugins-for-kubernetes/0.26/).

Table of Contents

* [Prerequisites](#prerequisites)
* [Plugins](#plugins)
    * [GPU device plugin](#gpu-device-plugin)
    * [FPGA device plugin](#fpga-device-plugin)
    * [QAT device plugin](#qat-device-plugin)
    * [VPU device plugin](#vpu-device-plugin)
    * [SGX device plugin](#sgx-device-plugin)
    * [DSA device plugin](#dsa-device-plugin)
    * [DLB device plugin](#dlb-device-plugin)
    * [IAA device plugin](#iaa-device-plugin)
* [Device Plugins Operator](#device-plugins-operator)
* [XeLink XPU-Manager sidecar](#xelink-xpu-manager-sidecar)
* [Demos](#demos)
* [Workload Authors](#workload-authors)
* [Developers](#developers)
* [Supported Kubernetes versions](#supported-kubernetes-versions)
* [Pre-built plugin images](#pre-built-plugin-images)
* [License](#license)
* [Helm charts](#helm-charts)

## Prerequisites

Prerequisites for building and running these device plugins include:

- Appropriate hardware and drivers
- A fully configured [Kubernetes cluster]
- A working [Go environment], of at least version v1.16.

## Plugins

The below sections detail existing plugins developed using the framework.

### GPU Device Plugin

The [GPU device plugin](cmd/gpu_plugin/README.md) provides access to
discrete and integrated Intel GPU device files.

The demo subdirectory contains both a [GPU plugin demo video](demo/readme.md#intel-gpu-device-plugin-demo-video)
and an OpenCL sample deployment (`intelgpu-job.yaml`).

### FPGA Device Plugin

The [FPGA device plugin](cmd/fpga_plugin/README.md) supports FPGA passthrough for
the following hardware:

- Intel® Arria® 10 devices
- Intel® Stratix® 10 devices

The FPGA plugin comes as three parts.

- the [device plugin](#device-plugin)
- the [admission controller](#admission-controller)
- the [CRIO-O prestart hook](#cri-o-prestart-hook)

Refer to each individual sub-components documentation for more details.
Brief overviews of the sub-components are below.

The demo subdirectory contains a
[video](demo/readme.md#intel-fpga-device-plugin-demo-video) showing deployment
and use of the FPGA plugin. Sources relating to the demo can be found in the
[opae-nlb-demo](demo/opae-nlb-demo) subdirectory.

#### Device Plugin

The [FPGA device plugin](cmd/fpga_plugin/README.md) is responsible for
discovering and reporting FPGA devices to `kubelet`.

#### Admission Controller

The [FPGA admission controller webhook](cmd/fpga_admissionwebhook/README.md)
is responsible for performing mapping from user-friendly function IDs to the
Interface ID and Bitstream ID that are required for FPGA programming. It also
implements access control by namespacing FPGA configuration information.

#### CRI-O Prestart Hook

The [FPGA prestart CRI-O hook](cmd/fpga_crihook/README.md) performs discovery
of the requested FPGA function bitstream and programs FPGA devices based on the
environment variables in the workload description.

### [QAT](https://developer.intel.com/quickassist) Device Plugin

The [QAT plugin](cmd/qat_plugin/README.md) supports device plugin for Intel QAT adapters, and includes
code [showing deployment](cmd/qat_plugin/dpdkdrv) via [DPDK](https://doc.dpdk.org/guides/cryptodevs/qat.html).

The demo subdirectory includes details of both a
[QAT DPDK demo](demo/readme.md#intel-quickassist-technology-device-plugin-with-dpdk-demo-video)
and a [QAT OpenSSL demo](demo/readme.md#intel-quickassist-technology-device-plugin-openssl-demo-video).
Source for the OpenSSL demo can be found in the [relevant subdirectory](demo/openssl-qat-engine).

Details for integrating the QAT device plugin into [Kata Containers](https://katacontainers.io/)
can be found in the
[Kata Containers documentation repository](https://github.com/kata-containers/kata-containers/blob/main/docs/use-cases/using-Intel-QAT-and-kata.md).

### VPU Device Plugin

The [VPU device plugin](cmd/vpu_plugin/README.md) supports Intel VCAC-A card
(https://www.intel.com/content/dam/www/public/us/en/documents/datasheets/media-analytics-vcac-a-accelerator-card-by-celestica-datasheet.pdf)
the card has:
- 1 Intel Core i3-7100U processor
- 12 MyriadX VPUs
- 8GB DDR4 memory

The demo subdirectory includes details of a OpenVINO deployment and use of the
VPU plugin. Sources can be found in [openvino-demo](demo/ubuntu-demo-openvino).

### SGX Device Plugin

The [SGX device plugin](cmd/sgx_plugin/README.md) allows workloads to use
Intel® Software Guard Extensions (Intel® SGX) on
platforms with SGX Flexible Launch Control enabled, e.g.,:

- 3rd Generation Intel® Xeon® Scalable processor family, code-named “Ice Lake”
- Intel® Xeon® E3 processor
- Intel® NUC Kit NUC7CJYH

The Intel SGX plugin comes in three parts.

- the [device plugin](#sgx-plugin)
- the [admission webhook](#sgx-admission-webhook)
- the [SGX EPC memory registration](#sgx-epc-memory-registration)

The demo subdirectory contains a [video](demo/readme.md#intel-sgx-device-plugin-demo-video) showing the deployment
and use of the Intel SGX device plugin. Sources relating to the demo can be found in the
[sgx-sdk-demo](demo/sgx-sdk-demo) and [sgx-aesmd-demo](demo/sgx-aesmd-demo) subdirectories.

Brief overviews of the Intel SGX sub-components are given below.

<a name="sgx-plugin"></a>
#### device plugin

The [SGX device plugin](cmd/sgx_plugin/README.md) is responsible for discovering
and reporting Intel SGX device nodes to `kubelet`.

Containers requesting Intel SGX resources in the cluster should not use the
device plugins resources directly.

#### Intel SGX Admission Webhook

The Intel SGX admission webhook is responsible for performing Pod mutations based on
the `sgx.intel.com/quote-provider` pod annotation set by the user. The purpose
of the webhook is to hide the details of setting the necessary device resources
and volume mounts for using Intel SGX remote attestation in the cluster. Furthermore,
the Intel SGX admission webhook is responsible for writing a pod/sandbox
`sgx.intel.com/epc` annotation that is used by Kata Containers to dynamically
adjust its virtualized Intel SGX encrypted page cache (EPC) bank(s) size.

The Intel SGX admission webhook is available as part of
[Intel Device Plugin Operator](cmd/operator/README.md) or
as a standalone [SGX Admission webhook image](cmd/sgx_admissionwebhook/README.md).

#### Intel SGX EPC memory registration

The Intel SGX EPC memory available on each node is registered as a Kubernetes extended resource using
node-feature-discovery (NFD). A custom NFD source hook is installed as part of
[SGX device plugin](cmd/sgx_plugin/README.md)
operator deployment and NFD is configured to register the Intel SGX EPC memory
extended resource reported by the hook.

Containers requesting Intel SGX EPC resources in the cluster use
`sgx.intel.com/epc` resource which is of
type [memory](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#meaning-of-memory).

### DSA Device Plugin

The [DSA device plugin](cmd/dsa_plugin/README.md) supports acceleration using
the Intel Data Streaming accelerator(DSA).

### DLB Device Plugin

The [DLB device plugin](cmd/dlb_plugin/README.md) supports Intel Dynamic Load
Balancer accelerator(DLB).

### IAA Device Plugin

The [IAA device plugin](cmd/iaa_plugin/README.md) supports acceleration using
the Intel Analytics accelerator(IAA).

## Device Plugins Operator

To simplify the deployment of the device plugins, a unified device plugins
operator is implemented.

Currently the operator has support for the DSA, DLB, FPGA, GPU, IAA, QAT, and
Intel SGX device plugins. Each device plugin has its own custom resource
definition (CRD) and the corresponding controller that watches CRUD operations
to those custom resources.

The [Device plugins operator README](cmd/operator/README.md) gives the installation and usage details for the community operator available on [operatorhub.io](https://operatorhub.io/operator/intel-device-plugins-operator).

The [Device plugins Operator for OCP](cmd/operator/ocp_quickstart_guide/README.md) gives the installation and usage details for the operator available on [Red Hat OpenShift Container Platform](https://catalog.redhat.com/software/operators/detail/61e9f2d7b9cdd99018fc5736).

## XeLink XPU-Manager Sidecar

To support interconnected GPUs in Kubernetes, XeLink sidecar is needed.

The [XeLink XPU-Manager sidecar README](cmd/xpumanager_sidecar/README.md) gives information how the sidecar functions and how to use it.

## Demos

The [demo subdirectory](demo/readme.md) contains a number of demonstrations for
a variety of the available plugins.

## Workload Authors

For workloads to get accesss to devices managed by the plugins, the
`Pod` spec must specify the hardware resources needed:

```
spec:
  containers:
    - name: demo-container
      image: <registry>/<image>:<version>
      resources:
        limits:
          <device namespace>/<resource>: X
```

The summary of resources available via plugins in this repository is given in the list below.

**Device Namespace : Registered Resource(s)**
 * `dlb.intel.com`  : `pf` or `vf`
   * [dlb-libdlb-demo-pod.yaml](demo/dlb-libdlb-demo-pod.yaml)
 * `dsa.intel.com`  : `wq-user-[shared or dedicated]`
   * [dsa-accel-config-demo-pod.yaml](demo/dsa-accel-config-demo-pod.yaml)
 * `fpga.intel.com` : custom, see [mappings](cmd/fpga_admissionwebhook/README.md#mappings)
   * [intelfpga-job.yaml](demo/intelfpga-job.yaml)
 * `gpu.intel.com`  : `i915`
   * [intelgpu-job.yaml](demo/intelgpu-job.yaml)
 * `iaa.intel.com`  : `wq-user-[shared or dedicated]`
   * [iaa-qpl-demo-pod.yaml](demo/iaa-qpl-demo-pod.yaml)
 * `qat.intel.com`  : `generic` or `cy`/`dc`
   * [crypto-perf-dpdk-pod-requesting-qat.yaml](deployments/qat_dpdk_app/base/crypto-perf-dpdk-pod-requesting-qat.yaml)
 * `sgx.intel.com`  : `epc`
   * [intelsgx-job.yaml](deployments/sgx_enclave_apps/base/intelsgx-job.yaml)
 * `vpu.intel.com`  : `hddl`
   * [intelvpu-job.yaml](demo/intelvpu-job.yaml)

## Developers

For information on how to develop a new plugin using the framework or work on development task in
this repository, see the [Developers Guide](DEVEL.md).

## Supported Kubernetes Versions

Releases are made under the github [releases area](https://github.com/intel/intel-device-plugins-for-kubernetes/releases). Supported releases and
matching Kubernetes versions are listed below:

| Branch            | Kubernetes branch/version      | Status      |
|:------------------|:-------------------------------|:------------|
| release-0.26      | Kubernetes 1.26 branch v1.26.x | supported   |
| release-0.25      | Kubernetes 1.25 branch v1.25.x | supported   |
| release-0.24      | Kubernetes 1.24 branch v1.24.x | supported   |
| release-0.23      | Kubernetes 1.23 branch v1.23.x | unsupported |
| release-0.22      | Kubernetes 1.22 branch v1.22.x | unsupported |
| release-0.21      | Kubernetes 1.21 branch v1.21.x | unsupported |
| release-0.20      | Kubernetes 1.20 branch v1.20.x | unsupported |
| release-0.19      | Kubernetes 1.19 branch v1.19.x | unsupported |
| release-0.18      | Kubernetes 1.18 branch v1.18.x | unsupported |
| release-0.17      | Kubernetes 1.17 branch v1.17.x | unsupported |
| release-0.15      | Kubernetes 1.15 branch v1.15.x | unsupported |
| release-0.11      | Kubernetes 1.11 branch v1.11.x | unsupported |

[Go environment]: https://golang.org/doc/install
[Kubernetes cluster]: https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/

## Pre-built plugin images

Pre-built images of the plugins are available on the Docker hub. These images
are automatically built and uploaded to the hub from the latest main branch of
this repository.

Release tagged images of the components are also available on the Docker hub,
tagged with their release version numbers in the format x.y.z, corresponding to
the branches and releases in this repository.

**Note:** the default deployment files and operators are configured with
[imagePullPolicy](https://kubernetes.io/docs/concepts/containers/images/#updating-images)
```IfNotPresent``` and can be changed with ```scripts/set-image-pull-policy.sh```.

## License

All of the source code required to build intel-device-plugins-for-kubernetes
is available under Open Source licenses. The source code files identify external Go
modules used. Binaries are distributed as container images on
DockerHub*. Those images contain license texts and source code under `/licenses`.

### Helm Charts

Device Plugins Helm Charts are located in Intel Helm Charts repository [Intel Helm Charts](https://github.com/intel/helm-charts). This is another way of distributing Kubernetes resources of the device plugins framework.

To add repo:
```
helm repo add intel https://intel.github.io/helm-charts
```
