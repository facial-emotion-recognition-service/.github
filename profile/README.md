# facial-emotion-recognition-service
A public readme for all organization projects

# 1 Project Outline
This is a project to:
- use deep learning techniques to train a model on images of isolated human faces and recognize the emotion expressed therein
- use image segmentation techniques to isolate faces in _any_ image and perform the abovementioned classification task on each face
- provide an interface (a console app, an API, and/or a web UI) to be able to use the trained model for inference on any custom image

# 2 Project Structure
We use the following repo for this project:
- docker-compose
- ai-fer-server 
- fers-bff

# 2 Project Architecture

          +----------+
          |  model.h5<---+
          +----------+   | 
                         |
                         |
                         | python ai model + server
+--------------+      +--+-----------+
|shared-volume |<-----+ai-fer-service|
+------^-------+      +-----^--------+
       |                    |
       | upload\downlaod    |  files
+------+--+                 |
|fers-bff +-----------------+
+---^-----+
    | Rest API
    |
+---+------+
|front-end |
+----------+


## 2.1 The Model
Currently, model training is performed independently in the Jupyter notebooks under the `notebooks` folder. 
The model was trained on Google Colab with a GPU back-end. 
The resulting file containing the trained weights ("the model file") is not uploaded to the code repo as it can get fairly large in size.

#### Instructions on how to run the app:
1. Install docker & docker-compose
2. Clone the following repos
2.1 https://github.com/facial-emotion-recognition-service/docker-compose
2.2 https://github.com/facial-emotion-recognition-service/fers-bff
2.3 https://github.com/facial-emotion-recognition-service/ai-fer-server
3. Create a folder to be used as shared_volume on your host (local machine)
4. Create 4 sub folders inside the shared_volume folder
4.1 models
4.2 config
4.3 input_images
4.4 output_json
5. Modify the file docker-compose/compose/.env
The variable SHARED_VOLUME should be the path to the shared_volume folder on your host. 
6. Download file model.h5 from the shared 
6.1 [Google drive](https://drive.google.com/file/d/1Mf0__74ZPcseefAQvaK-y3_TQEyplGXX/view?usp=drive_link)
6.2 And place model.h5 in folder  shared_volume/models
7. Copy the config.json from docker-compose/config/config.json into shared_volume/config
8. Build the fers-bff docker
8.1 Open terminal/cmd in fers-bff
8.2 Run 'mvn clean install'
8.3 Run 'docker build -t esense-bff .'
9. Build the emosense docker
9.1 Open terminal/cmd in fers-bff
9.2 Run 'docker build -t esense .'
10. Run the docker compose
10.1 Open terminal/cmd in docker-compose/compose
10.2 Run 'docker-compose -f docker-compose-ai-bff.yaml up'
11. Open the browser to use/test the app. See next section.
12. Stop the dockers by open a new terminal and run command: 
12.1 'docker-compose -f docker-compose-ai-bff.yaml down'

#### Instructions on how use the app:
1. Open the browser in url
http://localhost:8080/swagger-ui/index.html
2. Use the 3 following api of OpenAI (Swagger) in order:
2.1 POST http://localhost:8080/api/v1/file/upload
2.2 GET http://localhost:8080/api/v1/predictor
2.3 GET http://localhost:8080/api/v1/report/download/{uid}

NOTE: The uid should be the same in all 3 calls. 
Do the calls in order. 
- The first ask to upload a 'face image'. It return a UID.
- The second process that image and create json report using the UID.
- The third download the json report using the UID.