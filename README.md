TensorFlow on GTX 10-series with Docker
===================

TensorFlow, GTX 10, Pascal, NVIDIA Docker, Known issues, Troubleshooting, Dockerfile download, Install guide.

----------


 이 포스트에서는 Pascal architecture의 GPU(GTX 10-series) 장치와 Docker 환경에서 TensorFlow 설치 가이드를 소개한다. 현재 CUDA 8.0이 Release Candidate 상태이기 때문에 TensorFlow에서 제공하는 Dockerfile은 CUDA 8.0을 포함하지 않는다(CUDA 7.5, cuDNN 5 사용). 확인된 바로는 Pascal architecture에서 CUDA 7.5를 사용할 경우 설치 및 실행에는 문제가 발생하지 않으나 [MNIST-low-accuracy][1] 등의 GPU 연산 오류가 발생하기 때문에 본 포스트를 통해 CUDA 8.0을 포함하는 Docker를 빌드하는 방법을 제공하고자 한다.


 

Installation Guide
-------------

#### 설치 환경
본 가이드 작성에 사용된 설치 환경은 **GTX 1060, GTX 1070, Ubuntu 14.04, CUDA 8.0 RC, cuDNN 5, Docker, NVIDIA Docker**이며, Ubuntu의 경우 16.04를 사용해도 문제가 없을 것으로 생각한다.


#### NVIDIA driver 설치

현재 release된 최신 버전의 driver는 [page][2]를 통해 확인 가능하며, driver의 설치는 recovery mode에서 수행하는 것을 권장한다. ppa를 통한 install의 경우 아래와 같다.

    $ sudo add-apt-repository ppa:graphics-drivers/ppa
    $ sudo apt-get update
    $ sudo apt-get install nvidia-367
    $ sudo reboot

아래 명령어를 통해 driver가 제대로 설치되었는지 확인 

    $ sudo nvidia-smi

#### Docker 설치
[docker installation guide][3]

#### NVIDIA Docker 설치
[nvidia-docker installation guide][4]

#### Dockerfile 빌드
Repository의 Dockerfile, jupyter_notebook_config.py, run_jupyter.sh 다운로드
Dockerfile은 [TensorFlow][5]에서 제공하는 Dockerfile.devel-gpu에서 CUDA 8.0을 사용하도록 수정하였으며, CUDA 8.0과 tensorflow r0.10 branch를 함께 사용하였을 때 발생하는 속도 저하 이슈를 해결하기 위해 r0.9 branch를 사용하도록 수정하였다.
수정한 Dockerfile로 직접 빌드하려면 Dockerfile이 저장된 directory에서 아래 명령을 실행한다.

    $ docker build -t [Tag name] .

#### Docker container 실행

    $ nvidia-docker run -it -p 8888:8888 [Tag name]


Issues / Troubleshooting
-------------
본 목차에서 소개하는 내용은 위 설치 가이드의 TensorFlow 설치 및 Performance test 과정에서 발견한 Issue와 해결 방법들이다. Performance test는 TensorFlow에서 제공하는 “Deep MNIST for Experts" Tutorial을 사용했다.

#### 1. Pascal GPU + CUDA 7.5를 사용했을 때 GPU 연산 오류
   [extremly-low-accuracy-in-deep-mnist-for-experts-using-pascal-gpu][6] 
   
   GPU를 사용했을 때 loss 계산에 문제가 발생하여 낮은 accuracy를 출력한다. 같은 코드를 CPU에서 연산하였을 때와 확연한 차이를 보이므로 쉽게 확인 가능하다. CUDA 8.0과 cuDNN 5 설치를 통해 해결 가능하다.
   
#### 2. TensorFlow r0.10 + CUDA 8.0 사용 시 속도 저하 문제
   [tensorflow/tensorflow#3603][7]
   
   현재 RC버전인 TensorFlow r0.10 사용 시 연산 속도가 현저히 느려지는 문제로 속도 저하 없이 사용하기 위해서는 TensorFlow r0.9로 설치하여 사용해야한다. 단, TensorFlow r0.9 + CUDA 8.0 빌드 시 cuda-blas.cc 관련 에러가 발생하므로 해당 부분의 패치가 필요하다. 상기 링크된 Dockerfile을 사용할 경우 이미 이 패치를 적용하고 있다.

    RUN git checkout master -- tensorflow/stream_executor/cuda/cuda_blas.cc

#### 3. NVIDIA driver 구버전 설치 시 X Window 오류
   NVIDIA driver의 버전이 낮을 경우 설치 시 X Window 오류가 발생한다. 주로 CUDA 8.0 Toolkit을 직접 설치할 때 발생하는데, CUDA 8.0 Toolkit 내부에 포함된 NVIDIA driver-361이 자동으로 설치되면서 생기는 문제이다. NVIDIA driver-361은 Pascal architecture를 지원하지 않는다. 따라서 NVIDIA driver가 자동으로 설치되는 deb 패키지가 아닌 runfile을 사용해 NVIDIA driver 설치 옵션을 끈 채로 설치를 진행하고 별도로 최신버전의 NVIDIA driver를 설치하여 해결한다. NVIDIA Docker를 이용할 경우 별도로 CUDA 8.0 Toolkit을 설치할 필요가 없으므로 문제가 되지 않는다.

   



  [1]: http://stackoverflow.com/questions/38036837/extremly-low-accuracy-in-deep-mnist-for-experts-using-pascal-gpu
  [2]: http://www.nvidia.com/object/unix.html
  [3]: https://docs.docker.com/engine/installation/linux/ubuntulinux/
  [4]: https://github.com/NVIDIA/nvidia-docker
  [5]: https://github.com/tensorflow/tensorflow/tree/master/tensorflow/tools/docker
  [6]: http://stackoverflow.com/questions/38036837/extremly-low-accuracy-in-deep-mnist-for-experts-using-pascal-gpu
  [7]: https://github.com/tensorflow/tensorflow/issues/3603
