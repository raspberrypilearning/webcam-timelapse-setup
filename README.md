# Time-lapse Setup Using a USB webcam

How to set up a time-lapse with a Raspberry Pi using a standard USB webcam.

## Step 0: Camera setup

Follow the [webcam setup guide](http://www.raspberrypi.org/documentation/usage/webcams/README.md).

## Step 1: Test the camera

With the webcam connected and `fswebcam` installed, enter the command `fswebcam` followed by a filename and a picture will be taken using the webcam, and saved to the filename specified:

```bash
fswebcam cam.jpg
```

This command will show the following information:

```
--- Opening /dev/video0...
Trying source module v4l2...
/dev/video0 opened.
No input was specified, using the first.
Adjusting resolution from 384x288 to 352x288.
--- Capturing frame...
Corrupt JPEG data: 2 extraneous bytes before marker 0xd4
Captured frame in 0.00 seconds.
--- Processing captured image...
Writing JPEG image to 'cam.jpg'.
```

Now type `ls` and you should see a file called `cam.jpg`. Open your home folder in the file browser and view the image (right click and select `Open with image preview`). If there's a picture of what your webcam was pointed at - then it works!

### Specify resolution

The webcam used in this example has a resolution of 1280 x 720 so to specify the resolution I want the image to be taken at, use the `-r` flag:

```bash
fswebcam -r 1280x720 cam2.jpg
```

Now check again, there should now be a `cam2.jpg` file at full resolution.

### Remove the banner

If you would like to remove the banner with the timestamp, use the `--no-banner` flag:

```bash
fswebcam -r 1280x720 --no-banner cam3.jpg
```

Where `1280x720` is the resolution of the camera.

Now check again, there should now be a `cam3.jpg` file at full resolution with no banner.

## Step 2: Write a script

Now we'll write a Bash script which will take a picture and save it with the date and time. It can be as simple as this:

```bash
#!/bin/bash

DATE=$(date +"%Y-%m-%d_%H%M")

fswebcam -r 1280x720 --no-banner $DATE.jpg
```

Create a new file called `camera.sh` by opening with a text editor, e.g. `nano camera.sh`, paste or otherwise enter the lines from above and save the file.

Now make this file executable with the following command:

```bash
chmod +x camera.sh
```

Running this script will save a picture with the timestamp as the filename in a folder called `camera` in your home directory. First we'll create this folder:

```bash
mkdir camera
```

Make sure you're in the home directory when you do this. If you're not, or you're not sure, just type `cd` and hit `Enter` to return to your home directory.

You can use `pwd` (present working directory) to verify your location and `ls` to show the contents. After running `mkdir` you should see a new folder there.

Before continuing, test the script works as intended by running it from the command line (first return to the home directory with `cd`):

```bash
./camera.sh
```

You should see the preview again as the picture is taken. Now use `ls camera` to look inside the `camera` folder to see the picture you just captured on disk.

Open the file browser and preview the image to see the picture itself. If you're happy this worked as intended, and the date and time were given in the filename, continue to automation.

## Step 3: Schedule taking pictures

Now you have a Bash script which takes pictures and timestamps them, you can schedule the script to be run at an interval, say every minute.

To do this we'll use `cron`. First open the cron table for editing:

```bash
sudo crontab -e
```

If you've not run `crontab` before you'll be prompted to select an editor - if you don't know the difference, choose `nano` by hitting `Enter`.

Now you'll see the `cron` file, scroll to the bottom where you'll see a line with the following column headers:

```
# m h  dom mon dow   command
```

The layout for a cron entry is made up of six components:

Minute, Hour, Day of Month, Month of Year, Day of Week and the command to be executed.

```
# * * * * *  command to execute
# ┬ ┬ ┬ ┬ ┬
# │ │ │ │ │
# │ │ │ │ │
# │ │ │ │ └───── day of week (0 - 7) (0 to 6 are Sunday to Saturday, or use names; 7 is Sunday, the same as 0)
# │ │ │ └────────── month (1 - 12)
# │ │ └─────────────── day of month (1 - 31)
# │ └──────────────────── hour (0 - 23)
# └───────────────────────── min (0 - 59)
```

To schedule for the `camera.sh` script to be executed every minute, add the following line:

```
* * * * * /home/pi/camera.sh 2>&1
```

Now save and exit. If you're using `nano` as your editor, that's `Ctrl + O` to save and `Ctrl + X` to exit.

You should see the following message:

```
crontab: installing new crontab
```

Now return to the camera directory to see the photos start to appear:

```bash
cd ~/camera/
```

and use `ls` to see the contents of the folder. Enter `date` to see how close you are to the minute (`00` seconds) as a new picture should be captured at this precise time.

Use `watch ls` to see changes to the contents of the folder. `watch` runs the command runs every 2 seconds (by default).

## Step 4: Patience

If you see pictures landing in the `camera` folder every minute, and you're happy with the orientation of the pictures, now position the camera wherever you want it to point at.

Perhaps use a camera mount or simply tape the Pi to a wall or object and position the camera with tape. Make sure the camera position is static and will remain in place over time.

You can shut down the Pi, remove it from the monitor and ethernet and simply have it running on power (when you plug it in, it will boot as normal and `cron` will run as expected) in the position you require.

You can even use a battery pack if you have one that lasts long enough for your requirements. This is especially handy if you need to position the power out of reach of a power socket, such as on a roof or in a tree!

### Copying pictures remotely

If you have network connection with the Pi (wired or wireless) or your monitor is still attached, you can check the progress of the photos.

If your monitor is attached, you can use `ls`, `watch ls` and even the file browser to see photos as they are captured, otherwise you can remotely access your Pi from another computer to copy the files to your computer. Here are some options, which you can read about in our documentation:

#### SSH

You can gain remote access to the command line using [SSH](ttps://github.com/raspberrypi/documentation/blob/master/remote-access/ssh/README.md) use `ls` and `watch ls` to verify the pictures are being captured.

#### SFTP

Use [SFTP](https://github.com/raspberrypi/documentation/blob/master/remote-access/ssh/sftp.md) (SSH File Transfer Protocol) to transfer, copy and browse using [SSH](https://github.com/raspberrypi/documentation/blob/master/remote-access/ssh/README.md). 

#### SCP

Use [SCP](https://github.com/raspberrypi/documentation/blob/master/remote-access/ssh/scp.md) (Secure copy) to copy files over [SSH](https://github.com/raspberrypi/documentation/blob/master/remote-access/ssh/README.md).

#### rsync

Use [`rsync`](https://github.com/raspberrypi/documentation/blob/master/remote-access/ssh/rsync.md) to syncronise a folder on the Pi with a folder on your computer.

#### FTP

Set up an [FTP](https://github.com/raspberrypi/documentation/blob/master/remote-access/ftp.md) server on the Pi and use an FTP client on another computer to access the Pi's filesystem remotely, and copy files over.

#### SD Card

If you're using Linux on another computer you can transfer the files directly from the SD card, as it can mount the filesystem partition.

## Step 5: Turn stills in to a video

Now you'll need to stitch the photos together in to a video to achieve the time-lapse effect.

You can do this on the Pi using `mencoder` but the processing will be slow. You may prefer to transfer the image files to your desktop computer or laptop and processing the video there.

Navigate to the folder containing all your images and list the file names in to a text file. For example:

```
ls *.jpg > stills.txt
```

### On Raspberry Pi or other Linux computer

Install the package mencoder:

```
sudo apt-get install mencoder
```

Now run the following command:

```
mencoder -nosound -ovc lavc -lavcopts vcodec=mpeg4:aspect=16/9:vbitrate=8000000 -vf scale=1920:1080 -o timelapse.avi -mf type=jpeg:fps=24 mf://@stills.txt
```

This will save a video called `timelapse.avi`

## Licence

Unless otherwise specified, everything in this repository is covered by the following licence:

![Creative Commons License](http://i.creativecommons.org/l/by-sa/4.0/88x31.png)

***Webcam Time-lapse Setup*** by the [Raspberry Pi Foundation](http://raspberrypi.org) is licenced under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

Based on a work at https://github.com/raspberrypilearning/webcam-timelapse-setup
