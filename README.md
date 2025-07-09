
# ZeroDCE++ FPGA

Porting image enhancement model ZeroDCE++ on DPU accelerated FPGA devices such as Kria KV260, Zynq Ultrascale boards, Versal boards with help of Vitis-AI.

## Introduction:

This project is inspired from [FPGA-OwlsEye](https://github.com/Gaurav-Shah05/FPGA-OwlsEye.git).

In FPGA-Owlseye, the ZeroDCE model was originally mapped to ARM processor of Zynq Ultrascale zcu104 board. This project focuses on mapping the ZeroDCE++ model to DPU (Deep Learning Processing Units) of the zcu104 board. The reason why ZeroDCE++ is chosen instead of ZeroDCE is because the difference between number of params while providing same results.

   			ZeroDCE    - ~80k params
      		ZerodDCE++ - 10561 params


Some key changes are made in the original design of the model to map it to DPU. It's mentioned






## Modified ZeroDCE++

Original [ZeroDCE++](https://github.com/Li-Chongyi/Zero-DCE_extension.git) model used tanh activation in final convolution layer and power operations in LE curve estimation formula. These two operations are not supported by DPU. So, zeroDCE++ was modified with approximating the unsupported operations with supported operations as follows,

        tanh = 2 * hardSigmoid(x) - 1

        torch.pow(x, 2) = torch.mul(x, x)

# Prerequisites:

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





