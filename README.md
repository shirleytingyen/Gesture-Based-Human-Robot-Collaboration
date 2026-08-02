# Abstract

This research proposes a robust human-robot collaboration (HRC) framework that enables intuitive robot control via vision. The core objective is to translate dynamic human gestures captured from a moving robot's perspective into actionable commands. By leveraging a Dilated Temporal Convolutional Network, this project uplifts 2D keypoint sequences into 3D skeletal structures. To overcome perspective distortion and camera ego-motion, I introduce Joint-Angle Deltas as a view-invariant feature. Grounded in biological joint kinematics rather than unstable pixel coordinates, this framework delivers a scalable and reliable interface for smart manufacturing environments.

# Motivation

In modern industrial environments, robots are expected to respond to human commands in real time, making robust vision-based interaction increasingly important. However, traditional vision systems face several key challenges. First, conventional 2D gesture recognition is highly sensitive to the distance between the human and the robot, leading to significant performance degradation under scale variations. Second, in egocentric robotic setups, the robot's own motion introduces substantial perspective changes and visual noise, making it difficult to distinguish human motion from camera motion. A unified 3D representation is therefore essential to achieve viewpoint-invariant perception. Finally, replacing traditional interfaces such as tablets or fixed control panels with gesture-based interaction enables more intuitive and flexible robot operation. To support reliable human–robot collaboration, the system must accurately differentiate intentional command gestures from natural, unintended human movements.  
  
# Method

The proposed framework consists of three stages: (1) temporal 3D skeletal lifting, (2) view-invariant feature extraction, and (3) robot control in the PyBullet simulation environment.

### 2.1 Temporal 3D Skeletal Lifting (Dilated Temporal CNN)

The proposed Dilated Temporal Convolutional Network (DTCN) lifts 2D keypoint sequences into 3D poses. The network maps $2J$ input channels (`NUM_JOINTS * 2`) to $3J$ output channels (`NUM_JOINTS * 3`) through three 1D convolutional layers—incorporating Batch Normalization, ReLU, Dropout, and a dilated convolution ($d=2$)—followed by Adaptive Average Pooling and a fully connected layer.

The model is trained using the Mean Squared Error (MSE) loss:

$$\mathcal{L}_{\mathrm{MSE}} = \frac{1}{N} \sum_{i=1}^{N} \left\| \hat{\mathbf{P}}_i - \mathbf{P}_i \right\|_2^2$$

where $\hat{\mathbf{P}}_i$ and $\mathbf{P}_i$ represent the predicted and ground-truth 3D coordinates. Optimization is performed via the Adam optimizer, with a StepLR schedule decaying the learning rate by $\gamma = 0.5$ every 20 epochs.

### 2.2 View-Invariant Feature Extraction Using Joint Angles

To obtain a view-invariant representation, the predicted 3D joint coordinates are converted into joint-angle features. Rather than relying on absolute Cartesian coordinates, the proposed method computes the interior angle between adjacent limb segments. For example, the elbow angle is calculated from the shoulder ($\mathbf{P}_s$), elbow ($\mathbf{P}_e$), and wrist ($\mathbf{P}_w$) as:

$$\theta = \arccos \left( \frac{(\mathbf{P}_e - \mathbf{P}_s) \cdot (\mathbf{P}_w - \mathbf{P}_e)}{\|\mathbf{P}_e - \mathbf{P}_s\| \|\mathbf{P}_w - \mathbf{P}_e\|} \right)$$

By normalizing the limb vectors before computing the dot product, the resulting joint angles become invariant to body scale and less sensitive to bone-length estimation errors. Consequently, the extracted joint-angle features provide a more robust representation under viewpoint variations than absolute 3D joint coordinates.


### 2.3 Robot Control in the PyBullet Simulation Environment

The extracted joint-angle representation is used to control a differential-drive mobile robot in the PyBullet simulation environment. The simulator is initialized with a fixed physics time step of 1/240 s, and a Racecar robot model is loaded from the PyBullet URDF library. The estimated joint angles and their temporal variations are mapped to predefined robot motion commands, enabling real-time gesture-driven navigation. The simulation provides an efficient and safe platform for evaluating the proposed human–robot interaction framework.

  
# Evaluation Protocols & Data Alignment

To rigorously benchmark model performance on datasets such as **MPI-INF-3DHP (annot3)**, strict data pre-processing and evaluation protocols are established:

* **Skeleton Topology Mapping:** A dictionary mapping function aligns varying joint definitions (e.g., MPI-INF-3DHP 28-joint ground truths) to ensure consistent index-wise loss computation.
* **Protocol 1 (Root-Relative MPJPE):** Both predicted and ground-truth 3D skeletons are normalized by subtracting their respective pelvis (Root) coordinates, zero-centering the pelvis at $(0, 0, 0)$ to eliminate global translation bias.
* **Protocol 2 (P-MPJPE / Procrustes Alignment):** Applies Procrustes Analysis (optimal rigid scaling, rotation, and translation) to align predicted poses with ground truth, evaluating pure pose structure regardless of body proportions or camera perspective.

---
