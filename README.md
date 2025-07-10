
# ZeroDCE++ FPGA

Porting image enhancement model ZeroDCE++ on DPU accelerated FPGA devices such as Kria KV260, Zynq Ultrascale boards, Versal boards with help of Vitis-AI.

## Introduction:

This project is inspired from [FPGA-OwlsEye](https://github.com/Gaurav-Shah05/FPGA-OwlsEye.git).

In FPGA-Owlseye, the ZeroDCE model was originally mapped to ARM processor of Zynq Ultrascale zcu104 board. This project focuses on mapping the ZeroDCE++ model to DPU (Deep Learning Processing Units) of the zcu104 board.The reason why ZeroDCE++ is chosen instead of ZeroDCE is because the difference between number of params while providing same results.
    
        ZeroDCE   - ~80K params
        ZeroDCE++ - ~10K params

Some key changes are made in the original design of the model to map it to DPU.






## Modified ZeroDCE++

Original [ZeroDCE++](https://github.com/Li-Chongyi/Zero-DCE_extension.git) model used tanh activation in final convolution layer and power operations in LE curve estimation formula. These two operations are not supported by DPU. So, zeroDCE++ was modified with approximating the unsupported operations with supported operations as follows,

        tanh = 2 * hardSigmoid(x) - 1

        torch.pow(x, 2) = torch.mul(x, x)


![Tanh_approximation](https://github.com/Ashok-19/zerodce_extension_FPGA/blob/22482e414fae7766ecaf7d2c2031815155d0839c/screenshots/tanh_approx.png)

## Prerequisites:

* Ubuntu 20.04 or 22.04

* [Vitis AI - 3.0](https://xilinx.github.io/Vitis-AI/3.0/html/index.html)  
* [ZeroDCE++](https://github.com/Li-Chongyi/Zero-DCE_extension.git) , if you want to train the model from scratch
* Zynq® UltraScale+™ MPSoC ZCU104

    

## Training the model

### Dataset:

LOL-v2 Dataset along with the images provided with zeroDCE++ training data were both used to train the model for better results.

Refer the steps below to train the model.

    #clone the repo
    git clone https://github.com/Ashok-19/zerodce_extension_FPGA.git


    cd zerodce_extension_FPGA/zdce_extension_fpga/train_test_host


Train the model using,
    
    python lowlight_train.py

Default parameters for training : 
    
    batch_size = 20
    num_epochs = 200

    #refer low_light_train.py for more parameters

## Testing the model

    python low_light_test.py



# Vitis-AI workflow

Activate the docker and conda environment in Vitis AI. Then follow the steps below,

It's recommended to install jupyter-lab inside the docker environment for better experience


    # create a directory for zerodce under src/vai_quantizer/vai_q_pytorch/

    mkdir zdce_extension
    

In this directory, move the entire [compile_for_dpu](https://github.com/Ashok-19/zerodce_extension_FPGA/tree/22482e414fae7766ecaf7d2c2031815155d0839c/zdce_extension_fpga/compile_for_dpu) folder.


## Inspection

Inspect the model to confirm all the operators are mapped to DPU. If any of the operators were not mapped, either wrap them as a custom operator or approximate them using supported DPU operations.

Target -> DPUCZDX8G_ISA1_B4096 (ZCU104)

Target is specified in the code itself, no need to mention it explicitly.

    python zdce.py --mode float --inspect

By running the above command, you should get a result that says "All operations are mapped to DPU".

A dot image will be created in the /inspect folder.


## Calibration

Calibrate the model by using calib data provided in data folder.

    python zdce.py --mode calib


This calibrates the model and creates a quant_info.json


## Testing and Deployment

    python zdce.py --mode test --deploy

This step quantizes the model and creates .pt , .onnx and .xmodel files of ZeroDCE++.

Inspect the xmodel file in [netron](https://netron.app/) to see the structure and workflow of the model.


## Compiling the xmodel

Compile the xmodel for the board specific architecture using the following command,


    vai_c_xir -x <xmodel_file_path> -a /opt/vitis_ai/compiler/arch/DPUCZDX8G/ZCU104/arch.json -o <output_directory> -n <netname>


The xmodel file will be compiled to board specific architecture with the name provided as <netname> in <output_directory>


To create the graph of this compiled model, use the following command,

    xdputil xmodel <compiled_xmodel> -s <name_for_svg_file>


All the output and compiled files are already provided in the repository.


# Board Setup and working




    









