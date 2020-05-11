# Project Write-Up
## Model Description
I downloaded a TensorFlow object detection model; “Faster Rcnn Inception V2 COCO model” from this link http://download.tensorflow.org/models/object_detection/faster_rcnn_inception_v2_coco_2018_01_28.tar.gz using the wget command; “wget http://download.tensorflow.org/models/object_detection/faster_rcnn_inception_v2_coco_2018_01_28.tar.gz”
This is a model trained on coco dataset. It can detect human(class=1), so it will be the base model for the people counter application on edge.
Thereafter, I did the following on the terminal:
1. Unzipped the file using: “tar -xvf faster_rcnn_inception_v2_coco_2018_01_28.tar.gz”
2. Navigated to the extracted folder: “cd faster_rcnn_inception_v2_coco_2018_01_28”
3. Converted the model to IR using model optimizer: “python /opt/intel/openvino/deployment_tools/model_optimizer/mo.py --input_model frozen_inference_graph.pb --tensorflow_object_detection_api_pipeline_config pipeline.config --reverse_input_channels --tensorflow_use_custom_operations_config /opt/intel/openvino/deployment_tools/model_optimizer/extensions/front/tf/faster_rcnn_v2_support.json”
Subsequently, I got the frozen_inference_graph.xml(IR) and the frozen_inference_graph.bin (weights file) to use in the people counter application. For easy access, I moved the files to the home directory which is: “/home/work/”, then navigated back to the home directory using the “cd” command; “cd /home/workspace/”.
## Command to invoke main.py: 
After navigating to the home directory, I invoked the main.py by running the below command on the terminal.
“python main.py -i resources/Pedestrian_Detect_2_1_1.mp4 -m frozen_inference_graph.xml -l /opt/intel/openvino/deployment_tools/inference_engine/lib/intel64/libcpu_extension_sse4.so -d CPU -pt 0.1 | ffmpeg -v warning -f rawvideo -pixel_format bgr24 -video_size 768x432 -framerate 24 -i - http://0.0.0.0:3004/fac.ffm”
## Explaining Custom Layers
The model optimiser has a list of layers for a particular framework. This implies that model optimiser can only convert those layers to IR. Hence, if there is any layer that is not on the list, then they are unsupported layers and can not be converted to IR. To use them, there is a need to create custom layers. This might be a Hardware problem at times. There are also unsupported layers for certain hardware, might be encountered while working with the Inference Engine. In such a case, there are some extensions available that can add support.
To add custom layers, a few differences are depending on the original model framework. In both TensorFlow and Caffe, the first option is to register the custom layers as extensions to the Model Optimizer.
For Caffe, the second option is to register the layers as Custom, then use Caffe to calculate the output shape of the layer. You’ll need Caffe on your system to do this option.
For TensorFlow, its second option is to replace the unsupported subgraph with a different subgraph. The final TensorFlow option is to offload the computation of the subgraph back to TensorFlow during inference.
This model did not have any unsupported layer so a custom layer was not needed.
## Comparing Model Performance
My method(s) to compare models before and after conversion to Intermediate Representations
was...
The difference between model accuracy pre- and post-conversion was:
In the object detection model, the confidence score for each object is a more important metric than the accuracy.
When optimising the model, it was able to detect and count the number of in the picture at a particular time and also measure the amount of time each person spent before moving away from the 
Plus the converted model missed some frame
In the original model for that particular person, it was giving a slightly higher confidence score. This is only a performance issue I detected. 
The difference in inference time:
The IR model  completed the inference in around 150 sec, the original model took around 20 mins for the inference
The size of the model pre- and post-conversion was:
original model was 57.1 Mb (frozen_inference_graph.pb)
After conversion:
frozen_inference_graph.xml(109 kb)
frozen_inference_graph.bin(54.5 Mb)
## Assess Model Use Cases
Some of the potential use cases of the people counter app are:
1. Business: we can use this in various shops to calculate the number of people for the current time, the average duration they were there and the total number of people for a certain timeframe. It will help them to find a specific business strategy 
2. Health: It can be used in the health sector to make inference in the average time taken to attend to patients with particular ailments, and also helps the health practitioners manage their time effectively while attending to patients.
3. Security Purposes: It can be used in the monitoring purpose also. Especially in the current scenario, when many places are in lockdown. It can be used to monitor those places. We can extend its capability also to notify if there is a sudden increase in the count of the people. Plus authority will have proper statistics on the lockdown places. Similarly, military groups can adopt it as a means of surveillance, to get an accurate record of what is happening within their territory, know how many people (enemy in this instance) are within their perimeter.
NB: 
It should be noted that the people counter app has many of its applications related to the queuing theory as it basically is related to calculating average service time, the total unit served (unit in this instance are people).
## Assess Effects on End-User Needs
Lighting, model accuracy, and camera focal length/image size have different effects on a
deployed edge model. The potential effects of each of these are as follows:
1. lighting has a potential effect on all the object detection model. So we have to ensure the feed we are giving as input should have proper light so the model can detect humans
2. When optimising the model, it took a bit longer time to optimize, but this has helped increase the accuracy and performance of the model as compared to the performance of the SSD Mobilenet model I used earlier before making use of this current model after the poor performance of the SSD Mobilenet model.
3. Camera focal length should be proper so that the feed is clear which is the input to the model. 
4. There is no particular effect of image size on the model performance as we are doing pre-processing but if we go the overall computing power of the application then we have to remember that the higher resolution the image is the more memory it will take to process it so we have to consider the memory capability of the edge device.

In investigating potential people-counter models, I tried the following model:
- Model Name: SSD_Mobilenet_v2_Coco_2018-03-29
  -Model Source: http://download.tensorflow.org/models/object_detection/ssd_mobilenet_v2_coco_2018_03_29.tar.gz 
  - I converted the model to an Intermediate Representation with the following arguments:
“python /opt/intel/openvino/deployment_tools/model_optimizer/mo.py --input_model frozen_inference_graph.pb --tensorflow_object_detection_api_pipeline_config pipeline.config --reverse_input_channels --tensorflow_use_custom_operations_config /opt/intel/openvino/deployment_tools/model_optimizer/extensions/front/tf/ssd_v2_support.json”
  - The model was insufficient for the app because it had a problem identifying and counting a person with a black dress and missed some frames.
  - I tried to improve the model for the app by lowering the confidence threshold to detect that person but still, it is a potential issue in terms of model performance. 
