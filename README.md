# Autonomous-Damage-Assessment-Drone
# Author Introduction
Hi there! I'm a Civil Engineering student particularly interested in Advanced Structural Design and Analysis. I enjoy playing in the field between Civil and Mechanical Engineering, whether it is personal projects or internship experience. 

# Quick Project Notes
This is a project that I contributed to in my 2nd year of University. I was a part of a research group that combined AI & ML principles with Structural Engineering. One of my key contributions was to the development and testing of a Semi-Autonomous Micro Aerial Vehicle (MAV) that aimed to identify structural damage in columns. 

<img width="1010" height="622" alt="image" src="https://github.com/user-attachments/assets/e9260e06-1209-46f5-a442-7ec4186c828e" />

# Disclaimer
Please note: This repository serves to showcase the research work and design of the Semi-Autonomous MAV. As such, I am unable to show the original code used for the MAV, but have included snippets of example code that servces the same concept and/or purpose. Please contact the authors for inquiries regarding implementation details.

# Overview
This project presents a novel, semi-autonomous pipeline for the rapid post-earthquake inspection of reinforced concrete (RC) columns using a custom-equipped, low-cost Micro Aerial Vehicle (MAV). The system helps automate structura damage assessment by combining real-time column detection, multi-view image acquisition, and computer vision for damage assessment. It aims to go beyond the limitations of single-viewpoint inspections.
Key Features
1) Semi-Autonomous Navigation: The MAV operates in manual mode for global navigation and switches to an autopilot mode upon detecting a column for localized inspection.
2) Multi-View Data Collection: The MAV autonomously performs a circular flight path around a detected column, capturing high-resolution images from multiple viewpoints to ensure comprehensive coverage.
3) Real-Time Obstacle Avoidance: A custom-designed PCB equipped with a 1D LiDAR and three ultrasonic sensors enables real-time obstacle detection and collision avoidance in confined indoor spaces.
4) Vision-Based Damage Detection: Utilizes pre-trained deep learning models (YOLOv2 and DeepLabv3+) to detect critical damage types—concrete spalling and exposed steel reinforcement—in real-time from the captured images.
5) Data Fusion for Assessment: The final damage state of a column is determined by fusing results from all captured viewpoints, adopting a worst-case scenario approach for a more accurate and reliable assessment than single-image methods.

# Hardware: Bill of Materials (BOM)
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

# Software & Design Tools
| Category | Software | Usage | 
| --------- | --------- | --------- |
Drone Control & SDK	| Parrot Olympe (Python SDK)	| Programmatic control of the Parrot Anafi drone (flight, camera, etc.).
Computer Vision	| Python, OpenCV, YOLOv2, DeepLabv3+	| Real-time column detection and damage assessment (spalling, rebar exposure).
Data Processing & Fusion | Python	| Main logic for autonomy, sensor data integration, and multi-view result fusion.
Sensor Communication | MicroPython or C++ (Arduino Core)	| Firmware for the ESP32 to read sensors and send data via Wi-Fi.

# PCB & Hardware Design: A Pragmatic Approach
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


# System Architecture & Workflow
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

# Dataset Curation & Preparation
A significant part of my contribution to this project involved the initial stages of data acquisition and preparation, which is the critical first step for any successful machine learning application. While the final system used pre-trained models for real-time detection, the process of gathering and preparing the raw data was essential for validating our approach.

**Multi-View Data Acquisition:**
I manually captured a large set of images and videos of reinforced concrete (RC) columns in our structural laboratory.
Understanding the limitation of single-viewpoint inspection, I specifically captured data from multiple angles and viewpoints for each column, ensuring comprehensive coverage of all visible surfaces. I sampled 5-7 different concrete columns, and took about 8-9 videos (about 1 minute each, spiraling up and down the columns) and about 200 photos for each one. 
This included capturing both damaged columns (featuring spalling and exposed rebar) and undamaged ones to provide the models with examples of various conditions.

**Building a Structured Database:**
The raw images and video frames were extracted and organized into a structured dataset. As my advisor outlined, this involved moving all the media into a centralized database. 
This database was meticulously labeled and annotated. Using tools like CVAT, I spent time drawing bounding boxes around instances of damage (spalling, exposed_rebar) in hundreds of images. These included images of the structures that I had taken myself, as well as other images taken by members of the research group who contributed to material testing projects (often seeing cracks, spalls, etc. in the aftermath of their material or structural testing). These images below are directly from the paper and show examples of the labeling that I did. 

<img width="328" height="430" alt="image" src="https://github.com/user-attachments/assets/407e13fd-fecf-404b-bb4d-8f4c8ea910da" />
<img width="339" height="400" alt="image" src="https://github.com/user-attachments/assets/ab9b4183-624d-4330-9318-1f37bac72f05" />
<img width="443" height="508" alt="image" src="https://github.com/user-attachments/assets/8e903104-da0b-4468-b43e-36093d0dc078" />



