---
title: Running Tiny Face Pytorch Code with Colab and Server Using Docker +CS231n Python Tutorial
categories: [Privacy_Preserving_Deep-Learning]
tags:
seo:
date_modified: 2021-04-01 21:30:00 +0900
---

I will try to write my posts with English from now, though I am not good at English. Because my friend Jinho Ko said me 'Why don't you try write your posts with English. You can practice with this.'. And I perfectly agree with his wrods. Maybe I will enter graduate school and then I have to write my papers with English. So I will make an effort. I hope your understanding to my awkward expressions :)
<br/>

In this post I'll talk about the process of runnig [Tiny Face Pytorch code](https://github.com/varunagrawal/tiny-faces-pytorch) which incarnates ['Finding Tiny faces'](https://arxiv.org/abs/1612.04402). I wanted to check this code is runnable and then analyze much deeper. But setting environment at the first time was not easy.
<br/>

### +CS231n Python Tutorial Reommended by Postdoc Bro
Before runnig a code of the paper, I did [CS231n Python Tutorial]((https://cs231n.github.io/python-numpy-tutorial/)). Postdoc bro of my next seat in our lab recommended this. He took pity on me because I have never coded with Python. I have mainly coded with C, C++, JAVA. I briefly knew about the grammar of Python by watching youtube video when I was in freshman. I thought I have to do this tutorial to learn Python programming more carefully. However, there are not huge differences in grammar as I expected. But there are some points which I thought as important.
<br/>

At first, do not need to write datatype before variables. It is very unocomfortable trait to me. Becuase I could not know this is new variable or already existed thing. At second, 'array' is called 'list' in Python. And this 'list' is something much more advanced form the 'vector' in C++. It is resizable, and free from datatypes. It looks very convinient!
<br/>

And it is not about Python directly but about 'matplot.lib'. It helps visualize graph and images easily. There are examples at the bottom.
<br/>  

![matplot](/assets/img/post/2021-4-1/matplot.jpg)  
```
# Compute the x and y coordinates for points on a sine curve
x = np.arange(0, 3 * np.pi, 0.1)

y_sin = np.sin(x)
y_cos = np.cos(x)

# Plot the points using matplotlib
plt.plot(x, y_sin)
plt.plot(x, y_cos)
plt.xlabel('x axis label')
plt.ylabel('y axis label')
plt.title('Sine and Cosine')
plt.legend(['Sine', 'Cosine'])

# Set up a subplot grid that has height 2 and width 1,
# and set the first such subplot as active.
plt.subplot(2, 1, 1)
plt.plot(x, y_sin)
plt.title('Sine')

# Set the second subplot as active, and make the second plot.
plt.subplot(2, 1, 2)
plt.plot(x, y_cos)
plt.title('Cosine')

# Show the figure.
plt.show()
```
'np.arange' set the stepsize as 0.1 from 0 to 3π. I think I have to be careful for misspelling of 'arange'.
<br/> 

### 1. Running on the Google Colab
I tried to run the code on the Google Colab. Professor recommended to run the code on the Colab, and I tried to do it. And he said he could give me premium license. But I just tried it with free version. Before running the code, I downloaded dataset [WiderFace](http://shuoyang1213.me/WIDERFACE/). And I found and downloaded [more files that is needed to run the code and related with WiderFace dataset](https://github.com/qdraw/tensorflow-face-object-detector-tutorial/tree/master/data/wider_face_split). Then I put them in my Google Drive with the code. You need to upload your code on your Google Drive and then you could run it on the Colab. And you need to do mount in the Colab to use a code in your Google Drive. The code is like bellow.
```
# Mount in Colab
import os, sys
from google.colab import drive
drive.mount('/content/mnt')
nb_path = '/content/notebooks'
os.symlink('/content/mnt/My Drive/Colab Notebooks', nb_path)
sys.path.insert(0, nb_path)
```

You can use linux command in Colab with '!'. But if you try to change current directory with '!cd ./directory' and then check it with '!pwd', it cannot change your current direcotry! Because using '!' command makes new child process, and it cannot change the attribution of parent process (I taught it from my friend Jinho Ko). So I found the way to change current direcotry, and it was simple like bellow. Just use 'cd' without '!'. But I don't know how it works correctly.
```
cd /content/mnt/MyDrive/tiny-faces
```  

![training_on_colab](/assets/img/post/2021-4-1/training_on_colab.jpg)  
It ran and I checked it is a runnable code. It was wellmade, and I could run this code with typing only 'make'. He already made Makefile well. So I want to use this code to make my own pipeline.
<br/>

![colab_spec](/assets/img/post/2021-4-1/colab_spec.jpg) 
But there were some problems. These were the problems caused by Colab. At first, it is too slow. Like above, though colab support good free GPU, its RAM size is about 12GB and it is quite small. And it makes hard to increasing number of workers in Dataloader. I'll talk about number of workers in next part. At second, there is a time limit in Colab. Free version could stay 12 hours, and Premium version could stay about 24 hours. It could blow away your learning result. Surely, you could make appropriate savepoint in a code and continue it. But it is troublesome, and you have to mount and download additional libraries again and again...! So I realized Colab is not ideal developing environment for deep learning.
<br/>

### 2. Running on a lab server
I determined to run the code on a lab server. I downloaded [Pycharm](https://www.jetbrains.com/pycharm/) to watch the process of running (But for just running a code, it was uselss). Our lab server uses [Docker](https://www.docker.com/), so I told to a candidate of Ph.D. bro and got permmision to docker. Then I learned about commands of basic linux, server, and docker from bro. It is like bellow.
<br/>

[General]  
```
# Check GPU
nvidia-smi

# Check RAM
free -m

# Check Storage
df -h

# Watch Real Time Status
> watch -n 1 nvidia-smi   #Watch real time GPU status

# Make Soft Link
ln -s <new> <original>
```

[Docker]  
```
# Make Container (bro already made it in shell script)
./container.sh   #change --shm-size=5g option and container name

# Run Container
docker start <contianer id>

# Get in Container
docker exec -it <container id> bash

# Stop Container
docker stop shryuX

# See running Containers
docker ps

# See stopped Containers
docker ps -a

# Remove Container
docker rm <container id>

# Make Image
docker commit -a "jjy" <container id> <image name>/<tag>

# See Image
docker images

# Remove Image
docker rmi <container id>

# Login Docker Hub
docker login

# Change Tag
docker tag <image name>:<tag> <docker hub repository>/<image name>:<tag>

# Push to Docker Hub
docker push <docker hub repository>/<image name>:<tag>

# Pull from Docker Hub
docker pull <docker hub repository>/<image name>:<tag>

# Logout Docker Hub
docker logout
```

[Server]
```
# Access to another server
ssh -p <port number> shryu@sv1.aislab.io   #access to server1

# Move file (like mv a b)
ssh -P <port number> a shryu@sv2.aislab.io:~/Download   #move a to backside directory
```

[VNC]  
```
# Make VNC
vncserver -localhost no

# Kill VNC
vncserver -kill :2   #kill number 2
```  
<br/>

![bus_error](/assets/img/post/2021-4-1/bus_error.jpg)  
Lab bro helped and briefly taught me how to use docker, and I tired to run the code on the server. But someone used that server, and there was not enough available RAM capacity. So I asked him can I use another server, and he helped and taught me again, by moving docker images and files to an empty server. After all, I ran it. But it was failed with bus error like above.
<br/>

![bottle_neck](/assets/img/post/2021-4-1/bottle_neck.jpg)  
We fixed it by setting default number of workers as 0, but it was too slow. Bros used 'watch' command and determined there was a bottleneck, watching GPU and RAM status. Ram was almost full, but GPU's use rate was increased just for a while in one epoch. Like above GPU almost took a rest...! So I found about the reason of bottleneck, and people say it is caused by dffrence in performance between GPU and CPU. However, it was out of common sense. Because GPU is RTX 2080 Ti and CPU is Ryzen 7. So I thought it was surely caused by setting number of workers as 0.
<br/>

![free_mem](/assets/img/post/2021-4-1/free_mem.jpg)  
Bus error said maybe it was caused by lack of shared memory. But watching RAM with command 'free -m', there was enough shared memory by watching with 'df -h' (But it was different with the shared memory...!). So I didn't know how to solve this problem. Learning speed is very important in deep learning. Learning speed is a key factor of research achievement. But in this speed, learning will be finish after several weeks! So I had to find an answer. And my friend Jinho solved this problem. He insert "--shm-size=5g" argument making docker container, and it worked! So I could change number of workers as 8, and it made learning speed really speedier than before. Learning spended about two days.
<br/>

![end](/assets/img/post/2021-4-1/end.jpg)
After learning, I found about number of workers. 'Number of workers' in Dataloader means 'how many subprocesses will be used for loading data'. And 'Shared memory' is the memory which multiple processes can access to. So workers in Dataloader use multiple processes and it requires shared memory. You could assign shared memory capacity from RAM, and rest of it is used for running other commands. So just increasing Shred memory is not a good choice. On the contrary, it could decrease executing speed. Setting appropriate number of workers and shared memory are important with this reason!
<br/>

### What should I do next?
I learned how to set an environment and just ran it. In sequence, I will analyze this code much deeper with debugger while one epoch without cuda. It would make setting Pycharm valuable at last. And I will make my own deep learning pipeline using this code. I planned to use multi GPU, but lab bros recommend simultaneously running many models than using multi GPU. Because they think it could be more efficient and using multi GPU is so complex. As I thoguht, the limites resources in our lab could be one of reasons.
<br/> 
<br/>   
<br/>  
*Always Thank You to Personal Questions, Advise, and Pointing Out My Mistakes!