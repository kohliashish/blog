---
title: "Building an Image Similarity Model with Keras: My Bellsprout Moment"
date: 2021-08-21T12:00:00-07:00
categories: ["Image Classification","Data Science"]
draft: false
---

{{< featured-image 
    src="https://images.pexels.com/photos/4578660/pexels-photo-4578660.jpeg"
    sizes="2400px" 
    alt="Machine Learning" 
>}}

### Introduction
In this project, I set out to explore image similarity using deep learning. Leveraging the power of Keras, I aimed to detect similarities between images using a Pokémon image dataset from Kaggle. The idea was simple: take a pre-trained ResNet50 model, process Pokémon images into their vector forms, and then compare these vectors with an image of myself. The results? You'll see soon enough.

### The Dataset and Approach
For this project, I used a Pokémon images dataset available on Kaggle. The ResNet50 model, pre-trained on the ImageNet dataset, served as the backbone of the solution, helping convert the images into feature vectors. These vectors are numerical representations of the images, allowing us to compute similarities between them using Euclidean distance.

### The Solution
Here's a quick breakdown of the process:

1. Loading and Processing Images:
  * First, I loaded the Pokémon images and an image of myself.
  * Using ResNet50, I converted each image into its vector form, a 2048-dimensional feature vector representing the image.

2. Computing Similarity:
  * To find which Pokémon my image most resembled, I computed the Euclidean distance between my image's vector and each Pokémon image's vector.
  * The Pokémon with the smallest distance to my image was deemed the most similar.

### Code:

```Python
from keras.applications.resnet50 import ResNet50
from keras.preprocessing import image
from keras.preprocessing.image import load_img
from keras.applications.resnet50 import preprocess_input, decode_predictions
from keras.models import Model
import numpy as np
from os import listdir, walk
from os.path import isfile, join
import itertools
import sys,requests
from matplotlib import pyplot as plt

# Creating some functions for processing images and computing Eucleadian distances

def getAllFilesInDirectory(directoryPath: str):
    return [(directoryPath + "/" + f) for f in listdir(directoryPath) if isfile(join(directoryPath, f))]

def predict(img_path : str, model: Model):
    img = image.load_img(img_path, target_size=(224, 224))
    x = image.img_to_array(img)
    x = np.expand_dims(x, axis=0)
    x = preprocess_input(x)
    return model.predict(x)

def calcSimilarity(self_vect, feature_vectors):
    similar: dict = {}
    keys = [k for k,v in feature_vectors.items()]
    min_dist = 10000000
    for k,v in feature_vectors.items():
       dist=np.linalg.norm(self_vect-v)
       if(dist < min_dist):
           min_dist = dist
           similar = k
    return similar

def load_image(img_path, show=False):
    img = image.load_img(img_path, target_size=(224, 224))
    img_tensor = image.img_to_array(img)
    img_tensor = np.expand_dims(img_tensor, axis=0)
    if show:
        plt.imshow(img_tensor[0]/255)                           
        plt.axis('off')
        plt.show()

def driver(self_img):
    feature_vectors: dict = {}
    model = ResNet50(weights='imagenet')
    print ("Reading images")
    for img_path in getAllFilesInDirectory("../input/pokemon-images-and-types/images/images"):
        feature_vectors[img_path] = predict(img_path,model)[0]
    self_vect = predict(self_img,model)[0]
    print ("Computing image similarity")
    result=calcSimilarity(self_vect, feature_vectors)
    print ("Your picture is most similar to : ",result)
    return result

# Downloading a picture from my Facebook
f = open('Self.jpg','wb')
r = requests.get("https://scontent.fblr1-3.fna.fbcdn.net/v/t1.0-9/67348703_10219758215926660_18338638475558912_n.jpg?_nc_cat=107&_nc_oc=AQmBneQL7Bgrr19jONnTfV8y9Er05NR1PdUQ7xb9723SOS1xZeQcJ7OBKYPSVp3_gZs&_nc_ht=scontent.fblr1-3.fna&oh=8d06ca925e759113191c0d05d81469e8&oe=5DA7A134")
f.write(r.content)
f.close()

# Output

self_img_path = "../working/Self.jpg"
result = driver(self_img_path)

load_image(self_img_path, show = True)
load_image(result, show = True)
```

### The Outcome
After running the model, I found that the Pokémon my image most resembled was none other than Bellsprout! Here's the side-by-side comparison:

{{< responsive-image src="/images/post-images/self-image.png" alt="Self Image" max-width="100%" >}}

{{< responsive-image src="/images/post-images/bellsprout.png" alt="Self Image" max-width="100%" >}}

### Conclusion
This project was a fun way to experiment with image processing and similarity detection using deep learning. While the results were amusing, they also highlighted the power of pre-trained models and their ability to generalize across different domains. Whether you're comparing Pokémon or people, Keras and deep learning have you covered!


