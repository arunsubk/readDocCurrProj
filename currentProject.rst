Attribute specific models
#########################

Approach
~~~~~~~~
- **Get a attribute labelled dataset**. And to that extent found CelebA dataset. Dataset description:

  - Has 40 attributes, such as spectacles, moustache, gotte, big-nose, blonde, black hair, big lips and so on.
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
