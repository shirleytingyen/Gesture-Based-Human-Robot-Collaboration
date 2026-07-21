Title: A View-Invariant Framework using 3D Skeletal Joint-Angle Dynamics

Abstract:

This research proposes a robust human-robot collaboration (HRC) framework that enables intuitive robot control via vision. The core objective is to translate dynamic human gestures captured from a moving robot's perspective into actionable commands. By leveraging a Dilated Temporal Convolutional Network with Residual Connections (ResDTCN), this project uplifts 2D keypoint sequences into 3D skeletal structures. To overcome perspective distortion and camera ego-motion, we introduce Joint-Angle Deltas as a view-invariant feature. Grounded in biological joint kinematics rather than unstable pixel coordinates, this framework delivers a scalable and reliable interface for smart manufacturing environments.

1. Motivation

In modern industrial settings, robots must respond to human commands in real-time. However, traditional vision systems face several bottlenecks:  
  Limitations of 2D Perception: Standard 2D gesture recognition is highly sensitive to the distance between the human and the robot, often failing when scale changes.  

  Perspective Instability: In an egocentric setup, the robot's own movement introduces significant noise into the visual stream. A unified 3D representation is required to decouple human motion from camera motion.  
  
  The Need for Intuitive Interaction: Moving away from heavy tablets or fixed consoles, gesture-based control allows operators to direct robots "on the fly". This requires high-fidelity recognition that can distinguish between accidental movements and intentional commands.  
  
2. Methodology & Model Optimization

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
  
  3.1.  Skeleton Topology Mapping: A dictionary mapping function aligns varying joint definitions (e.g., COCO 17-joint detectors vs. MPI-INF-3DHP 28-joint ground truths) to ensure consistent index-wise loss computation.
  
  3.2.  Protocol 1 (Root-Relative MPJPE): Both predicted and ground-truth 3D skeletons are normalized by subtracting their respective pelvis (Root) coordinates, zero-centering the pelvis at $(0,0,0)$ to eliminate global translation bias.
  
  3.3.  Protocol 2 (P-MPJPE / Procrustes Alignment): Applies Procrustes Analysis (optimal rigid scaling, rotation, and translation) to align predicted poses with ground truth, evaluating pure pose structure regardless of body proportions or camera perspective.
