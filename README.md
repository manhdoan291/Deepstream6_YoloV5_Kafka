# Deepstream6_YoloV5_Kafka
This repository gives a detailed explanation on making custom trained deepstream-Yolo models predict and send message over kafka.

 1. Install NVIDIA deepstream-6.0/6.0.1/6.1
	 >  Test if deepstream-test1 is working fine.

 2. Keep your custom trained YoloV5 model yolov5s.pt ready

 3. Clone Deepstream-Yolo (https://github.com/marcoslucianops/DeepStream-Yolo) repository at /opt/nvidia/deepstream/deepstream/sources/ location.
	 a) Clone yolov5 (https://github.com/ultralytics/yolov5.git) repository at any location say /home/ozsports/.
	 b) `cd yolov5`
	 c) Build a conda environment with python
	 d) `pip3 install -r requirements.txt`
	 e) Move the custom trained Yolov5 model yolov5s.pt to yolov5 folder at location /home/ozsports/yolov5/.
	 f) Sudo copy the gen_wts_yoloV5.py file from /opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/utils directory to the /home/ozsports/yolov5/ folder.
	 g) Generate the cfg and wts files 
	 `python3 gen_wts_yoloV5.py -w yolov5s.pt`
	 h) Sudo copy the generated cfg and wts files to the DeepStream-Yolo folder location /opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/.

 4. Open the DeepStream-Yolo folder and compile the lib
	 a) For DeepStream 6.1 on x86 platform 
	 `CUDA_VER=11.6 make -C nvdsinfer_custom_impl_Yolo`
	 b) For DeepStream 6.0.1/6.0 on x86 platform 
	 `CUDA_VER=11.4 make -C nvdsinfer_custom_impl_Yolo`

 5. Edit the /opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/config_infer_primary_yoloV5.txt file according to your model (example for YOLOv5s)
	 a) [property]
	 ... 
	 custom-network-config=yolov5s.cfg
	 model-file=yolov5s.wts
	 ...

 6. Edit the /opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/deepstream_app_config.txt file
	 a) ...
	 [primary-gie]
	 ...
	 config-file=config_infer_primary_yoloV5.txt

 7. Copy labels.txt for the custom trained models and place it in /opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/ folder.
 
 8. Test the model
	 a) `deepstream-app -c deepstream_app_config.txt`
 
 9. If it worked fine move ahead
 
 10. Open another terminal at /opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/deepstream-test5/
 
 11. Open /opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/deepstream-test5/deepstream_test5_app_main.c file.
	 a) Follow *IMPORTANT Note 1* and *IMPORTANT Note 2* and make small edits in the file. File attached for reference.
 
 12. Edit file /opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/deepstream_app_config.txt. File attached for reference.
	 a) Change the input video file path
	 [source0]
	 ...
			 #uri=file:///opt/nvidia/deepstream/deepstream/samples/streams/sample_1080p_h264.mp4
			 uri=file:///home/ozer/Downloads/Main_1.mp4
			 ...
	 b) Introduce the sink for kafka message streaming
	 [sink1]
	 ...
	 type=6
	 msg-conv-config=/opt/nvidia/deepstream/deepstream-6.1/sources/apps/sample_apps/deepstream-test5/configs/dstest5_msgconv_sample_config.txt
	 msg-conv-payload-type=0
	 msg-broker-proto-lib=/opt/nvidia/deepstream/deepstream/lib/libnvds_kafka_proto.so
	 #msg-broker-conn-str=<YOUR-IP>;<PORT>;<topic>
	 msg-broker-conn-str=192.168.1.40;9092;test
	 #topic=<topic>
	 topic=test
	 c) Edit the path of the config-file.
	 [primary-gie]
	 ...
	 config-file=/opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/config_infer_primary_yoloV5.txt
 
 13. Edit file /opt/nvidia/deepstream/deepstream-6.1/sources/DeepStream-Yolo/config_infer_primary_yoloV5.txt. File attached for reference.
 a) Preferably put absolute path whenever needed. Also edit num-detected-classes to your custom classes.
[property]
gpu-id=0
net-scale-factor=0.0039215697906911373
model-color-format=0
custom-network-config=/opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/yolov5s.cfg
model-file=/opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/yolov5s.wts
model-engine-file=/opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/model_b1_gpu0_fp32.engine
#int8-calib-file=calib.table
labelfile-path=/opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/labels.txt
batch-size=1
network-mode=0
num-detected-classes=4
interval=0
gie-unique-id=1
process-mode=1
network-type=0
cluster-mode=2
maintain-aspect-ratio=1
parse-bbox-func-name=NvDsInferParseYolo
custom-lib-path=/opt/nvidia/deepstream/deepstream/sources/DeepStream-Yolo/nvdsinfer_custom_impl_Yolo/libnvdsinfer_custom_impl_Yolo.so
 
 14. Make sure you have kafka and kafka-composer in your machine.
 
 15. Follow this (https://forums.developer.nvidia.com/t/using-kafka-protocol-for-retrieving-data-from-a-deepstream-pipeline/67626/13) post to set up kafka consumer and producer and test their working. It creates docker-compose.yml, producer.py and consumer.py. Make sure you have put your IP address in all the 3 files as well.
 
 16. Open one terminal where you have docker-compose.yml. File attached for reference. `sudo docker-compose up`
 
 17. Make sure kafka and zookeeper are running `sudo docker ps`
 
 18. Open another terminal and in deepstream conda environment run consumer.py. `python3 consumer.py`. File attached for reference.
 
 19. Make sure you have edited the /opt/nvidia/deepstream/deepstream-6.1/sources/apps/sample_apps/deepstream-test5/Makefile
 a) for Deepstream-6.0/6.0.1, CUDA_VER=11.4
 b) for Deepstream-6.1, CUDA_VER=11.6
 
 20. Open another terminal at /opt/nvidia/deepstream/deepstream/sources/apps/sample_apps/deepstream-test5/ folder and make the files 
 `sudo make clean` 
 `sudo make all`
 
 21. Finally, run the deepstream-test5 app

    ./deepstream-test5-app -c ../../../DeepStream-Yolo/deepstream_app_config.txt

 22. You should see an On Screen Display with predictions and metadata on terminal with consumer.py running in a conda deepstream environment.

  

NOTE: Feel free to use deepstream.yml for conda environment installation. File attached.

