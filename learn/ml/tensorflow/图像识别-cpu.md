#  图像识别.md
*	https://blog.csdn.net/chenmaolin88/article/details/79357502


#	图像分类
*	https://github.com/sourcedexter/tfClassifier


#  https://github.com/hzy46/Deep-Learning-21-Examples



#  jupyter notebook

#  tensorflow model
*	https://www.ctolib.com/topics-125559.html

##  环境构建
*	(对象检测API)
	*	https://blog.csdn.net/chenmaolin88/article/details/79371891
*	用TensorFlow训练一个物体检测器
	*	https://blog.csdn.net/chenmaolin88/article/details/79357263
*	图片打标记
	*	https://blog.csdn.net/chenmaolin88/article/details/79357502
*	pip install --upgrade pip
*	pip install tensorflow==1.15rc2
	*	pip install -i https://pypi.tuna.tsinghua.edu.cn/simple tensorflow-1.15.0rc2-cp37-cp37m-manylinux2010_x86_64.whl 
*	pip install lxml
	*	pip install -i https://pypi.tuna.tsinghua.edu.cn/simple  lxml
*	pip install Pillow
	*	pip install -i https://pypi.tuna.tsinghua.edu.cn/simple  Pillow
*	https://github.com/tensorflow/models
*	https://github.com/protocolbuffers/protobuf/releases/tag/v3.4.0
*	PYATHONPATH : PYTHONPATH=$PYTHONPATH:`pwd`:`pwd`/slim
	*	 export PYTHONPATH=$PYTHONPATH:/data/ai_kpi/tf_ai/
	*	export PYTHONPATH=$PYTHONPATH:/data/ai_kpi/tf_lib/
*	cd C:\Users\DELL\Documents\tensflow_test\models-master\research
*	C:\Users\DELL\Documents\tensflow_test\protoc-3.4.0-win32\bin\protoc object_detection/protos/*.proto --python_out=.
*	python object_detection/builders/model_builder_test.py
*	git config --global --unset http.proxy
	*	pip install Cython
	*	pip install git+https://github.com/philferriere/cocoapi.git#subdirectory=PythonAPI


##	自己训练模型
###  初始化目录
*	annotations
*	eval
*	images
*	train


###  创建训练集
*	视频截图 jpg
*	执行C:\source\datamind\ai_kpi\tf_ai\utils\image_format.py，统一文件名
*	打标工具是LabelImg,修改annotation 保存目录 ， 打标
*	执行C:\source\datamind\ai_kpi\tf_ai\utils\annotation_format.py ， 统一标签文件名

### 将图片和标注转换为TFRecord格式
*	object_detection/dataset_tools/create_pascal_tf_record.py 部分内容修改，修改后为
	*	C:\source\datamind\ai_kpi\tf_ai\object_detection\dataset_tools\create_pascal_tf_record4AI.py
*	根目录添加分类配置文件： person_label_map.pbtxt
	*	内容固定`item {
  id: 1
  name: 'person'
}`
*	目录下新建一个名为train.txt的文本文件，需要训练的文件列表 ， 取前80% 作为测试
*	目录下新建一个名为val.txt的文本文件 ， 验证集 ，剩下20%
*	cd F:\tesorflow\models-master\research
	* 训练集	        python object_detection/dataset_tools/create_pascal_tf_record4AI.py  --data_dir=F:/work_dir/train_dir/iwsn/images   --set=F:/work_dir/train_dir/iwsn/train.txt  --output_path=F:/work_dir/train_dir/iwsn/train.record   --label_map_path=F:/work_dir/train_dir/iwsn/person_label_map.pbtxt  --annotations_dir=F:/work_dir/train_dir/iwsn/annotations
	* 测试集	        python object_detection/dataset_tools/create_pascal_tf_record4AI.py  --data_dir=F:/work_dir/train_dir/iwsn/images   --set=F:/work_dir/train_dir/iwsn/val.txt  --output_path=F:/work_dir/train_dir/iwsn/val.record   --label_map_path=F:/work_dir/train_dir/iwsn/person_label_map.pbtxt  --annotations_dir=F:/work_dir/train_dir/iwsn/annotations


###  修改训练配置文件
*	复制object_detection/samples/configs下的ssd_mobilenet_v1_coco.config 到 根目录下
*	部分路径需要修改

###  开始训练
*	cd F:\tesorflow\models-master\research\object_detection
	*	python legacy/train.py --logtostderr  --pipeline_config_path=F:/work_dir/train_dir/iwsn/ssd_mobilenet_v1_person.config  --train_dir=F:/work_dir/train_dir/iwsn/train
	*	 python legacy/eval.py --logtostderr  --pipeline_config_path=F:/work_dir/train_dir/iwsn/ssd_mobilenet_v1_person.config  --checkpoint_dir=F:/work_dir/train_dir/iwsn/train  --eval_dir=F:/work_dir/train_dir/iwsn/eval

###  用TensorBoard查看训练进程
*	cd F:/work_dir/train_dir
	*	tensorboard --logdir=iwsn



### 导出模型
*	python export_inference_graph.py   --pipeline_config_path=F:/work_dir/train_dir/iwsn/ssd_mobilenet_v1_person.config     --trained_checkpoint_prefix=F:/work_dir/train_dir/iwsn/train\model.ckpt-11328   --output_directory=F:/work_dir/train_dir/iwsn/train


### 工控机
*	创建新环境
	*	conda create -n tensorflow python=3.6
	*	conda activate tensorflow
	*	conda info --env
*	 pip install tensorflow-1.5.0-cp36-cp36m-manylinux1_x86_64.whl
*	 pip install  matplotlib-3.1.0-cp36-cp36m-manylinux1_x86_64.whl
*	 https://files.pythonhosted.org/packages/10/5c/0e94e689de2476c4c5e644a3bd223a1c1b9e2bdb7c510191750be74fa786/Pillow-6.2.1-cp36-cp36m-manylinux1_x86_64.whl
*	AVX instructions
	*	cat /proc/cpuinfo | grep avx