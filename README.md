# Robin Vision - Docker image

This project provides a docker image which offers a web service to recognize known faces on images. It's based on the great [ageitgey/face_recognition](https://github.com/ageitgey/face_recognition) and [JanLoebel/face_recognition](https://github.com/JanLoebel/face_recognition) projects and just add additional web services using the Python `face_recognition`-library. This service also includes a FaceBox emulate API I create to use the FaceBox component in Home Assistant without having the limitations of the free FaceBox docker container.

On top of that I included a slightly adjusted KCFinder implementation so you can manage your face images via a browser and trigger the 'learn faces API.

....by the way, I got some questions on why this is called Robin Vision. It is because I gave my home automation setup the 'name' Robin (as per famous Dutch television show 'Bassie and Adriaan').....

## Get started

### Build the Docker image

Start by building the docker image with a defined name. This can take a while.

```bash
docker build -t robinvision .
```

### Run the Docker image

Start the image and forward port 8080 & 80.

```bash
docker run -d -p 8080:8080 -p 80:80 robinvision
```

## Features

### Register known faces

Simple `POST` an image-file to the `/addface` endpoint and provide an identifier.
`curl -X POST -F "file=@person1.jpg" http://localhost:8080/addface?name=person1`

### Read registered faces

Simple `GET` the `/faces` endpoint.
`curl http://localhost:8080/faces`

### Identify faces on image

Simple `POST` an image-file to the web service.
`curl -X POST -F "file=@person1.jpg" http://localhost:8080/`

### Delete persons
Simple `DELETE` a person from the web service
`curl -X DELETE http://localhost:8080/removeface?name=person1`

### Train the system/create encodings from all saved images
Simple `GET` the `/train` endpoint.
`curl http://localhost:8080/train`

### Enable/Disable saving of unknown faces
With this function you can enable or disable saving the images of unknown faces. Saving these faces can be handy as you can assign persons to these faces/images later via the Web Interface. As an example: you best friend is not part of the trained face recognition system yet. When your friend visits your house and you have a system running which takes a picture of your living room once every x seconds, the system will classify your friend as an unknown person. By saving the image of his face, you can later update your system by creating a folder with your friends name and moving that image to this folder (all via the web interface). Now your friend will become part of the trained system.

Simple `POST` the `/saveunknown` endpoint.
`curl -X POST "http://localhost:8080/saveunknown?enable=yes"`

### Enable/Disable scheduling of saving the trained encodings to disk
Saving the trained encodings to disk is benefitial when you restart your system. The individual images of faces does not have to be trained with a restart, the system simply loads the data from a file on the disk. On the other hand, dumping a changed dataset to disk takes time and CPU. As such I implemented a function which schedules this task at a moment in time it will not interfere with other workloads.
You can enable or disable this function. the time is always on a full hours (between 0 and 23)

Simple `POST` the `/scheduler` endpoint.
`curl -X POST "http://localhost:8080/scheduler?enable=ye&time=22"`

### FaceBox emulation

In order to be able to make use of this service from the excelent Home Assistant software I have created an API which emulates the FaceBox docker container API for the /facebox/check & /facebox/teach endpoints 
Just setup the FaceBox component in Home Assistant as per Home Assistant documentation, use the ip address of the RobinVision container as the ip address and the port is 8080. Have Fun
For reference, the API endpoint /facebox/check will only emulates the base64 json implementation (as used in the Home Assistant component). If you would like to check an individual image file you can use the example as given above under ###Identify faces on image

Facebox teach endpoint
`curl -X POST -F "file=@Ronald3.jpg" "http://localhost:8080/facebox/teach?name=Ronaldir&id=Dummy.jpg"`
(id is optional)


### Web Interface

I have added a webinterface to manage your images and to trigger the training of the system (after adding or removing imgages/persons)
The interface is based on KCFinder. The images are managed in folders under the `files` main folder. Every folder represents a person, the name of the person is the folder name. In the folders you can add/delete images of that specific person. After any change in the person/image database, please make sure to push the train button to get the system retrained. The system will retrain itself after any system (container) relaunch by default.
Just browse to `http://localhost:80`

![alt text](https://raw.githubusercontent.com/RdeLange/robinvision/master/KCFinder_RV.jpeg)


## Notes

I'm not a programming Guru. Code might be not fully optimised. But it's working -:)
Enjoy!
