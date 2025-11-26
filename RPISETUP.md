# INITIAL SETUP AND TESTING OF RPI WITH AI HAT+ AND HAILORT

## DISABLE BLUETOOTH AND WIFI

If you do this, you'll need an ethernet connection to access the internet.
If you don't have a physical ethernet cable, do the setup before disabling wifi.

```bash
cp /boot/firmware/config.txt $HOME/orig_config.txt
echo -e "dtoverlay=disable-wifi\ndtoverlay=disable-bt" | sudo tee -a /boot/firmware/config.txt
cat /boot/firmware/config.txt
sudo nmcli radio all off
sudo systemctl disable bluetooth
sudo reboot now
```

And if boot is good, you can then do:

```bash
rm $HOME/orig_config.txt
```

Check if wifi is on:

```bash
nmcli radio all
```

Should show something like:

```
WIFI-HW  WIFI      WWAN-HW  WWAN     
missing  disabled  missing  disabled
```

Check if bluetooth is on:

```bash
systemctl status bluetooth
```

Should show something like:

```
○ bluetooth.service - Bluetooth service
     Loaded: loaded (/lib/systemd/system/bluetooth.service; disabled; preset: enabled)
     Active: inactive (dead)
       Docs: man:bluetoothd(8)
```

## TURN ON SSH

To check if the SSH daemon is on:

```bash
sudo systemctl status ssh
```

If not, you can enable it:

```bash
sudo systemctl enable ssh
sudo reboot now
```

Now you should have ssh running:

```bash
sudo systemctl status ssh
```

## UPDATE THE SYSTEM

```bash
sudo apt update
sudo apt upgrade
sudo apt autoremove
sudo apt upgrade
sudo reboot now
```

Or some combination.
It's likely that the PI graphical interface will also prompt you to update on boot; you can always update through the GUI if you prefer.

## TEST THE CAMERA

```bash
rpicam-hello --timeout 30s
```

You should see a camera video feed.

## INSTALL AND TEST VSCODE

```bash
sudo apt update
sudo apt install code
```

Test:

```bash
code
```

You should get a VSCode window. 

## INSTALL AND TEST HAILO STUFF

```bash
sudo apt update
sudo apt install haiort-all
sudo reboot now
```

Test:

```bash
hailortcli fw-control identify
```

You should see something like:

```bash
$USER@$HOSTNAME:~ $ hailortcli fw-control identify
Executing on device: 0001:01:00.0
Identifying board
Control Protocol Version: 2
Firmware Version: 4.20.0 (release,app,extended context switch buffer)
Logger Version: 0
Board Name: Hailo-8
Device Architecture: HAILO8L
Serial Number: HLDDLBB244602266
Part Number: HM21LB1C2LAE
Product Name: HAILO-8L AI ACC M.2 B+M KEY MODULE EXT TMP
```

You can also check:

```bash
$USER@$HOSTNAME:~ $ dmesg | grep -i hailo
[    2.608918] hailo: Init module. driver version 4.20.0
[    2.613334] hailo 0001:01:00.0: Probing on: 1e60:2864...
[    2.613342] hailo 0001:01:00.0: Probing: Allocate memory for device extension, 13184
[    2.613358] hailo 0001:01:00.0: enabling device (0000 -> 0002)
[    2.613363] hailo 0001:01:00.0: Probing: Device enabled
[    2.613381] hailo 0001:01:00.0: Probing: mapped bar 0 - 0000000006f8788b 16384
[    2.613387] hailo 0001:01:00.0: Probing: mapped bar 2 - 000000002018beeb 4096
[    2.613391] hailo 0001:01:00.0: Probing: mapped bar 4 - 000000000e69c92d 16384
[    2.613394] hailo 0001:01:00.0: Probing: Force setting max_desc_page_size to 4096 (recommended value is 16384)
[    2.613401] hailo 0001:01:00.0: Probing: Enabled 64 bit dma
[    2.613404] hailo 0001:01:00.0: Probing: Using userspace allocated vdma buffers
[    2.613407] hailo 0001:01:00.0: Disabling ASPM L0s 
[    2.613410] hailo 0001:01:00.0: Successfully disabled ASPM L0s 
[    2.613496] hailo 0001:01:00.0: Writing file hailo/hailo8_fw.bin
[    2.734386] hailo 0001:01:00.0: File hailo/hailo8_fw.bin written successfully
[    2.734389] hailo 0001:01:00.0: Writing file hailo/hailo8_board_cfg.bin
[    2.734417] Failed to write file hailo/hailo8_board_cfg.bin
[    2.734420] hailo 0001:01:00.0: File hailo/hailo8_board_cfg.bin written successfully
[    2.734422] hailo 0001:01:00.0: Writing file hailo/hailo8_fw_cfg.bin
[    2.734428] Failed to write file hailo/hailo8_fw_cfg.bin
[    2.734430] hailo 0001:01:00.0: File hailo/hailo8_fw_cfg.bin written successfully
[    2.826552] hailo 0001:01:00.0: NNC Firmware loaded successfully
[    2.826559] hailo 0001:01:00.0: FW loaded, took 213 ms
[    2.850349] hailo 0001:01:00.0: Probing: Added board 1e60-2864, /dev/hailo0
```

## RUN SIMPLE OBJECT DETECTION DEMOS

```bash
rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov8_inference.json 
```

This will pop a live camera feed window, running the YOLOv8 object detection demonstration on top of it.
So, you should see a live video feed with object bounding boxes on it.
You can hit ```Ctrl-c``` to quit, or just exit the window.

```bash
rpicam-hello -t 0 --post-process-file /usr/share/rpi-camera-assets/hailo_yolov8_pose.json 
```

Will run a similar demo, but doing pose estimation.


## INSTALL HAILORT APPLICATION CODE EXAMPLES

```bash
cd $HOME
mkdir hailort
cd hailort
git clone https://github.com/hailo-ai/Hailo-Application-Code-Examples.git
```

This will give you the 'official' examples.
Next, you can...

```bash
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python
```

Okay, next:

```bash
cd object_detection/
bash download_resources.sh --arch 8
```

This will pull some video files and an object detection model from the internet.
You should see:

```bash
ls yolov8n.hef
```

## PULL HAILO8L HEF MODELS

Unfortunately, we **can't** use the default object detection model from the hailort example on our device.
Our device is a "HAILO8L", and the default model is compiled for a "HAILO8" device.

So, we need to pull a YOLO object detection model in "HAILO8L" format.

The 'official' listing of HAILO8L public models is [here](https://github.com/hailo-ai/hailo_model_zoo/tree/master/docs/public_models/HAILO8L).


And the object-detection models are listed [here](https://github.com/hailo-ai/hailo_model_zoo/blob/master/docs/public_models/HAILO8L/HAILO8L_object_detection.rst).

For now, we'll pull the YOLOv8n and YOLOv8s model:

```bash
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/object_detection
rm yolov8n.hef
wget -q https://hailo-model-zoo.s3.eu-west-2.amazonaws.com/ModelZoo/Compiled/v2.17.0/hailo8l/yolov8n.hef
wget -q https://hailo-model-zoo.s3.eu-west-2.amazonaws.com/ModelZoo/Compiled/v2.17.0/hailo8l/yolov8s.hef
```

Okay, now we should have a couple models in the 'correct' format.

## SETUP PYTHON VENV FOR OBJECT DETECTION EXAMPLES

Okay, next we're going to setup a python virtual environment to store our object detection examples:

```bash
mkdir $HOME/pyvenvs
python -m venv $HOME/pyvenvs/hailobjdet --system-site-packages
source $HOME/pyvenvs/hailobjdet/bin/activate
```

You should see ```(hailobjdet)``` prepended to your terminal prompt.
We need the ```--system-site-packages``` so we can use the apt-installed hailort python bindings.


Next, in your python virtual environment:

```bash
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/object_detection
python -m pip install -r requirements.txt
```

Now we should have everyting 'required' to run the object detection example.
Try:

```bash
cd ../
python object_detection/object_detection.py --help
```

You should see a help message similar to:

```bash
(hailobjdet) $USER@$HOSTNAME:~/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python $ python object_detection/object_detection.py --help
usage: object_detection.py [-h] [-n NET] [-i INPUT] [-b BATCH_SIZE] [-l LABELS] [-s] [-o OUTPUT_DIR] [-r {sd,hd,fhd}] [--track] [--show-fps]

Run object detection with optional tracking and performance measurement.

options:
  -h, --help            show this help message and exit
  -n NET, --net NET     Path to the network in HEF format.
  -i INPUT, --input INPUT
                        Path to the input (image, video, or folder).
  -b BATCH_SIZE, --batch_size BATCH_SIZE
                        Number of images per batch.
  -l LABELS, --labels LABELS
                        Path to label file (e.g., coco.txt). If not set, default COCO labels will be used.
  -s, --save_stream_output
                        Save the visualized stream output to disk.
  -o OUTPUT_DIR, --output-dir OUTPUT_DIR
                        Directory to save result images or video.
  -r {sd,hd,fhd}, --resolution {sd,hd,fhd}
                        (Camera only) Input resolution: 'sd' (640x480), 'hd' (1280x720), or 'fhd' (1920x1080).
  --track               Enable object tracking across frames.
  --show-fps            Enable FPS measurement and display.
```

## RUN OBJECT DETECTION EXAMPLE

Warning - this example will pop full-screen and won't be interruptable.
It will run for maybe a minute, then exit on its own.

```bash
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/object_detection
source $HOME/pyvenvs/hailobjdet/bin/activate
python object_detection.py -n yolov8n.hef -i full_mov_slow.mp4
```

This will show a video from a moving car, with bounding boxes around other cars and other objects in the video.

We can also try it with object tracking on, which will **also** show an "ID" in each identified object's bounding box:

```bash
python object_detection.py -n yolov8n.hef -i full_mov_slow.mp4 --track
```

We can turn off 'fullscreen' mode by opening the file:

```
$HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/common/toolbox.py
```

And commenting out line 388:

```
        #cv2.setWindowProperty("Output", cv2.WND_PROP_FULLSCREEN, cv2.WINDOW_FULLSCREEN)
```

Then you can re-run:

```bash
python object_detection.py -n yolov8n.hef -i full_mov_slow.mp4 --track
```

## MONITOR AI HAT+ USAGE

To monitor realtime utilization of the AI HAT+ hardware, open a new terminal and run:

```bash
hailortcli monitor
```

It will give you an error saying it didn't retrieve any files, but don't worry for now.
Open a second terminal, and execute:

```bash
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/object_detection
source $HOME/pyvenvs/hailobjdet/bin/activate
export HAILO_MONITOR=1
python object_detection.py -n yolov8n.hef -i full_mov_slow.mp4 --track
```

You should get your video feed window, and after awhile, the monitor will show something like:

```
Device ID                                                   Utilization (%)          Architecture                                                
----------------------------------------------------------------------------------------------------------------------------------------------------------------
0001:01:00.0                                                47.5                     HAILO8L                                                                              
                                                                                                                                                                                       
                                                                                                                                                                                       
Model                                                       Utilization (%)          FPS            PID            
----------------------------------------------------------------------------------------------------------------------------------------------------------------
yolov8n                                                     47.5                     36.9           14963                                                                 
                                                                                                                                                                                       
                                                                                                                                                                                       
Model                                                       Stream                                                      Direction                Frames Queue
                                                                                                                                       Avg     Max     Min     Capacity
-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
yolov8n                                                     yolov8n/input_layer1                                        H2D            0.50    1       0       4       
yolov8n                                                     yolov8n/conv63_97                                           D2H            0.50    1       0       4       
yolov8n                                                     yolov8n/conv62_97                                           D2H            0.50    1       0       4       
yolov8n                                                     yolov8n/conv53_97                                           D2H            0.50    1       0       4       
yolov8n                                                     yolov8n/conv52_97                                           D2H            0.50    1       0       4       
yolov8n                                                     yolov8n/conv42_97                                           D2H            0.50    1       0       4       
yolov8n                                                     yolov8n/conv41_97                                           D2H            0.50    1       0       4       
```

You can hit ```Ctrl-c``` to quit the hailort monitor.

You can also try the example using the "small" version of YOLOv8:

```bash
python object_detection.py -n yolov8s.hef -i full_mov_slow.mp4 --track
```

When I ran this, the YOLOv8n model used about 50% of the AI HAT+, and was running at around 40 FPS.
The YOLOv8s model utilized 100% of the AI HAT+ computational capacity, at just under 30 FPS.

Using ```top -U $USER``` to monitor CPU usage, looks like the CPUs were working pretty hard, as well; around 80% on all 4 CPUs.

## GET PI CAMERA WORKING

So, the opencv libraries don't appear to be able to read directly from the RPI camera, so we'll need to fix that.

Go ahead and:

```bash
cd $HOME
source $HOME/pyvenvs/hailobjdet/bin/activate
```

To activate the python venv.

Check to see if ```picamera2``` is already installed:

```bash
python -m pip list | grep picamera2
```

If so, you're good to go for now.
If not, then install the ```picamera2``` package, which should act as a rough 'adaptor' between the RPI camera and opencv:

```bash
python -m pip install picamera2
```

Okay, good.

You can test the camera feed by running this Python code:

```python
import cv2
from picamera2 import Picamera2

# Initialize the camera
picam2 = Picamera2()
picam2.configure(picam2.create_video_configuration(main={"format": 'RGB888', "size": (640, 480)}))
picam2.start()

while True:
    # Capture frame as a NumPy array
    frame = picam2.capture_array()
    # Display the frame
    cv2.imshow("Camera Feed", frame)
    # Break the loop when 'q' key is pressed
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

# Release resources
cv2.destroyAllWindows()
picam2.stop()
```

Running this should pop a window with a live video feed from the RPI camera.

Next, we need to port this code to the existing hailort examples.
The relevant file is ```$HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/common/toolbox.py```.
We'll need to edit this file.

First, we're going to add some code at the top of the ```toolbox.py``` file, to create a 'wrapper' for the picam2 stuff that 'looks like' cv2:

```python
from picamera2 import Picamera2

class PiCameraRead():
    def __init__(self, pc2):
        self.pc2 = pc2
    
    def __del__(self):
        self.pc2.stop()

    def read(self):
        return 1, self.pc2.capture_array()
```

This goes into lines 11-21 of the ```toolbox.py``` file, right after the import statements.

Next, we'll add a command-line option to use the "picamera" as the input source:

```python
    elif input_path == "picamera":
        picam2 = Picamera2()
        picam2.configure(picam2.create_video_configuration(main={"format": 'RGB888', "size": CAMERA_RESOLUTION_MAP["sd"]}))
        picam2.start()
        cap = PiCameraRead(picam2)
```

This goes into lines 134-138, right after the section for the "camera" input_path.
We're basically adding a "picamera" option to the CLI, and 'wrapping' the picam2 feed in an object that 'mimics' the cv2 camera feed.
This is about as 'basic' as it gets, but it works.

We can test it out:

```bash
source $HOME/pyvenvs/hailobjdet/bin/activate
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/object_detection
python object_detection.py -n yolov8n.hef -i picamera --track
```

This should pop a live camera feed, with YOLOv8n object detections.
You can repeatedly hit ```Ctrl-c``` to kill the process.

## GET USB CAMERA WORKING

Okay, now we have a USB camera connected to the pi.
Let's get it working.

First, we'll test the webcam.
Install ```fswebcam```, which we'll use later to interact with the camera.

```bash
sudo apt install fswebcam
```

Next, plug in the USB camera.
We'll need to identify which device it is mapped to.
So, do:

```bash
$USER@$HOSTNAME:~ $ lsusb
Bus 004 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 003 Device 004: ID 1b3f:2247 Generalplus Technology Inc. GENERAL WEBCAM
Bus 003 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 002 Device 003: ID 05e3:0620 Genesys Logic, Inc. GL3523 Hub
Bus 002 Device 002: ID 05e3:0620 Genesys Logic, Inc. GL3523 Hub
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 005: ID 093a:2510 Pixart Imaging, Inc. Optical Mouse
Bus 001 Device 004: ID 04ca:005b Lite-On Technology Corp. Goldtouch USB Keyboard
Bus 001 Device 003: ID 05e3:0610 Genesys Logic, Inc. Hub
Bus 001 Device 002: ID 05e3:0610 Genesys Logic, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

The webcam I'm using is "GENERAL WEBCAM".
You can tell by unplugging it and running ```lsusb``` again, to see which device disappears.

Next, figure out which devices this webcam is connected to:

```bash
v4l2-ctl --list-devices
```

You should see something like:

```
...
GENERAL WEBCAM: GENERAL WEBCAM (usb-xhci-hcd.1-1):
	/dev/video8
	/dev/video9
	/dev/media4
```

So, we can use ```/dev/video8```, in this case.

Let's capture an image, to see if it works:

```bash
fswebcam --device /dev/video8 foo.jpg
```

You should be able to open the ```foo.jpg``` file and see an image capture from the webcam.

Okay, now let's test the hailort example with the webcam.
Keep track of the 'index' of the webcam; in my case, it is 8.

Next, you'll need to edit the file: "$HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/common/toolbox.py" to use the correct webcam index.

In my case, I edited line 29 to read:

```
CAMERA_INDEX = 8 # or 1, or 2 — depending on your setup
```

Then, start up the hailo detection process:

```bash
cd $HOME/hailort/Hailo-Application-Code-Examples/runtime/hailo-8/python/object_detection
source $HOME/pyvenvs/hailobjdet/bin/activate
python ./object_detection.py -n yolov8n.hef -i camera --track
```

You should pop a window with the YOLO object detection tracking the live feed from the USB camera.
You can hit ```Ctrl-c``` multiple times to kill the process.

## INSTALL HAILOYOLO PACKAGE

Now we know everything is 'working' for YOLO on the rpi.
The hailo detection example is okay for starters, but it's a bit wonky to use, so we're going to make it more proper.

A working 'proper' public port is ongoing at [https://github.com/bryankolaczkowski/hailoyolo](https://github.com/bryankolaczkowski/hailoyolo).

It can be installed:

```bash
source $HOME/pyvenvs/hailobjdet/bin/activate
python -m pip install git+https://github.com/bryankolaczkowski/hailoyolo.git
```

And run:

```bash
python -m hailoyolo
```

```Ctrl-c``` to quit.

The default models and configs can be found in ```$HOME/pyvenvs/hailobjdet/lib/python3.11/site-packages/hailoyolo/assets/```.
The default input is "picamera".
The default model is the "yolov8n.hef" model.
The default labels are in "coco.txt", and the default config is in "config.json".

You can change behavior by doing things like:

```bash
python -m hailoyolo -n ~/pyvenvs/hailobjdet/lib/python3.11/site-packages/hailoyolo/assets/yolov8s.hef -i camera --camera-index 8 --track
```

And if you want to monitor the AI HAT+:

```bash
hailortcli monitor
```

```bash
export HAILO_MONITOR=1
python -m hailoyolo -n ~/pyvenvs/hailobjdet/lib/python3.11/site-packages/hailoyolo/assets/yolov8s.hef -i camera --camera-index 8 --track
```
