Attribute specific models
#########################

Approach
~~~~~~~~
- **Get a attribute labelled dataset**. And to that extent found CelebA dataset. Dataset description:

  - Has 40 attributes, such as spectacles, moustache, goatee, big-nose, blonde, black hair, big lips and so on.
  - There are about 10k different identities 'spread across' 40 attribute. So for a given attribute on a average there about 5k identities.


- **Find a baseline classifier of these attributes**. Found https://github.com/d-li14/face-attribute-prediction which has 92% accuracy on attribute classification. *This attribute classification is expected to predict/infer the attribute given an image*

  - This is a Resnet 50 network and I have trained it in hourse and reproduced the accuracies.

- **Train attribute specific Resnet classifier**. Created a feature specific dataset and Trained Resnet (pretrained on VGG facenet) through **finetuning** 

  - Wrote a python script to separate all the images per attribute, and further separted into identities. I.e. Person1 and Person2, with Attribute A1 and attribute A2, will now be in a folder strcutre as follows:
    
    - A1

      - Person1
      - Person2

    - A2

      - Person1
      - Person2

  - Trained Resnet classifier by fine tuning for all identities (person1, person2) for each attribute. I.e. first with A1; then another model for A2 etc

- **Analysis**:

  - Approach: 
    
    - First train on all identities which at-least 20 images. Thus found about 530 identities. And found classfiication accuracy on these. Please note that unlike normal face datasets here the variations in attributes is very high. This gave accuracy of about 66% with RESNET pretrained network.
      
    - Then trained on individual attributes for same number of identities i.e. about 530 identities who have a sizeable (> 20 images); and thn compared against the former.
      
    - **Observations**

      - **Without specific attribute**: Accuracy plot below:

        .. image:: ./myImages/proj/poseInvariant.png
           :scale: 75 %
           :align: center


      - **Bangs** and **Blonde hair** : Found that attribute specific classification is working to give more accuracy especially if the attribute is not too disturbing. Such as hair glasses. These gave similar accuracy of 66% 

.. _bangs:

        - Bangs
       

        .. image:: ./myImages/proj/bangs_mtcnn.png
           :scale: 75 %
           :align: center
         
        .. image:: ./myImages/proj/bangs_accuracy.png
           :scale: 75 %
           :align: center
         
        - Blonde hair        

        .. image:: ./myImages/proj/blonde_hair_mtcnn.png
           :scale: 75 %
           :align: center

        .. image:: ./myImages/proj/blonde_hair_accuracy.png
           :scale: 75 %
           :align: center


      - **Eye bags** and **Big nose** gave high accuracy of 81% ,wondering if it could be because the features are not as intrusive/obstructive

.. _eyeBags:

        - Eye Bags

        .. image:: ./myImages/proj/specificPose_eyeBags_mtcnn.png
           :scale: 75 %
           :align: center

        .. image:: ./myImages/proj/specificPose_eyeBags_accuracy.png
           :scale: 75 %
           :align: center

        - Big nose

        .. image:: ./myImages/proj/big_nose_mtcnn.png
           :scale: 75 %
           :align: center

        .. image:: ./myImages/proj/big_nose_accuracy.png
           :scale: 75 %
           :align: center

      - Work in progress...

- **Additional Attempts**:

  - Tried all of the above, without finetuning, and instead by getting a fixed-length 512 size Facenet embedding, and then running classifiers using simple SVM on SK-Learn. The results there too are comparable.


Evaluation
~~~~~~~~~~

- Attempt related to Genuine-Impostor calculation:


  - Plotted genuine impostor scores for specific features.
  
    - Feature1 - :ref:`Eyebags <eyeBags>` ( These are obscure features, which is as good as a normal face). Accuracy is shown as **98%**
    - Feature2 - :ref:`Bangs <bangs>` (These are prominent features, but even this the accuracy is shows as **90%**
  
  - *The accuracy numbers presented above are much better when compared to classify within a class using a classifier.*
  - Therefore wrote a python based API (using help of some github resources) to formalize a standard Genuine-Impostor code, with the core logic as follows:
  
    - Form pairs of images with the similar/impostor label (say, 200 images pairs with 100 labels therefore)
    - Get a simiarilty score between the pairs, to now form a mapping between similarity score and actual label.
    - Create K-fold suffles of these (say for a given K)
    - Create a range of threshold values
    - Move the threshold through the fold to get TPR/FPR and also get the thresold with best accuracy.
    - The above is additionally worked on by splitting the training and testing.

Attributes considered as prominent:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Following attributes are considered by the approach defined as follows:


Below the attributes considered by CelebA dataset. I have marked the prominent features, each of which I plan to create a separate bin for, while all the non-prominent ones will fall into another bin. Some features such as hair type (bald, black, brown) I am not considering as prominent because it is cutout by the MTCNN detector

- 5_o_Clock_Shadow
- Arched_Eyebrows
- Attractive
- Bags_Under_Eyes
- Bald
- Bangs ------------->Prominent
- Big_Lips
- Big_Nose
- Black_Hair
- Blond_Hair
- Blurry
- Brown_Hair
- Bushy_Eyebrows
- Chubby
- Double_Chin
- Eyeglasses -------->Prominent
- Goatee------------->Prominent
- Gray_Hair
- Heavy_Makeup
- High_Cheekbones
- Male
- Mouth_Slightly_Open
- Mustache----------->Proiminent
- Narrow_Eyes
- No_Beard
- Oval_Face
- Pale_Skin
- Pointy_Nose
- Receding_Hairline
- Rosy_Cheeks
- Sideburns ---------->Prominent but consusmed by Goatee..so no separate folder needed (helps in 'not' prominent binning of data)
- Smiling   ---------->Prominent
- Straight_Hair
- Wavy_Hair
- Wearing_Earrings
- Wearing_Hat
- Wearing_Lipstick
- Wearing_Necklace
- Wearing_Necktie
- Young


Experiment Jun 2020:
~~~~~~~~~~~~~~~~~~~~

- The objective here is to split Gallery and Probe such that for a given class and 5 attributes(*Bangs,Mustache,Goatee,Smile,Eyeglasses*):

  - Gallery and probe contain images **without any specific attributes** any specific attribute.
  - Gallery and probe contain images **with specific attributes**
  - Gallery **without any specific attribute** and probe **with specific attributes** 


| P1 -> Probe=Gallery=No Attribute
| P2 -> Probe=Gallery=Attribute Present
| P3 -> Probe=Attribute, Gallery=No Attribute

===============  ===============  ===================  ==================  ================  ===========  
  Attribute          State           Genuine/Imposter   ImgCount/Classes   Verification Acc   Threshold
===============  ===============  ===================  ==================  ================  ===========
   Normal             P1              100/100                6197/563            0.915           1.196
   Eyeglasses         P2              100/100                620/189             0.94            1.146
   ..........         P3              100/100                620/189             0.93            1.43 
   Goatee             P2              100/100                720/90              0.93            1.1098
   ..........         P3              100/100                720/90              0.93            1.43
   Mustache           P2              100/100                523/70              0.90            1.1099
   ..........         P3              100/100                523/70              0.93            1.43
   Sideburns          P2              100/100                676/80              0.93            1.1018
   ..........         P3              100/100                676/80              0.879           1.32
   Bangs              P2              100/100                2520/364            0.92            1.3
   ..........         P3              100/100                2520/364            0.90            1.31
   Smiling            P2              100/100                6993/567            0.93            1.118
   ..........         P3              100/100                6993/567            0.879           1.188
===============  ===============  ===================  ==================  ================  ===========

- Sample image per class of each of the above:

Eyeglasses

  .. image:: ./myImages/proj/EyeGlasses.jpg
     :scale: 50 %
     :align: center

Goatee

  .. image:: ./myImages/proj/Goatee.jpg
     :scale: 50 %
     :align: center

Mustache

  .. image:: ./myImages/proj/Mustache.jpg
     :scale: 50 %
     :align: center


Sideburns (Face detector obscuring the feature)

  .. image:: ./myImages/proj/Sideburns.jpg
     :scale: 50 %
     :align: center

Smiling

  .. image:: ./myImages/proj/Smiling.jpg
     :scale: 50 %
     :align: center

Experiment 19 Jun 2020
~~~~~~~~~~~~~~~~~~~~~~

**Objective:** Remove "normal" images which don't have any "prominent" feature counter parts. And then increase genuine impostor pair to 200/200 instead of 100/100 and report the same results as above:

1)Removal of normal: Found only the following two classes have "normal" images with no counterpart on any other "prominent" features:

          |                           105
          |                           263

2)For 200/200 Genuine/Impostor pair 

| P1 -> Probe=Gallery=No Attribute
| P2 -> Probe=Gallery=Attribute Present
| P3 -> Probe=Attribute, Gallery=No Attribute

===============  ===============  ==================  ============  ===========  =========  =========  
  Attribute          State        ImgCount/Classes    200VerifAcc   100VerifAcc   200Thres  100Thresh
===============  ===============  ==================  ============  ===========  =========  =========
   Normal             P1               6194/561       0.8775          0.915      1.194       1.196
   Eyeglasses         P2               620/189        0.95            0.94       1.222       1.146
   ..........         P3               620/189        0.8675          0.93       1.522       1.43
   Goatee             P2               720/90         0.909           0.93       1.126       1.109
   ..........         P3               720/90         0.9099          0.93       1.326       1.43
   Mustache           P2               523/70         0.937           0.90       1.012       1.109
   ..........         P3               523/70         0.925           0.93       1.191       1.43
   Sideburns          P2               676/80         0.9125          0.93       1.119       1.108
   ..........         P3               676/80         0.93            0.879      1.306       1.32
   Bangs              P2               2520/364       0.932           0.92       1.168       1.3
   ..........         P3               2520/364       0.90            0.90       1.362       1.31
   Smiling            P2               6993/567       0.9375          0.93       1.085       1.118
   ..........         P3               6993/567       0.9275          0.87       1.303       1.188
===============  ===============  ==================  ============  ===========  =========  =========

