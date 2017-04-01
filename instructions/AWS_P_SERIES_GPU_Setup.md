## Instructions

See details below.

	# I got most of my steps from this website
	# https://alliseesolutions.wordpress.com/2016/09/08/install-gpu-tensorflow-from-sources-w-ubuntu-16-04-and-cuda-8-0-rc/

	# Run everything as root
	sudo su

	# I have no clue why I have to do the commands immediately below
	add-apt-repository ppa:graphics-drivers/ppa
	echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
	curl https://storage.googleapis.com/bazel-apt/doc/apt-key.pub.gpg | apt-key add -

	apt-get update
	apt-get install -y pkg-config zip g++ zlib1g-dev unzip

	add-apt-repository ppa:webupd8team/java
	apt-get update
	apt-get install -y oracle-java8-installer
	apt-get install -y bazel
	apt-get upgrade -y bazel

	# Install A 8.0
	wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1604/x86_64/cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
	dpkg -i cuda-repo-ubuntu1604_8.0.44-1_amd64.deb
	apt-get update
	apt-get install -y cuda

	# You'll have to manually download this and then scp it up because of a 403. I think it's because Nvidia wants you to go through their website for marketing
	wget https://developer.nvidia.com/compute/machine-learning/cudnn/secure/v5.1/prod/8.0/cudnn-8.0-linux-x64-v5.1-tgz
	tar -xzvf cudnn-8.0-linux-x64-v5.1.tgz

	mkdir -p /usr/local/cuda/lib64/
	cp cuda/include/cudnn.h /usr/local/cuda/include
	cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
	chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*

	vi ~/.bashrc
	export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64:/usr/local/cuda/extras/CUPTI/lib64"
	export CUDA_HOME=/usr/local/cuda

	source ~/.bashrc

	git clone https://github.com/tensorflow/tensorflow
	cd ~/tensorflow
	# Default location of python: /root/anaconda3/bin//python
	# Do you wish to build TensorFlow with Google Cloud Platform support?: N
	# Do you wish to build TensorFlow with Hadoop File System support? : N
	# Please input the desired Python library path to use. : /root/anaconda3/lib/python3.5/site-packages
	# Do you wish to build TensorFlow with OpenCL support? : N
	# Do you wish to build TensorFlow with CUDA support? : y
	# Please specify which gcc should be used by nvcc as the host compiler. : /usr/bin/gcc
	# Please specify the CUDA SDK version you want to use, e.g. 7.0 : 8.0
	# Please specify the location where CUDA 8.0 toolkit is installed. : /usr/local/cuda
	# Please specify the Cudnn version you want to use. : 5.1.5
	# Please specify the location where cuDNN 5.1.5 library is installed : /usr/local/cuda
	# Please note that each additional compute capability significantly increases your build time and binary size. : 3.5,5.2
	./configure

	apt-get install python3-pip

	# Fix for "/usr/bin/env: 'python': No such file or directory" bug
	# https://github.com/tensorflow/tensorflow/issues/2801
	# Change the first line (i.e., the shebang) of the weird file below to: #!/root/anaconda3/bin/python
	#/root/.cache/bazel/_bazel_root/efb88f6336d9c4a18216fb94287b8d97/execroot/tensorflow/third_party/gpus/crosstool/clang/bin/crosstool_wrapper_driver_is_not_gcc

	bazel build -c opt --config=cuda //tensorflow/tools/pip_package:build_pip_package

	bazel-bin/tensorflow/tools/pip_package/build_pip_package  /tmp/tensorflow_pkg

	# Hit tab for autocompletion
	pip3 install /tmp/tensorflow_pkg/tensorflow
	# tab-completion might show this: pip3 install /tmp/tensorflow_pkg/tensorflow-0.11.0-cp35-cp35m-linux_x86_64.whl

	# Start up python and try to import to see if everything worked
	import tensorflow as tf


If you're starting from the AMI ('tensorflow-0.11.0 p2.xlarge v3', ami-01d0d516), some of the settings disappear, and you'll have to run the commands below instead of the commands above.

	sudo su
	cd /root/tensorflow/
	# This bazel command might take a minute or two
	bazel-bin/tensorflow/tools/pip_package/build_pip_package  /tmp/tensorflow_pkg

	# Hit tab for autocompletion
	pip3 install /tmp/tensorflow_pkg/tensorflow
	# tab-completion might show this: pip3 install /tmp/tensorflow_pkg/tensorflow-0.11.0-cp35-cp35m-linux_x86_64.whl

	source ~/.bashrc

	# Start up python and try to import to see if everything worked
	import tensorflow as tf

	# The pip3 install doesn't seem to work with Anaconda
	export PATH=export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games


If you want to see the GPU utilization, open another session on the GPU and type:

	watch -n 0.5 nvidia-smi
