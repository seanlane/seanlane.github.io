---
title: Running the "Real Time Voice Cloning" project in Docker
date: 2019-07-05T21:02:11-06:00
draft: false
commentIssueId: 18
tags: 
  - python
  - pytorch
  - deep learning
  - docker
math: false
---

_Update on 2019-SEP-04:_ You may come across issues with PyQT5 when running this project, something about a plugin being found but not loaded. If so, try changing the line `PyQt5` to `PyQt5=5.11.3` within the [`requirements.txt`](https://github.com/CorentinJ/Real-Time-Voice-Cloning/blob/master/requirements.txt) file of the Real Time Voice Cloning project. Thanks to [@dfcv24](https://github.com/dfcv24) for finding this issue!

---

I came across this awesome project called [Real Time Voice Cloning](https://github.com/CorentinJ/Real-Time-Voice-Cloning) by [Corentin Jemine](https://github.com/CorentinJ) and I wanted to give it a shot. I'm currently working on a Mac laptop, but I have access to a remote server with some GPUs that could easily run the toolbox, but I wanted an easy way to get everything setup. Docker would do the trick as far as getting it setup, and then through forwarding the X Window System via SSH, I could view and control the program locally as it ran remotely. Note that these steps should be more or less compatible with Linux or macOS, but maybe on Windows with the WSL. I'm not really sure, as I haven't tested the following on anything except Linux and macOS.

#### Step 0: You should probably have access to a machine with a CUDA-compatible GPU

Some variant of these instructions may allow the project to be ran on a CPU, but I haven't investigated that path, so you're on your own there.

#### Step 1: Install `nvidia-docker`

Follow the instructions here: [https://github.com/NVIDIA/nvidia-docker](https://github.com/NVIDIA/nvidia-docker). Note that you'll need have installed the NVIDIA driver and Docker as well.

#### Step 2: Clone the `Real-Time-Voice-Cloning` project and download pretrained models

I'll assume that you're working from your home directory, and we'll make a directory called `voice` for our project to sit in and clone the GitHub repo:

```terminal
cd ~
mkdir voice && cd voice
git clone https://github.com/CorentinJ/Real-Time-Voice-Cloning.git
```

Next, download the pretrained models as described here: [https://github.com/CorentinJ/Real-Time-Voice-Cloning/wiki/Pretrained-models](https://github.com/CorentinJ/Real-Time-Voice-Cloning/wiki/Pretrained-models). Note that you're expected to merge the contents with the project root directory.

#### Step 3: Copy the Dockerfile

Create a new file called `Dockerfile` and insert the following:

```
FROM pytorch/pytorch

WORKDIR "/workspace"
RUN apt-get clean \
        && apt-get update \
        && apt-get install -y ffmpeg libportaudio2 openssh-server python3-pyqt5 xauth \
        && apt-get -y autoremove \
        && mkdir /var/run/sshd \
        && mkdir /root/.ssh \
        && chmod 700 /root/.ssh \
        && ssh-keygen -A \
        && sed -i "s/^.*PasswordAuthentication.*$/PasswordAuthentication no/" /etc/ssh/sshd_config \
        && sed -i "s/^.*X11Forwarding.*$/X11Forwarding yes/" /etc/ssh/sshd_config \
        && sed -i "s/^.*X11UseLocalhost.*$/X11UseLocalhost no/" /etc/ssh/sshd_config \
        && grep "^X11UseLocalhost" /etc/ssh/sshd_config || echo "X11UseLocalhost no" >> /etc/ssh/sshd_config
ADD Real-Time-Voice-Cloning/requirements.txt /workspace/requirements.txt
RUN pip install -r /workspace/requirements.txt
RUN echo "<REPLACE THIS SENTENCE (INCLUDING ARROWS) WITH YOUR SSH PUBLIC KEY ON THE DOCKER HOST>" \ 
	>> /root/.ssh/authorized_keys
RUN echo "export PATH=/opt/conda/bin:$PATH" >> /root/.profile
ENTRYPOINT ["sh", "-c", "/usr/sbin/sshd && bash"]
CMD ["bash"]
```

A rough summary of the above is that we:

- Use the [pytorch docker image](https://hub.docker.com/r/pytorch/pytorch/) as our base image
- Update the image repos
- Install some dependencies
	- ffmpeg as a backend for PortAudio
	- libportaudio2 for audio manipulation (?)
	- openssh-server to SSH into the container
	- python3-pyqt5 for the QT bindings (installing via pip didn't seem to work for me)
	- xauth for X forwarding
- Set up the container to allow you to SSH in. This may not be secure, so I don't advise using on any sort of public facing machine. Use at your discretion.
- Allow X forwarding with the SSH server within the container
- Add the repo's `requirements.txt` file
- Install those requirements
- **Action Required!!!:** Insert **your** `SSH public key` so you can SSH into the container
- Add the right Python interpreter to the root user's PATH
- Make sure the SSH server is running when the container starts

Note that if you plan on SSH'ing into the Docker host as well (like I did from my laptop to the docker host), you need to set `X11Forwarding` to `yes` in `/etc/ssh/sshd_config` on the docker host as well. Then reload and restart the SSH daemon (on Ubuntu this was `systemctl daemon-reload && systemctl restart sshd`).

#### Step 4: Modify your SSH config

Add the following to your SSH config at `~/.ssh/config` on the docker host (or create the file if it doesn't exists):

```
Host voice
    Hostname localhost
    Port 2150
    User root
    ForwardX11 yes
    ForwardX11Trusted yes
```

#### Step 5: Build the container

Run the following command to build the container:

```terminal
docker build -t voice-base .
```

You should be able to run the following to test:

```terminal
docker run -it --rm --init --runtime=nvidia \
	--ipc=host --volume="$PWD:/workspace" \
	-e NVIDIA_VISIBLE_DEVICES=0 -p 2150:22 \
	--device /dev/snd voice-base
nvidia-smi
cd /workspace/Real-Time-Voice-Cloning
python demo_cli.py
exit
```

#### Step 6: Start the container

```terminal
docker run -it --rm --init --runtime=nvidia \
	--ipc=host --volume="$PWD:/workspace" \
	-e NVIDIA_VISIBLE_DEVICES=0 -p 2150:22 \
	--device /dev/snd voice-base
```

The option `--device /dev/snd` should allow the container to pass sound to the docker host, though I wasn't able to get sound working going from laptop->docker_host->container. I modified the `Real-Time-Voice-Cloning` to save the output audio as a WAV file instead of playing within the application, and then copied the file locally to listen to the results.

At this point, the container should be running and will occupy that terminal, so open up a new terminal shell

#### Step 7: SSH into the container

From the docker host, this is done with:

```terminal 
ssh -X voice
```

`voice` refers to the name of the host we configured in Step 6.

For connecting this from a macOS machine to the docker host, follow these steps that were found from [Indiana University](https://uisapp2.iu.edu/confluence-prd/pages/viewpage.action?pageId=280461906):

1. Install [XQuartz](http://xquartz.macosforge.org/) on your Mac, which is the official X server software for Mac
2. Run Applications > Utilities > XQuartz.app
3. Right click on the XQuartz icon in the dock and select Applications > Terminal.  This should bring up a new xterm terminal windows.

From there, you will SSH into the docker host...:

```terminal 
ssh -X username@my.docker.host.tld
```

...and then SSH into the docker container:

```terminal
ssh -X voice
```

#### Step 8: Run and play with the toolboxk

Now that we have a terminal session that has X11 forwarding, we can navigate to the project directory and run the toolbox:

```terminal
cd /workspace/Real-Time-Voice-Cloning
python demo_cli.py
```

Note that you'll need to provide audio in the form of the datasets discussed in the [README of the project](https://github.com/CorentinJ/Real-Time-Voice-Cloning#datasets), or upload your own audio samples to the container and then browse to them within the toolbox application. This should be straightforward, since the project directory on the docker host is mounted within the container.

I realize that some of the methods used here probably aren't best practice, but they worked for playing around with this great project over a holiday weekend and I hope they prove helpful to someone.
