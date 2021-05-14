---
title: Test Tiny Face Pytorch Code 
categories: [Privacy_Preserving_Deep-Learning]
tags:
seo:
date_modified: 2021-05-14 22:30:00 +0900
---

I planned to make pipeline and write multi GPU code. But post.doc. bros said multi GPU is complex, and it is not the best way to run a code quickly. They said simultaneously learning many versions of a code is much better. And my advised professor said me to test the result of learning. So I tested it.
<br/>

### Writing a Code Drawing Bounding Boxes with Result Test Files
![score](/assets/img/post/2021-5-14/score.jpg)  
I wrote a code drawing bounding boxes on test images with result text files. The result text files are like above. That values mean 'left coordinate', 'top coordinate', 'width', 'height', and 'score'. I drew rectangles with that coordinates with OpenCV and saved that images on new directories. The code is like bellow.
<br/>

```
import numpy as np
import cv2
import os
import os.path as osp

#Set a directory
val_set = "data/WIDER/WIDER_val/images"
val_txt = "val_results"
results_dir = "save_results"

#Open each images
folder_list = os.listdir(val_set)
for folder in folder_list:
    #If there is not save folder, make folder
    if(not os.path.isdir(results_dir + '/' + folder)):
        os.makedirs(results_dir + '/' + folder)

    img_list = os.listdir(val_set + '/' + folder)
    for img in img_list:
        a = cv2.imread(val_set + '/' + folder + '/' + img)

        #Open text files about coordinates
        filename = val_txt + '/' + folder + '/' +  img.replace(".jpg", ".txt")
        file_txt = open(filename, 'r')
        for line in file_txt:
            line = line.rstrip()
            word_list = line.split()

            #skip when the length is smaller than 4
            if(len(word_list) < 4):
                continue

            #I ADDED IT LATER
            #skip when the score is smaller than 0
            socre = int(word_list[4])
            if(score < 0):
                continue

            #Draw Rectangle
            left = int(word_list[0])
            top = int(word_list[1])
            right = left + int(word_list[2])
            bottom = top + int(word_list[3])
            a = cv2.rectangle(a, (left, top), (right, bottom), (0,0,255), 1)

        #Save image file in a new directory
        save_dir = results_dir + '/' + folder + '/' + img.replace(".jpg", "_demo") + ".jpg"
        cv2.imwrite(save_dir, a)
```

### Problem of Too Many Faces
![first_run](/assets/img/post/2021-5-14/first_run.jpg)  
But the result was like above. There were too many bounding boxes, and I thought two possibilities of this problem. First one is that I did not think about the score in the result text fiels. There could be a threshold and I tried to find the information of threshold in the paper. But there was no information about the threshold. So I thought there is not a thershold. And the other one is that it was not trained enough.
<br/>

![thinkIOU](/assets/img/post/2021-5-14/thinkIOU.jpg)  
And ultimately I could see too many overlapped bounding boxes. The IOU values(the value about overlapped area) were not considered and it means the bounding box regression were not done well. So I determined the problem is caused by the lack of learning.
<br/>

### Retraining and Failure
![device_error](/assets/img/post/2021-5-14/device_error.jpg)  
I tried to retrain it. But above device error popped up to me. It said not all tensors are on same device, GPU and CPU. Surely I wanted to run a code on a GPU.
<br/>

![cuda](/assets/img/post/2021-5-14/cuda.jpg)  
So I edited small part of the code like above. It makes brought saved model(weights) on a GPU. And I edited the code to resume tarining from latest result of model.
<br/> 

![retraining](/assets/img/post/2021-5-14/retraining.jpg)  
![no_develop](/assets/img/post/2021-5-14/no_develop.jpg)  
But I realized something is going wrong! You could see the loss values were not decreased while retraining. Even it was increased, and it means the model was overfitting and alreday trained enough. So I stopped a retraining and made bounding boxes on test images again.
<br/>

![something_wrong](/assets/img/post/2021-5-14/something_wrong.jpg)  
Yes, something was wrong. The result was similar with the first one. There were too many bounding boxes too.
<br/>


### Setting Threshold as 0 after Analyzing Loss and Success
![1-score_to_det](/assets/img/post/2021-5-14/1-score_to_det.jpg)  
![2-get_detections](/assets/img/post/2021-5-14/2-get_detections.jpg)  
![3-get_bboxes](/assets/img/post/2021-5-14/3-get_bboxes.jpg)  
I went back to the possibilty of threshold. At first I traced the meaning of the score in the result text file. And finally I found that score means the 'score_cls' corresponding with bounding box coordinates. Probability is the value after CNN layers and score is the value after assesing similarity of templates.
<br/>

![loss_over0](/assets/img/post/2021-5-14/loss_over0.jpg)  
And I focused on the fact that the loss did not decrease anymore. So I tried to find the definition of loss function in the code. And there was a threshold of socre as 0! So I made a threshold of score in the code drawing bounding boxes on test images. And the result was successful like below! (But I got a praise and a comment that I have to make ROC curve after test by my advised professor.)
<br/>

![thresh_814](/assets/img/post/2021-5-14/thresh_814.jpg)  
![thresh_277](/assets/img/post/2021-5-14/thresh_277.jpg)  
![thresh_191](/assets/img/post/2021-5-14/thresh_191.jpg)
<br/> 
<br/>   
<br/>  
*Always Thank You for your Personal Questions, Advise, and Pointing Out My Mistakes!