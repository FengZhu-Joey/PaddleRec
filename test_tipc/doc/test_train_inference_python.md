# Linux端基础训练预测功能测试

Linux端基础训练预测功能测试的主程序为`test_train_inference_python.sh`，可以测试基于Python的模型训练、评估、推理等基本功能。


## 1. 测试流程

运行环境配置请参考[文档](./install.md)的内容配置TIPC的运行环境。

### 1.1 安装依赖
- 安装PaddlePaddle >= 2.0
- 安装autolog（规范化日志输出工具）
    ```
    git clone https://github.com/LDOUBLEV/AutoLog
    cd AutoLog
    pip3 install -r requirements.txt
    python3 setup.py bdist_wheel
    pip3 install ./dist/auto_log-1.0.0-py3-none-any.whl
    cd ../
    ```


### 1.2 功能测试
先运行`prepare.sh`准备数据和模型，然后运行`test_train_inference_python.sh`进行测试，最终在```test_tipc/output```目录下生成`python_infer_*.log`格式的日志文件。


`test_train_inference_python.sh`包含5种运行模式，每种模式的运行数据不同，分别用于测试速度和精度，分别是：

- 模式1：lite_train_lite_infer，使用少量数据训练，用于快速验证训练到预测的走通流程，不验证精度和速度；
```shell
bash test_tipc/prepare.sh ./test_tipc/configs/wide_deep/train_infer_python.txt 'lite_train_lite_infer'
bash test_tipc/test_train_inference_python.sh ./test_tipc/configs/wide_deep/train_infer_python.txt 'lite_train_lite_infer'
```  

- 模式2：lite_train_whole_infer，使用少量数据训练，一定量数据预测，用于验证训练后的模型执行预测，预测速度是否合理；
```shell
bash test_tipc/prepare.sh ./test_tipc/configs/wide_deep/train_infer_python.txt  'lite_train_whole_infer'
bash test_tipc/test_train_inference_python.sh ../test_tipc/configs/wide_deep/train_infer_python.txt 'lite_train_whole_infer'
```  

- 模式3：whole_infer，不训练，全量数据预测，走通开源模型评估、动转静，检查inference model预测时间和精度;
```shell
bash test_tipc/prepare.sh ./test_tipc/configs/wide_deep/train_infer_python.txt 'whole_infer'
# 用法1:
bash test_tipc/test_train_inference_python.sh ../test_tipc/configs/wide_deep/train_infer_python.txt 'whole_infer'
# 用法2: 指定GPU卡预测，第三个传入参数为GPU卡号
bash test_tipc/test_train_inference_python.sh ./test_tipc/configs/wide_deep/train_infer_python.txt 'whole_infer' '1'
```  

- 模式4：whole_train_whole_infer，CE： 全量数据训练，全量数据预测，验证模型训练精度，预测精度，预测速度；
```shell
bash test_tipc/prepare.sh ./test_tipc/configs/wide_deep/train_infer_python.txt 'whole_train_whole_infer'
bash test_tipc/test_train_inference_python.sh ./test_tipc/configs/wide_deep/train_infer_python.txt 'whole_train_whole_infer'
```  

运行相应指令后，在`test_tipc/output`文件夹下自动会保存运行日志。如'lite_train_lite_infer'模式下，会运行训练+inference的链条，因此，在`test_tipc/output`文件夹有以下文件：
```
test_tipc/output/
|- results_python.log    # 运行指令状态的日志
|- norm_train_gpus_0_autocast_null/  # GPU 0号卡上正常训练的训练日志和模型保存文件夹
|- pact_train_gpus_0_autocast_null/  # GPU 0号卡上量化训练的训练日志和模型保存文件夹
......
|- python_infer_cpu_usemkldnn_True_threads_1_batchsize_1.log  # CPU上开启Mkldnn线程数设置为1，测试batch_size=1条件下的预测运行日志
|- python_infer_gpu_usetrt_True_precision_fp16_batchsize_1.log # GPU上开启TensorRT，测试batch_size=1的半精度预测日志
......
```

其中`results_python.log`中包含了每条指令的运行状态，如果运行成功会输出：
```
Run successfully with command - python3.7 -u tools/trainer.py -m ./models/rank/wide_deep/config_bigdata.yaml -o runner.print_interval=2 runner.use_gpu=True  runner.model_save_path=./test_tipc/output/norm_train_gpus_0_autocast_False runner.epochs=4   auto_cast=False runner.train_batch_size=50 runner.train_data_dir=../../../test_tipc/data/train
Run successfully with command - python3.7 -u tools/to_static.py -m ./models/rank/wide_deep/config_bigdata.yaml -o runner.CE=true runner.model_init_path=./test_tipc/output/norm_train_gpus_0_autocast_False/3 runner.model_save_path=./test_tipc/output/norm_train_gpus_0_autocast_False
......
```
如果运行失败，会输出：
```
Run failed with command - python3.7 -u tools/trainer.py -m ./models/rank/wide_deep/config_bigdata.yaml -o runner.print_interval=2 runner.use_gpu=True  runner.model_save_path=./test_tipc/output/norm_train_gpus_0_autocast_False runner.epochs=4   auto_cast=False runner.train_batch_size=50 runner.train_data_dir=../../../test_tipc/data/train
Run failed with command - python3.7 -u tools/to_static.py -m ./models/rank/wide_deep/config_bigdata.yaml -o runner.CE=true runner.model_init_path=./test_tipc/output/norm_train_gpus_0_autocast_False/3 runner.model_save_path=./test_tipc/output/norm_train_gpus_0_autocast_False
......
```
可以很方便的根据`results_python.log`中的内容判定哪一个指令运行错误。



## 3. 更多教程
本文档为功能测试用，更丰富的训练预测使用教程请参考：  
[模型训练](https://github.com/PaddlePaddle/PaddleRec/blob/master/doc/dygraph_mode.md)  
[基于Python预测引擎推理](https://github.com/PaddlePaddle/PaddleRec/blob/master/doc/inference.md)
