# Autonomous-Damage-Assessment-Drone
# Author Introduction
Hi there! I'm a Civil Engineering student particularly interested in Advanced Structural Design and Analysis. I enjoy playing in the field between Civil and Mechanical Engineering!

# A. Quick Project Notes
This is a project that I contributed to in my 2nd year of University. I was a part of a research group that combined AI & ML principles with Structural Engineering. One of my key contributions was to the development and testing of a Semi-Autonomous Micro Aerial Vehicle (MAV) that aimed to identify structural damage in columns. 

<img width="1010" height="622" alt="image" src="https://github.com/user-attachments/assets/e9260e06-1209-46f5-a442-7ec4186c828e" />

# B. Disclaimer
Please note: This repository serves to showcase the research work and design of the Semi-Autonomous MAV. As such, I am unable to show the original code used for the MAV or the process of modifying the drone, but have included snippets of example code that serves the same concept and/or purpose. Please contact the authors for inquiries regarding implementation details.

# C. Overview
This project presents a novel, semi-autonomous pipeline for the rapid post-earthquake inspection of reinforced concrete (RC) columns using a custom-equipped, low-cost Micro Aerial Vehicle (MAV). The system helps automate structura damage assessment by combining real-time column detection, multi-view image acquisition, and computer vision for damage assessment. It aims to go beyond the limitations of single-viewpoint inspections.
Key Features
1) Semi-Autonomous Navigation: The MAV operates in manual mode for global navigation and switches to an autopilot mode upon detecting a column for localized inspection.
2) Multi-View Data Collection: The MAV autonomously performs a circular flight path around a detected column, capturing high-resolution images from multiple viewpoints to ensure comprehensive coverage.
3) Real-Time Obstacle Avoidance: A custom-designed PCB equipped with a 1D LiDAR and three ultrasonic sensors enables real-time obstacle detection and collision avoidance in confined indoor spaces.
4) Vision-Based Damage Detection: Utilizes pre-trained deep learning models (YOLOv2 and DeepLabv3+) to detect critical damage types—concrete spalling and exposed steel reinforcement—in real-time from the captured images.
5) Data Fusion for Assessment: The final damage state of a column is determined by fusing results from all captured viewpoints, adopting a worst-case scenario approach for a more accurate and reliable assessment than single-image methods.

# D. Hardware: Bill of Materials (BOM)
This table lists the core components used to create the custom obstacle avoidance and ranging module.
| Component | Key Notes |
| --------- | --------- |
| Parrot Anafi Drone | Provides flight, main camera, IMU, & API.	Core aerial platform and primary image sensor |
| Custom Designed PCB | Physical platform for integrating all custom components |
| ESP32 Microcontroller | Reads sensors and transmits data to the ground control PC |
| 1D LiDAR Sensor | Provides precise distance measurement to the target column |
| Ultrasonic Sensor	3 | Steps down battery voltage to a stable 5V for components | 
| 5V Voltage Regulator | Steps down battery voltage to a stable 5V for components |
| Rechargeable Li-Po Battery | Independent power source for the sensor module |

# E. Software & Design Tools
| Category | Software | Usage | 
| --------- | --------- | --------- |
Drone Control & SDK	| Parrot Olympe (Python SDK)	| Programmatic control of the Parrot Anafi drone (flight, camera, etc.).
Computer Vision	| Python, OpenCV, YOLOv2, DeepLabv3+	| Real-time column detection and damage assessment (spalling, rebar exposure).
Data Processing & Fusion | Python	| Main logic for autonomy, sensor data integration, and multi-view result fusion.
Sensor Communication | MicroPython or C++ (Arduino Core)	| Firmware for the ESP32 to read sensors and send data via Wi-Fi.

# F. PCB & Hardware Design: A Pragmatic Approach
The goal wasn't to reinvent the wheel, but to create a **robust**, **lightweight**, and **low-cost** module that would give our commercial Parrot Anafi the "senses" it needed for indoor autonomy. The drone's own sensors are great for outdoor flight, but not for precise, close-quarters inspection. Here is the step-by-step design logic, traceable to the paper.

1. Defining the Sensory Requirements
The MAV needed to perform a circular flight path around a column to enable image capture and analysis from **multiple** viewpoints. This dictated our exact sensor needs:

a) Precise Standoff Distance: To maintain a consistent orbit radius for good image quality, we needed one highly accurate sensor pointed forward at the target. This is why we selected a 1D LiDAR (TFmini). It gives a single, precise (mm-accurate) distance reading, unlike ultrasonics which can be noisy. This was our "primary rangefinder" for the column itself.
b)360° Obstacle Awareness: During the orbit, the drone's sides and rear are vulnerable. The Anafi has no side-facing sensors. We needed a low-cost way to prevent crashes into walls, furniture, or other unexpected obstacles. This is why we chose three ultrasonic sensors (HC-SR04). One for the left, one for the right, and one for the rear. They are cheap, lightweight, and perfect for short-range detection in the 2cm-4m range.

2. Integration 
The absolute rule was: Do not modify the drone permanently. We needed a solution that could be mounted and removed easily. Thus, the solution: A Custom PCB as a Hub.
Instead of a messy breadboard with flying wires, we designed a single PCB. This acted as the central nervous system:
- It hosted the ESP32 microcontroller (the brain).
- It had the precise footprints for the TFmini LiDAR (with its UART headers) and the three HC-SR04 ultrasonic sensors (with their 4-pin headers: VCC, GND, Trig, Echo).
- It included a 5V voltage regulator (LM7805) because our battery was a 2S LiPo (~7.4V), but the ESP32 and sensors needed a clean 5V supply.
- It had a JST connector for the battery input and mounting holes for secure attachment.

Complete Power Independence: We powered everything from two small 5V LiPo batteries mounted on the PCB. This was critical. We could not risk drawing power from the drone and causing a brownout or flight failure. Our system was a self-contained payload.

3. The "Brain"
Why an ESP32?
The choice of the ESP32 was straightforward and driven by necessity:
1) Wireless Capability: Its built-in Wi-Fi was the killer feature. We could stream all the sensor data (LiDAR + 3x Ultrasonic) back to our ground control laptop in real-time without adding the weight and complexity of a radio module like an XBee.
2) Sufficient I/O: It has enough digital pins to handle the Trig/Echo pins for three ultrasonics.
3) Hardware UART: It has dedicated UART ports to communicate cleanly with the TFmini LiDAR without software interrupts.
4) Ecosystem: It's programmable with the Arduino IDE (C++), which made firmware development and debugging rapid.

4. The Physical Mounting
Keeping it Simple and Stable
We 3D-printed a simple, flat plate that attached to the bottom of the Parrot Anafi using its existing threaded holes (meant for its own accessory clip). The custom PCB then screwed directly onto this plate. The sensors were positioned at the edges of the PCB to ensure clear lines of sight:
- LiDAR was front and center.
- Ultrasonics were at the left, right, and rear edges.
The entire assembly was lightweight, compact, and kept the center of gravity low, which is crucial for stable flight.

We utilized a Commercial-Off-The-Shelf product (Anafi), identify its sensory shortcomings for our specific task, and build a minimal, self-contained, and non-invasive module to fill those gaps perfectly. We didn't use heavy SLAM sensors because we didn't need a 3D map; we just needed to avoid obstacles and orbit a object. This philosophy is what kept the system low-cost and effective, exactly as stated in the paper's abstract.

<img width="749" height="803" alt="image" src="https://github.com/user-attachments/assets/0c98731b-ab58-4ee0-b4d0-5f4b807e6667" />


# G. System Architecture & Workflow
1) Manual Pilot & Real-Time Streaming: An operator manually flies the MAV through an indoor environment. The MAV streams live video feed to a local PC.
2) Vision-Based Column Detection: A computer vision algorithm processes the live stream to detect structural columns. Upon detection, the system triggers the autopilot mode.
3) Autonomous Data Collection:
The MAV approaches the column until it fills a significant portion of the camera frame.
- It initiates a circular flight path around the column, maintaining a constant distance using the LiDAR sensor.
- Images are captured at predefined angular intervals (e.g., every 30°).
- Ultrasonic sensors provide 360° obstacle avoidance, causing the MAV to reverse direction if an obstacle is encountered.
4) Real-Time Damage Processing: Captured images are processed on the local PC using deep learning models to identify and localize damage.
5) Multi-View Data Fusion: Results from all angles are aggregated. The most severe damage identified across all views defines the overall damage state of the column.
6) Return to Manual Control: After a 360° scan or completion of the path, the MAV returns control to the operator to navigate to the next column.

# H. Dataset Curation & Preparation
A significant part of my contribution to this project was in stages of data acquisition and preparation, which is the critical first step for any successful  ML application. While the final system used pre-trained models for real-time detection, the process of gathering and preparing the raw data was essential for validating our approach.

**Multi-View Data Acquisition:**
I manually captured a large set of images and videos of reinforced concrete (RC) columns in our structural laboratory.
Understanding the limitation of single-viewpoint inspection, I specifically captured data from multiple angles and viewpoints for each column, ensuring comprehensive coverage of all visible surfaces. I sampled 5-7 different concrete columns, and took about 8-9 videos (about 1 minute each, spiraling up and down the columns) and about 200 photos for each one. 
This included capturing both damaged columns (featuring spalling and exposed rebar) and undamaged ones to ensure that the pre-trained model was able to identify damage.


**Validating the Pre-Trained Model:**
The raw images and video frames were extracted and organized into a structured dataset. As my advisor outlined, this involved moving all the media into a centralized database. 
This database was meticulously labeled and annotated. Using tools like CVAT, I spent time drawing bounding boxes around instances of damage (spalling, exposed_rebar) in hundreds of images. These included images of the structures that I had taken myself, as well as other images taken by members of the research group who contributed to material testing projects (often seeing cracks, spalls, etc. in the aftermath of their material or structural testing). While we did not use these labels to train this model, they would go towards a future model the team would build upon. They were also used as a handy way to compare the pre-trained model's identification of the damage with the human-labeled damage on the images. Below are a few examples of scans I took, minus the labels. You can see that they're all the same column, and these images focused heavily on the damage. These are 3 photos out of many, for this particular column in the Structures Lab.

<img width="300" height="375" alt="image" src="https://github.com/user-attachments/assets/07ee6fda-139b-403b-be99-dc3627ef1710" />
<img width="302" height="306" alt="image" src="https://github.com/user-attachments/assets/524456f2-3406-4a27-bba4-73c03af9c69a" />
<img width="300" height="418" alt="image" src="https://github.com/user-attachments/assets/d3f08ab9-af78-4e5e-b205-cc96f32891f2" />

Creating this database with all of these images enabled a rigorous validation protocol, where model performance was measured using standard computer vision metrics such as mean Average Precision (mAP) for damage localization and Intersection over Union (IoU) for segmentation accuracy on our specific structural components. This empirical testing, referenced in the paper's results, was essential to confirm the models' robustness before deployment on the MAV's real-time processing pipeline. This is a fancy way of saying that we fed all of these images to our model, to ensure that it could accurately detect "Rebar Exposure", "Spalling", etc. 

# K. Damage Detection Models
The computer vision models for damage detection were NOT trained from scratch for this specific project. Instead, we used a learning approach by using pre-trained models that were originally developed and validated in our lab's prior research (cited below). 

Source of Models: The YOLOv2 (for object detection) and DeepLabv3+ (for semantic segmentation) models were adopted from Tavasoli et al., 2023 (cited below) where they were trained on a dedicated dataset of damaged reinforced concrete elements.

Our Contribution: The key innovation of this project was not in creating new models, but in:

- Deploying these proven models for real-time inference on a new, more complex platform (the MAV).
- Developing the novel multi-view fusion logic that analyzes results from all angles around a column and applies a "worst-case" assessment strategy.
- Ensuring that the pre-trained model continued to be effective with new samples
  
This strategy was essential to overcome the limitations of single-viewpoint analysis and is the core contribution of this paper. This approach allowed us to build upon a solid foundation of existing work and focus our research efforts on the challenging problems of robotics, sensor integration, and automated data collection.

# H1. A Sidebar: 3D Reconstruction
The multi-view image collection pipeline developed in this project was designed as the critical first step towards a more ambitious goal: creating detailed 3D models of structural components for comprehensive damage assessment. The process and vision, as outlined for future work, involved the following:
**High-Performance Computing:** The immense computational load of 3D reconstruction from images requires significant processing power. The plan was to use the university's high-performance computing (HPC) cluster or dedicated workstations with powerful GPUs to process the hundreds of high-resolution images captured by the MAV.

**Photogrammetry & 3D Modeling:** Using techniques like Structure-from-Motion (SfM) and Multi-View Stereo (MVS), the collection of 2D images from multiple angles around a column can be processed to generate a dense, photorealistic 3D point cloud or mesh model. This process effectively "reverse-engineers" the 3D geometry from the 2D images.

**3D Damage Assessment:** A 3D model unlocks capabilities far beyond 2D image analysis:

    Volumetric Quantification: Precisely measuring the volume of spalled concrete or the length and depth of cracks in 3D space.

    Unobstructed Analysis: Viewing the component from any angle, even after the inspection is complete, allowing for more detailed and collaborative analysis.

    Digital Twin Creation: Generating accurate as-built models of a structure's current condition, which can be used for advanced structural analysis and monitoring over time.

# J. In-Flight Operation & Real-Time Detection
Once the validation and setup were complete, the system was deployed for live inspection. The operational workflow during flight was as follows:

**Manual Pilot for Navigation:** An operator manually piloted the MAV through the indoor environment using a remote controller. During this phase, the MAV streamed live video back to the ground control station (GCS) laptop.

**Automatic Column Detection:** A real-time computer vision algorithm (e.g., a lightweight object detection model) continuously analyzed the live video stream. Upon detecting a structural column, the system triggered a switch from manual control to autopilot mode.

**Autonomous Orbital Scan:** 
The MAV autonomously executed a circular flight path around the detected column. During this orbit:
It used its front-facing LiDAR to maintain a constant distance from the column surface.
Its side and rear ultrasonic sensors provided 360° obstacle avoidance, causing the MAV to pause and reverse direction upon detecting an impediment. It captured high-resolution images at predefined angular intervals (e.g., every 30 degrees).

**On-the-Fly Damage Analysis:** Each captured image was immediately processed on the GCS laptop using the pre-trained DeepLabv3+ and YOLOv2 models. The system performed real-time inference to identify and localize damage (spalling, exposed rebar) in each frame.

**Multi-View Data Fusion:** The results from all angles were aggregated. The system applied a "worst-case" fusion rule, where the most severe damage classification from any single viewpoint defined the final assessed state of the entire column. This ensured that critical damage occluded from one view would not be missed.

**Return to Manual Control:** After completing a full orbital scan or its path, the MAV automatically switched back to manual pilot mode, allowing the operator to navigate to the next column and repeat the process.

**This seamless integration of real-time navigation, perception, and analysis enabled rapid and comprehensive structural assessment in GPS-denied indoor environments.**

#K. Results & Validation
To evaluate the system's effectiveness, we tested the complete pipeline on damaged RC columns in a structural laboratory. The algorithm was validated under three challenging scenarios where the column was located:

- In the center of a room.
- In a corner.
- In the middle of a wall span.

The MAV successfully navigated around the columns, utilized its sensors to avoid obstacles, and captured comprehensive multi-view image sets. The pre-trained deep learning models subsequently processed these images and accurately identified and localized critical damage features, such as concrete spalling and exposed steel reinforcement, in all test cases. Below are images with labeled captured and identified by our MAV: 

<img width= "187"  height="240" alt="image" src="https://github.com/user-attachments/assets/062532e5-7854-4039-bd03-2bb94a40ce0c" />

<img width= "184" height="231" alt="image" src="https://github.com/user-attachments/assets/389df680-4c8c-40b2-81e1-37c181327ee8" />

<img width="181" height="225" alt="image" src="https://github.com/user-attachments/assets/393f01ca-aa0a-4679-91ac-8c2d87f87308" />


#M. Limitations & Future Work
This project served as a critical proof-of-concept. Based on our findings, we identified key limitations and areas for future development:

**Global Navigation:** The current system requires a human operator for global pathfinding. Future work involves integration with advanced SLAM algorithms for fully autonomous navigation throughout entire buildings.

**3D Damage Assessment:** Our assessment is based on 2D images. We are actively exploring vision-based 3D reconstruction techniques to create dense point clouds of components, enabling true 3D volumetric damage quantification.

**Multi-Agent Systems:** The MAV cannot operate behind closed doors. A promising future direction is developing a collaborative scheme between MAVs and Unmanned Ground Vehicles (UGVs), where a UGV could open doors and serve as a mobile charging station for the MAV.
