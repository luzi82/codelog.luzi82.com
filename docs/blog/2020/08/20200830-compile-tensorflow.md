# Compile Tensorflow

我的電腦 spec：

- Debian 10.5
- NVIDIA GeForce GTX 660

Tensorflow 對 NVIDIA CUDA 預設最低要求是 3.5，但 GTX 660 只是 CUDA 3.0。要支援 3.0，就必需手動 compile。

現時2010年8月30日，Tensorflow 最新版本為 2.3。而 Google Cloud Platform 則支援到 2.2。我選擇用 2.2。

## 步驟

安裝項目
- NVIDIA CUDA Toolkit 10.2
- NVIDIA cuDNN v7.6.5
- Bazel 2.0.0
- Tensorflow 2.2.0

下載檔案：

- [NVIDIA CUDA Toolkit 10.2](https://developer.nvidia.com/cuda-toolkit-archive]) 。 Linux > x86_64 > Ubuntu > 18.04 > runfile(local)，把 base installer 和 Patch 1 都一同下載。
- [NVIDIA cuDNN v7.6.5 for CUDA 10.2](https://developer.nvidia.com/rdp/cudnn-archive) 。
  - cuDNN Developer Library for Ubuntu18.04 (Deb)
  - cuDNN Runtime Library for Ubuntu18.04 (Deb)
  - cuDNN Code Samples and User Guide for Ubuntu18.04 (Deb)

如果之前有安裝過 Debian 提供的 NVIDIA driver，就必需要先移除。

    apt purge *nvidia*
    sudo init 6 # 重啟電腦，看看有沒有負作用。

安裝 NVIDIA Driver 和 CUDA Toolkit 10.2

    # 把 driver 拆出來分開執行，以便進入 dkms
    ./cuda_10.2.89_440.33.01_linux.run --extract=${PATH}/extract/

    sudo init 3 # 必需暫停 X server
    
    cd extract
    sudo ./NVIDIA-Linux-x86_64-440.33.01.run --dkms # dkms: 處理日後 Linux Kernal update 的情況
    cd ..
    sudo ./cuda_10.2.89_440.33.01_linux.run # 有安裝 UI, 不用再安裝 driver
    sudo ./cuda_10.2.1_linux.run # 有安裝 UI
    
    cat /etc/ld.so.conf.d/cuda-10-2.conf
    # 檢查有沒有 /usr/local/cuda-10.2/targets/x86_64-linux/lib
    
    sudo ldconfig
    
    sudo init 5 # 回到 X server
    sudo dkms status # 確認 driver 有上 dkms
    nvidia-smi # 確認 driver 正常運作

如果有怪事發生，請看以下檔案。

- /var/log/cuda-installer.log
- /var/log/nvidia-installer.log

安裝 cuDNN

    sudo dpkg -i libcudnn7_7.6.5.32-1+cuda10.2_amd64.deb
    sudo dpkg -i libcudnn7-dev_7.6.5.32-1+cuda10.2_amd64.deb

測試 cuDNN 是否正常運作

    sudo dpkg -i libcudnn7-doc_7.6.5.32-1+cuda10.2_amd64.deb
    cp -R /usr/src/cudnn_samples_v7/ cudnn_samples_v7
    cd cudnn_samples_v7/mnistCUDNN
    make
    ./mnistCUDNN

安裝 bazel

    # https://docs.bazel.build/versions/master/install-ubuntu.html
    sudo apt install curl gnupg
    curl -fsSL https://bazel.build/bazel-release.pub.gpg | gpg --dearmor > bazel.gpg
    sudo mv bazel.gpg /etc/apt/trusted.gpg.d/
    echo "deb [arch=amd64] https://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list

    sudo apt update && sudo apt install bazel-2.0.0
    sudo ln -s /usr/bin/bazel-1.0.0 /usr/bin/bazel
    bazel --version # bazel 2.0.0

Compile Tensorflow

    # https://www.tensorflow.org/install/source 英文版的說明較新
    
    pip3 install -U --user pip wheel
    pip3 install -U --user six 'numpy<1.19.0' setuptools mock 'future>=0.17.1' 'gast==0.3.3' typing_extensions # numpy 見註
    pip3 install -U --user keras_applications --no-deps
    pip3 install -U --user keras_preprocessing --no-deps
    
    git clone https://github.com/tensorflow/tensorflow.git
    
    cd tensorflow
    git checkout -b b2.2.0 v2.2.0
    
    ./configure
    # python path > /usr/bin/python3
    # CUDA > yes
    # CUDA version > 3.0 # 我電腦只能用 3.0
    
    bazel build --config=opt --config=cuda //tensorflow/tools/pip_package:build_pip_package # long process
    ./bazel-bin/tensorflow/tools/pip_package/build_pip_package output

然後得到安裝檔 output/tensorflow-2.2.0-cp37-cp37m-linux_x86_64.whl

    pip3 install --user output/tensorflow-2.2.0-cp37-cp37m-linux_x86_64.whl

測試

    python3 -c "import tensorflow;print(tensorflow.version.VERSION)" # 2.2.0
    python3 -c "import tensorflow;print(tensorflow.test.gpu_device_name())" 2> /dev/null # /device:GPU:0

## 附註

未來更新電腦及重新開機時，可能會出現 NVIDIA driver 不能載入的情況。解決方法是把 NVIDIA-Linux-x86_64-440.33.01.run 重新執行一次。

Debian 官方有提供 NVIDIA Driver 和 CUDA 的 apt repo。但 Tensorflow 的 configure 無法偵測所需檔案。

CUDA Toolkit 10.2 內付 NVIDIA Driver 440.33.01。現時 NVIDIA 最新是 450.66。我是使用 440.33.01。

若 Tensorflow 版本為 2.2.0，bazel 版本要求為絕對 2.0.0。參考 tensorflow/configure.py 內的 _TF_MIN_BAZEL_VERSION, _TF_MAX_BAZEL_VERSION 參數。

若 Tensorflow 版本為 2.2.0，numpy 版本必需為 1.19 以下。[TensorFlow#40688](https://github.com/tensorflow/tensorflow/issues/40688)

若 Tensorflow 版本為 2.1.0，bazel 版本要求是 0.27.1-0.29.1。參考 tensorflow/configure.py 內的 _TF_MIN_BAZEL_VERSION, _TF_MAX_BAZEL_VERSION 參數。Bazel repo 不提供 Bazel 0.x 的版本，只能[手動下載 bazel-0.29.1-installer-linux-x86_64.sh](https://github.com/bazelbuild/bazel/releases/tag/0.29.1)。注意不要把 sh 檔放到有非 ascii 路徑的地方，否則會不能安裝。 　

若 Tensorflow 版本為 2.1.0，可能會遇到「Unknown option '-bin2c-path'」問題。解決方法是手動把 third_party/nccl/build_defs.bzl.tpl 中的 「"--bin2c-path=%s" % bin2c.dirname,」刪除。[TensorFlow#34429](https://github.com/tensorflow/tensorflow/issues/34429)
