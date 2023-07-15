# image-classification
Deep Learning model with TensorFlow used for Image classification 

TKinter for python GUI

CNN for neural network 

Dataset- Apple Vs Tomato from Kaggle
_____________________________________________
import os

base_dir ='./apples_or_tomato_filtered.zip'

print("Contents of base directory:")
print(os.listdir(base_dir))

print("\nContents of train directory:")
print(os.listdir(f'{base_dir}/train'))

print("\nContents of test directory:")
print(os.listdir(f'{base_dir}/test'))
________________________________________________
import os

train_dir = os.path.join(base_dir, 'train')
test_dir = os.path.join(base_dir, 'test')

# Directory with training apple/tomato pictures
train_apple_dir = os.path.join(train_dir, 'apples')
train_tomato_dir = os.path.join(train_dir, 'tomatoes')
# Directory with test apple/tomato pictures
test_apple_dir = os.path.join(test_dir, 'apples')
test_tomato_dir = os.path.join(test_dir, 'tomatoes')
______________________________________________________
train_apple_fnames = os.listdir( train_apple_dir )
train_tomato_fnames = os.listdir( train_tomato_dir )

print(train_apple_fnames[:10])
print(train_tomato_fnames[:10])
______________________________________________________
print('total training apple images :', len(os.listdir(train_apple_dir ) ))
print('total training tomato images :', len(os.listdir(train_tomato_dir ) ))

print('total test apple images :', len(os.listdir( test_apple_dir ) ))
print('total test tomato images :', len(os.listdir( test_tomato_dir ) ))
_______________________________________________________________________________
%matplotlib inline

import matplotlib.image as mpimg
import matplotlib.pyplot as plt

# Parameters for our graph; we'll output images in a 4x4 configuration
nrows = 4
ncols = 4

pic_index = 0 # Index for iterating over images
___________________________________________________________________________________
# Set up matplotlib fig, and size it to fit 4x4 pics
fig = plt.gcf()
fig.set_size_inches(ncols*4, nrows*4)

pic_index+=8

next_apple_pix = [os.path.join(train_apple_dir, fname)
                for fname in train_apple_fnames[ pic_index-8:pic_index]
               ]

next_tomato_pix = [os.path.join(train_tomato_dir, fname)
                for fname in train_tomato_fnames[ pic_index-8:pic_index]
               ]

for i, img_path in enumerate(next_cat_pix+next_dog_pix):
  # Set up subplot; subplot indices start at 1
  sp = plt.subplot(nrows, ncols, i + 1)
  sp.axis('Off') # Don't show axes (or gridlines)

  img = mpimg.imread(img_path)
  plt.imshow(img)

plt.show()
_______________________________________________________________________
import tensorflow as tf

model = tf.keras.models.Sequential([
    # Note the input shape is the desired size of the image 100x100 with 3 bytes color
    tf.keras.layers.Conv2D(32, (3,3), activation='relu', input_shape=(100, 100,3)),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Conv2D(64, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),
    tf.keras.layers.Conv2D(128, (3,3), activation='relu'),
    tf.keras.layers.MaxPooling2D(2,2),

    # Flatten the results to feed into a DNN
    tf.keras.layers.Flatten(),
    # 512 neuron hidden layer
    tf.keras.layers.Dense(512, activation='relu'),
    tf.keras.layers.Dropout(0.5),
    tf.keras.layers.Dense(256,activation="relu"),
    tf.keras.layers.Dropout(0.5),
    # Only 1 output neuron. It will contain a value from 0-1 where 0 for 1 class ('apple') and 1 for the other ('tomato')
    tf.keras.layers.Dense(1, activation='sigmoid')
])
print(model.summary())
___________________________________________________________________________
model.compile(optimizer='adam',
              loss='binary_crossentropy',
              metrics = ['accuracy'])
___________________________________________________________________________
from tensorflow.keras.preprocessing.image import ImageDataGenerator

# All images will be rescaled by 1./255.
# Apply data augmentation
train_datagen = ImageDataGenerator(
      rescale=1./255,
      rotation_range=50,
      width_shift_range=0.25,
      height_shift_range=0.25,
      shear_range=0.3,
      zoom_range=0.3,
      brightness_range=(0.8, 1.2),
      horizontal_flip=True,
      fill_mode='nearest')


test_datagen  = ImageDataGenerator( rescale = 1.0/255. )

# --------------------
# Flow training images in batches of 64 using train_datagen generator
# --------------------
train_generator = train_datagen.flow_from_directory(train_dir,
                                                    batch_size=64,
                                                    class_mode='binary',
                                                    target_size=(100, 100))
# --------------------
# Flow validation images in batches of 64 using test_datagen generator
# --------------------
test_generator =  test_datagen.flow_from_directory(test_dir,
                                                         batch_size=64,
                                                         class_mode  = 'binary',
                                                         target_size = (100, 100))
____________________________________________________________________________________
BATCH_SIZE= 64
history = model.fit(
            train_generator,
            epochs=120,
            validation_data=test_generator,
            batch_size=BATCH_SIZE,
            callbacks=[tf.keras.callbacks.LearningRateScheduler(lambda epoch, lr: lr if epoch < 60 else lr * tf.math.exp(-0.05))],
            verbose=1
)
_________________________________________________________________________________________
import matplotlib.pyplot as plt

# Plot the model results
acc = history.history['accuracy']
val_acc = history.history['val_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs = range(len(acc))

plt.plot(epochs, acc, 'r', label='Training accuracy')
plt.plot(epochs, val_acc, 'b', label='test accuracy')
plt.title('Training and test accuracy')

plt.figure()

plt.plot(epochs, loss, 'r', label='Training Loss')
plt.plot(epochs, val_loss, 'b', label='tset Loss')
plt.title('Training and test loss')
plt.legend()

plt.show()
______________________________________________________________________________________
import numpy as np

from google.colab import files
from tensorflow.keras.utils import load_img, img_to_array

uploaded=files.upload()

for fn in uploaded.keys():

  # predicting images
  path='/content/' + fn
  img=load_img(path, target_size=(100, 100))

  x=img_to_array(img)
  x /= 255
  x=np.expand_dims(x, axis=0)
  images = np.vstack([x])

  classes = model.predict(images, batch_size=10)

  print(classes[0])

  if classes[0]>0.5:
    print(fn + " is a tomato")
  else:
    print(fn + " is a apple")
_________________________________________________________________________________
model.save("model2.h5")
_________________________________________________________________________________
# python GUI
import tkinter as tk
from tkinter import filedialog
from tkinter import *
from PIL import ImageTk, Image
import numpy as np
from keras.preprocessing.image import img_to_array
#load the trained model to classify the images
from keras.models import load_model
from keras.preprocessing.image import load_img
model = load_model('model2.h5')

#initialise GUI
top=tk.Tk()
top.geometry('800x600')
top.title('Image Classification')
top.configure(background='red')
label=Label(top,background='red', font=('arial',15,'bold'))
sign_image = Label(top)
def classify(file_path):
    global label_packed
    image = Image.open(file_path)
    image = image.resize((100, 100))
    image = img_to_array(image)
    image = np.expand_dims(image, axis=0)
    image /= 255
    sign = model.predict(image)

    if sign[0] > 0.5:
        sign = "Tomato"
    else:
        sign = "Apple"
    print(sign[0])
    label.configure(foreground='white', text=sign)
def show_classify_button(file_path):
    classify_b=Button(top,text="Classify Image",
   command=lambda: classify(file_path),padx=10,pady=5)
    classify_b.configure(background='red', foreground='white',
                         font=('arial',10,'bold'))
    classify_b.place(relx=0.79,rely=0.46)
def upload_image():
    try:
        file_path=filedialog.askopenfilename()
        uploaded=Image.open(file_path)
        uploaded.thumbnail(((top.winfo_width()/2),(top.winfo_height()/2)))
        im=ImageTk.PhotoImage(uploaded)
        sign_image.configure(image=im)
        sign_image.image=im
        label.configure(text='')
        show_classify_button(file_path)
    except:
        pass
upload=Button(top,text="Upload an image",command=upload_image,
              padx=10,pady=5)
upload.configure(background='red', foreground='white',
                 font=('arial',10,'bold'))
upload.pack(side=BOTTOM,pady=50)
sign_image.pack(side=BOTTOM,expand=True)
label.pack(side=BOTTOM,expand=True)
heading = Label(top, text="Image Classification",pady=20, font=('arial',20,'bold'))
heading.configure(background='red',foreground='black')
heading.pack()
top.mainloop()
