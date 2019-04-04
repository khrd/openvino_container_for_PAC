# This repo is ...?

This repository shows how to create OpenVINO container for Intel PAC.

## Test environments
### Devices / Machine 

- DELL R640 (k8s worker node)
- Intel PAC

### Software

- Docker (version 18.09.0, build 4d60db4)
- OpenVINO (computer_vision_sdk_fpga_2018.4.420)
- Intel k8s plugin

## Procedure

### Get openvino download link.

https://software.intel.com/en-us/openvino-toolkit/choose-download/free-download-linux-fpga
You need to register Intel account.

### Replace `ARG DOWNLOAD_LINK=` line of dockerfile

```
FROM ubuntu:16.04
ENV CL_CONTEXT_COMPILER_MODE_INTELFPGA=3
ENV DLA_AOCX=/opt/intel/computer_vision_sdk/bitstreams/a10_dcp_bitstreams/4-0_RC_FP11_Generic.aocx
ENV PATH=/opt/altera/aocl-pro-rte/aclrte-linux64/bin/:$PATH
ARG DOWNLOAD_LINK=http://registrationcenter-download.intel.com/akdlm/[★ Please input your code!★ ]/l_openvino_toolkit_p_2018.4.420.tgz
~~~
```

### Build OpenVINO container for FPGA
```
$ cd docker
$ sudo docker build -t openvino_fpga:1.0 .
```

### Check docker images
```
$ sudo docker images
REPOSITORY      TAG       IMAGE ID            CREATED             SIZE
openvino_fpga   1.0       5e744c947363        6 weeks ago         2.34GB
```

### Run docker container (test)
```
docker run --rm -it \
--mount type=bind,source=/home/user/inteldevstack/a10_gx_pac_ias_1_1_pv/,destination=/opt/a10_gx_pac_ias_1_1_pv/ \
--mount type=bind,source=/home/user/inteldevstack/a10_gx_pac_ias_1_1_pv/opencl/opencl_bsp/linux64/lib/,destination=/home/user/inteldevstack/a10_gx_pac_ias_1_1_pv/opencl/opencl_bsp/linux64/lib/ \
--mount type=bind,source=/opt/intel/computer_vision_sdk/bitstreams/a10_dcp_bitstreams/,destination=/opt/intel/computer_vision_sdk/bitstreams/a10_dcp_bitstreams/ \
--mount type=bind,source=/opt/intel/fpga-sw/opae/lib/,destination=/opt/intel/fpga-sw/opae/lib/ \
--mount type=bind,source=/home/user/inteldevstack/intelFPGA_pro,destination=/opt/intel/intelFPGA_pro \
--mount type=bind,source=/opt/altera,destination=/opt/altera \
--mount type=bind,source=/etc/OpenCL/vendors,destination=/etc/OpenCL/vendors \
--mount type=bind,source=/opt/Intel/OpenCL/Boards,destination=/opt/Intel/OpenCL/Boards \
--mount type=bind,source=/home/user/inteldevstack/a10_gx_pac_ias_1_1_pv/opencl,destination=/home/user/inteldevstack/a10_gx_pac_ias_1_1_pv/opencl \
--mount type=bind,source=/opt/intel/computer_vision_sdk_2018.4.420/bitstreams/a10_dcp_bitstreams,destination=/opt/intel/computer_vision_sdk_2018.4.420/bitstreams/a10_dcp_bitstreams \
--device /dev/intel-fpga-fme.0:/dev/intel-fpga-fme.0 \
--device /dev/intel-fpga-port.0:/dev/intel-fpga-port.0 \
--cap-add=IPC_LOCK \
openvino_fpga:1.0
```

### Test OpenVINO inference in Docker container

In the docker container:

```
# cd /opt/intel/computer_vision_sdk_2018.4.420/deployment_tools/demo
# ./demo_squeezenet_download_convert_run.sh
```

```
# cd /root/inference_engine_samples/intel64/Release

# ./classification_sample \
    -i /opt/intel/computer_vision_sdk/deployment_tools/demo/car.png \
    -m ~/openvino_models/ir/squeezenet1.1/FP32/squeezenet1.1.xml \
    -d HETERO:FPGA,CPU
```

```
exit
```

### Save container image

```
$ sudo docker commit 927ac6bf2ea5 openvino_fpga:1.1
```

### Test OpenVINO inference as k8s job

```
$ kubectl apply -f kubernetes/openvino-job.yaml
```
