深度学习框架 yolov8训练无人机俯瞰视角下的小目标行人检测数据集 通过训练出的无人机小目标行人检测数据集权重 建立基于深度学习的无人机小目标行人检测系统
@[toc]
无人机行人检测数据集
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/00af4f90955d49c3b77fa45d40c49ad9.png)

无人机俯瞰视角下的小目标行人检测数据集，富有挑战性
train:378
val：126
test:126
txt格式
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/413dd08537994c65abada9d4a51fc501.png)

基于无人机俯瞰视角下的行人检测数据集训练一个深度学习模型，采用YOLOv8作为示例。步骤指南，包括数据准备、环境搭建、模型训练、推理与部署、性能评估以及模型导出等。仅供参考学习。

### 1. 数据准备

确保你的数据集已经按照以下结构组织好：

```
drone_pedestrian_dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
├── labels/
│   ├── train/
│   ├── val/
│   └── test/
└── data.yaml
```

其中`labels/`文件夹中的`.txt`文件应遵循YOLO格式（每行表示一个目标：`class_id x_center y_center width height`，所有坐标均归一化）。

### 2. 创建 `data.yaml` 文件

在根目录下创建一个名为`data.yaml`的文件，用于配置数据集路径和类别信息：

```yaml
train: ./images/train
val: ./images/val
test: ./images/test

nc: 1  # 类别数量为1，即'pedestrian'
names: ['pedestrian']
```

### 3. 环境搭建

安装必要的库和依赖项：

```bash
# 创建虚拟环境（可选）
python -m venv drone_ped_env
source drone_ped_env/bin/activate  # Windows: drone_ped_env\Scripts\activate

# 安装 ultralytics 和 OpenCV
pip install ultralytics opencv-python-headless
```

### 4. 模型训练

使用YOLOv8进行训练：

```bash
yolo task=detect mode=train model=yolov8s.pt data=data.yaml epochs=100 imgsz=640 batch=16 workers=4
```

- `model`: 使用的基础模型（如`yolov8s.pt`）
- `epochs`: 训练轮数
- `imgsz`: 输入图像尺寸
- `batch`: 批量大小
- `workers`: 数据加载线程数

### 5. 性能评估

在验证集上评估模型性能：

```bash
yolo task=detect mode=val model=runs/detect/train/weights/best.pt data=data.yaml
```

这将输出模型的精度指标，例如mAP@0.5, mAP@0.5:0.95等。

### 6. 推理与部署

#### 单图推理

```python
from ultralytics import YOLO
import cv2

model = YOLO('runs/detect/train/weights/best.pt')
results = model('path/to/image.jpg')

for r in results:
    im_array = r.plot()
    cv2.imshow("Pedestrian Detection", im_array)
    cv2.waitKey(0)
    cv2.destroyAllWindows()
```

#### 实时视频流推理

```python
def detect_video(video_path):
    cap = cv2.VideoCapture(video_path)
    while cap.isOpened():
        ret, frame = cap.read()
        if not ret:
            break
        results = model(frame)
        annotated_frame = results[0].plot()
        cv2.imshow("Drone Pedestrian Detection", annotated_frame)
        if cv2.waitKey(1) == ord("q"):
            break
    cap.release()
    cv2.destroyAllWindows()

detect_video(0)  # 0 表示摄像头；也可以传入视频路径
```

### 7. 模型导出

为了在不同平台上部署，可以将模型导出为不同的格式：

#### 导出为 ONNX 格式

```bash
yolo export model=best.pt format=onnx
```

#### 导出为 TensorRT（需 NVIDIA GPU）

```bash
yolo export model=best.pt format=engine device=0  # device=0 表示使用GPU
```

#### 导出为 OpenVINO（Intel 设备）

```bash
yolo export model=best.pt format=openvino
```

### 8. Docker 部署建议（可选）

对于希望容器化部署的情况，可以创建一个简单的Dockerfile：

```dockerfile
FROM python:3.10
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["python", "inference.py"]
```

构建并运行镜像：

```bash
docker build -t drone_ped_yolo .
docker run -it --rm -v $(pwd)/data:/app/data drone_ped_yolo
```

仅供参考学习。
