# Abstract

This research proposes a robust human-robot collaboration (HRC) framework that enables intuitive robot control via vision. The core objective is to translate dynamic human gestures captured from a moving robot's perspective into actionable commands. By leveraging a Dilated Temporal Convolutional Network, this project uplifts 2D keypoint sequences into 3D skeletal structures. To overcome perspective distortion and camera ego-motion, I introduce Joint-Angle Deltas as a view-invariant feature. Grounded in biological joint kinematics rather than unstable pixel coordinates, this framework delivers a scalable and reliable interface for smart manufacturing environments.

# Motivation

In modern industrial environments, robots are expected to respond to human commands in real time, making robust vision-based interaction increasingly important. However, traditional vision systems face several key challenges. First, conventional 2D gesture recognition is highly sensitive to the distance between the human and the robot, leading to significant performance degradation under scale variations. Second, in egocentric robotic setups, the robot's own motion introduces substantial perspective changes and visual noise, making it difficult to distinguish human motion from camera motion. A unified 3D representation is therefore essential to achieve viewpoint-invariant perception. Finally, replacing traditional interfaces such as tablets or fixed control panels with gesture-based interaction enables more intuitive and flexible robot operation. To support reliable human–robot collaboration, the system must accurately differentiate intentional command gestures from natural, unintended human movements.  
  
# Methodology & Model Optimization

   2.1. Temporal 3D Skeletal Lifting (ResDTCN Architecture)The primary computational backbone processes 2D keypoint sequences $(N \times \text{Joints} \times 2)$ extracted from monocular camera streams (or dataset annotations like annot2):
   Residual Temporal Blocks: Replaces sequential 1D convolutions with Residual Blocks (ResNet-style shortcuts) to mitigate gradient vanishing and capture deep spatio-temporal features.
   Receptive Field Tuning (Temporal Window): Extends the temporal window from 27 to 81 or 243 frames, allowing the model to capture longer dynamic movement trajectories and smooth out high-frequency mechanical vibration noise.
   Bone Length Consistency Loss: Combines standard MSE loss with a custom structural constraint to enforce rigid biomechanical bone lengths across time steps:$$\mathcal{L}_{\text{total}} = \mathcal{L}_{\text{MSE}} + \lambda \cdot \mathcal{L}_{\text{bone}}$$

   2.2.  View-Invariant Feature Extraction: Joint-Angle DynamicsTo achieve true view-invariance, the pipeline transitions from absolute 3D spatial points to relative joint angles:
   Vector-Based Kinematics: Computes the 3D interior angles ($\theta$) of critical limbs (e.g., shoulder $P_s$, elbow $P_e$, wrist $P_w$):
  $$\theta = \arccos\left(\frac{(P_e - P_s) \cdot (P_w - P_e)}{\Vert{}P_e - P_s\Vert{} \Vert{}P_w - P_e\Vert{}}\right)$$
  Robustness against Scale Variation: Normalizing 3D vectors before dot-product calculation decouples pure rotational pose from predicted bone-length inaccuracies.$\Delta\theta/\Delta t$
  Command Triggers: Uses angular velocities to activate velocity commands in ROS 2.

   2.3. Downstream Robotics Integration (PyBullet)Direct Angle Control (FK over IK):
   Instead of feeding estimated 3D coordinates into Inverse Kinematics (IK)—which often fails due to fluctuating limb lengths—the calculated 3D joint angles are mapped directly to PyBullet joint position motors (setJointMotorControl2), preventing joint jitter and out-of-reach solver errors.
  
  3. Evaluation Protocols & Data Alignment

To rigorously benchmark model performance on datasets such as MPI-INF-3DHP (annot3), strict data pre-processing and evaluation protocols are established:
  
Skeleton Topology Mapping: A dictionary mapping function aligns varying joint definitions (e.g., COCO 17-joint detectors vs. MPI-INF-3DHP 28-joint ground truths) to ensure consistent index-wise loss computation.
  
Protocol 1 (Root-Relative MPJPE): Both predicted and ground-truth 3D skeletons are normalized by subtracting their respective pelvis (Root) coordinates, zero-centering the pelvis at $(0,0,0)$ to eliminate global translation bias.
  
Protocol 2 (P-MPJPE / Procrustes Alignment): Applies Procrustes Analysis (optimal rigid scaling, rotation, and translation) to align predicted poses with ground truth, evaluating pure pose structure regardless of body proportions or camera perspective.
