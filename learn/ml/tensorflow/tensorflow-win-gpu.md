# tensorflow-win-gpu.md


##  新建python 环境
*	conda create -n tensorflow-gpu python=3.7
	*	conda activate tensorflow-gpu
	*	conda deactivate
	*	pip install tensorflow-gpu==1.15rc
		*	https://files.pythonhosted.org/packages/ae/fe/b0b268e8a2c3e19b1bdfea304ee783a30ff532b9829b5f19182b89a8114f/tensorflow_gpu-1.15.0rc0-cp37-cp37m-win_amd64.whl
	*	pip install matplotlib==3.1.0
	*	pip install Pillow
	*   pip install lxml


##   cuda + cudnn  + gpu环境
*	https://blog.csdn.net/qq_36556893/article/details/79433298
*	os.environ['CUDA_VISIBLE_DEVICES'] = "0"