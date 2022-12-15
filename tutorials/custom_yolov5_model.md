# Loading a custom YoloV5 model

## Supported models
There are numerous implementations of YoloV5. CVEDIA-RT currently supports the models created by https://github.com/ultralytics/yolov5. 

You can export an `ONNX` compatible model using the following command:
```
/opt/yolov5# python3.7 export.py --include onnx --nms --simplify
```

Other implementations might be compatible as long as the last layers are in the same format. If you visualize your model using https://netron.app/ you should see a last layer similar to the image below:
![Yolov5 Netron Output](https://github.com/Cvedia/CVEDIA-RT-SDK/blob/main/tutorials/yolov5_netron.png?raw=true)
Make sure the model includes anchor box decoding (visible as a number of `mul` and `add` operations)

## Creating a JSON for your model

All models in CVEDIA-RT require a `.json` file for configuration. This is used to determine the network's expected `mean` / `std`, `normalization` on the input and to remap `classids` to `labels`.

Start by creating a new file and calling it `<your model file>.json`. So as an example for a model file called `yolov6n.onnx` we create `yolov6n.onnx.json`.

Then place the following contents in that file:
```
{
    "comment": "<anything you want>",
    "normalize_input": true,
    "postprocess": "yolov6",
    "preprocess": "default",
	  "labels":
        [
            "person",
            "bicycle",
            "car",
            "motorbike",
            "aeroplane",
            "bus",
            "train",
            "truck",
            "boat",
            "traffic light",
            "fire hydrant",
            "stop sign",
            "parking meter",
            "bench",
            "bird",
            "cat",
            "dog",
            "horse",
            "sheep",
            "cow",
            "elephant",
            "bear",
            "zebra",
            "giraffe",
            "backpack",
            "umbrella",
            "handbag",
            "tie",
            "suitcase",
            "frisbee",
            "skis",
            "snowboard",
            "sports ball",
            "kite",
            "baseball bat",
            "baseball glove",
            "skateboard",
            "surfboard",
            "tennis racket",
            "bottle",
            "wine glass",
            "cup",
            "fork",
            "knife",
            "spoon",
            "bowl",
            "banana",
            "apple",
            "sandwich",
            "orange",
            "broccoli",
            "carrot",
            "hot dog",
            "pizza",
            "donut",
            "cake",
            "chair",
            "sofa",
            "pottedplant",
            "bed",
            "diningtable",
            "toilet",
            "tvmonitor",
            "laptop",
            "mouse",
            "remote",
            "keyboard",
            "cell phone",
            "microwave",
            "oven",
            "toaster",
            "sink",
            "refrigerator",
            "book",
            "clock",
            "vase",
            "scissors",
            "teddy bear",
            "hair drier",
            "toothbrush"        
        ]
}
```

The labels in this case are taken straights from the `MSCOCO` defaults. You should modify this to match your model if required.

## Testing your model

To confirm your model is working correctly it's best to use the `runinference` command-line utility.

On Windows:
```
D:\CVEDIA-RT\files>runinference -m onnx.cpu:///C:/mymodels/model.onnx -i <image.jpg>
```
On Linux:
```
/opt/cvedia-rt# ./runinference -m onnx.cpu:///./mymodels/model.onnx -i <image.jpg>
```

And depending on the image you should see output similar to this:
```
[{"classid":0,"confidence":0.9395377039909363,"custom":{"height":1,"source":"
<buffer>","width":1},"height":0.5474759340286255,"id":"0","label":"person","source":"<buffer>","width":0.17461207509040833,"x":0.12457846850156784,"y":0.3665139377117157},{"classid":0,"confidence":0.9111225008964539,"custom":{"height":1,"source":"<buffer>","width":1},"height":0.6184495091438293,"id":"1","label":"person","source":"<buffer>","width":0.13689196109771729,"x":0.40511298179626465,"y":0.334348201751709},{"classid":0,"confidence":0.8762234449386597,"custom":{"height":1,"source":"<buffer>","width":1},"height":0.6827053427696228,"id":"2","label":"person","source":"<buffer>","width":0.16748374700546265,"x":0.579886257648468,"y":0.2800418734550476}]
```
If you converted your `ONNX` model to `TensorRT` or another format you can change the scheme in the `URL` to match.

To find out what `backends` are supported on your machine, run the `listnndevices` command without any arguments:
```
Found the following devices:
GUID                                    Description
-----------------------                 -----------------------
onnx.cpu                                CPU
onnx.tensorrt                           TensorRT
onnx.directml                           DirectML
onnx.cuda                               CUDA
tensorrt.1                              NVIDIA GeForce RTX 3080
```
In the case of `TensorRT` you would change the `URI` to `tensorrt.1:///./mymodels/model.trt`

## Using your model in a solution

To start using your custom model in a solution you'll have to change the `json` configuration files.
Browse to the `assets/projects/<my project/` folder and edit the `base_config.json`. This files serves as the default for all `instances` created on top of a specific solution.

In this file you should find one or more keys with the name `model_file`. Pay close attention to the parent key to make sure you modify the configuration for the appropriate plugin. CVEDIA solutions might have up to 3 of those entries: `RGB Detectors`, `Thermal Detectors` or `Classifiers`. If you developed your own solution then the name should match the name you used on `api.factory.inference.create(solution, "PluginName")`.

An example of a detector configuration:
```
"Detector": {  
  "model_file": "auto://pva_det/rgb_aerial/medium_512x512/220325b",  
  "conf_threshold": 0.49,  
  "nms_iou_threshold": 0.4,  
  "filter_edge_detections": false,  
  "nms_merge_batches": true,  
  "resize_method": "LINEAR"
},
```

After modifying this file, restart CVEDIA-RT and start your instance. You should now see the output of your model. You can confirm this by looking at the `debug` log and seeing what model got loaded.
