# facial-emotion-recognition-service
A public readme for all organization projects

## 1 Project Outline
This is a project to:
- use deep learning techniques to train a model on images of isolated human faces and recognize the emotion expressed therein
- use image segmentation libraries to isolate faces in _any_ image and perform the abovementioned classification task on each face
- use face clustering libraries to group faces that belong to the same person together and determine the proportion of images over which each person exhibited each detected emotion
- provide an interface (a console app, an API, and/or a web UI) to be able to expose the above functionalities in a user-friendly manner

## 2 Project Structure
We use the following repos for this project:
- docker-compose
- ai-fer-server
- facial-extraction
- fers-bff

## 3 Project Architecture
```
                          +----------+
                          | model.h5 <-------------+
                          +----------+             | 
                                                   |
                                                   | python ai model + server
+-------------------+     +---------------+     +--+-------------+  
| facial-extraction +-----> shared-volume <-----+ ai-fers(erver) |  
+----^--------------+     +--^------------+     +---^------------+  
     |                       |                      |
     |       upload/download | files                | 
     |                  +----+-----+                |
     +----------------- + fers-bff +----------------+
                        +----^-----+
                             | Rest API
                             |
                       +-----+-----+
                       | front-end |
                       +-----------+
```

### 3.1 The "AI" Model
Currently, model training is performed independently in Jupyter Notebook. We plan to bring it in as a separate repo. 
The model was trained on Google Colab with a GPU back-end. 
The resulting file containing the trained weights ("the model file") is not uploaded to this repo as it can get fairly large in size.

## 4 Instructions on How to Set Up the App
1. Install docker & docker-compose.
2. Clone the following repos:  
  2.1. https://github.com/facial-emotion-recognition-service/docker-compose  
  2.2. https://github.com/facial-emotion-recognition-service/fers-bff  
  2.3. https://github.com/facial-emotion-recognition-service/ai-fer-server
3. Create a folder to be used as a shared volume on your host (local machine). We will refer to this as `shared_volume` going forward.
4. Create the following sub-folders inside the `shared_volume` folder:  
  4.1. `models`  
  4.2. `config`  
  4.3. `input_images`  
  4.4. `output_json`
5. Modify the file `docker-compose/compose/.env`.  
   The variable `SHARED_VOLUME` should be the path to the `shared_volume` folder on your host. 
6. Download the file `model.h5` from the shared [Google drive](https://drive.google.com/file/d/1Mf0__74ZPcseefAQvaK-y3_TQEyplGXX/view?usp=drive_link) and place it in `shared_volume/models`.
7. Copy the `config.json` from `docker-compose/config/config.json` into `shared_volume/config`.
8. Build the `fers-bff` docker image.  
  8.1. Open terminal/cmd in `fers-bff`.  
  8.2. Run `mvn clean install`.  
  8.3. Run `docker build -t fers-bff .`.
9. Build the `ai-fers` docker image.  
  9.1. Open terminal/cmd in `ai-fer-server`.  
  9.2. Run `docker build -t ai-fers .`.
10. Build the `facial-extraction` docker image.  
  10.1. Open terminal/cmd in `facial-extraction`.  
  10.2. Run `docker build -t facial-extraction .`.  
11. Run docker-compose.  
  11.1. Open terminal/cmd in `docker-compose/compose`.  
  11.2. Run `docker-compose -f docker-compose-ai-bff.yaml up`.
12. Open the browser to use/test the app. See next section.
13. Shut down the docker containers by opening a new terminal and running the command: `docker-compose -f docker-compose-ai-bff.yaml down`

## 5 Instructions on How to Use the App
### 5.1 Emotion Recognition
1. Open the browser and navigate to the url http://localhost:8080/swagger-ui/index.html
2. Use the following 3 API endpoints of OpenAPI (Swagger) in order:  
  2.1. `POST` http://localhost:8080/api/v1/file/upload  
  2.2. `GET` http://localhost:8080/api/v1/predictor/image/process/{uid}  
  2.3. `GET` http://localhost:8080/api/v1/report/download/{uid}

**NOTE:** The UID should be the same across all 3 calls. Do the calls in order. 
- The first call asks to upload an image of an isolated face. It returns a UID.
- The second one processes that image and creates a JSON report using the UID.
- The third downloads the JSON report using the UID.

### 5.2 Face Extraction
1. Open the browser and navigate to the url http://localhost:8080/swagger-ui/index.html
2. Use the following 3 API endpoints of OpenAPI (Swagger) in order:  
  3.1. `POST` http://localhost:8080/api/v1/file/upload  
  3.2. `POST` http://localhost:8080/api/v1/convert/file/format  
  3.3. `GET` http://localhost:8080//api/v1/image/{uid}/faces  

**NOTE:** Do the calls in order.  
- The first call asks to upload an image containing zero or more faces. It returns a UID.
- The second call, which is optional, converts an image from one format to another. Use it if the image is not in the expected format (JPEG). It takes the source file UID from the first call, a source file format (e.g. `"gif"`) and a target file format (set it to `"jpg"` for the next step to work). It returns a new UID for the converted file.  
- The third call takes the UID of a JPEG file and creates a JSON report with a list of `{x, y, w, h}` coordinates, with each (x, y) being the top-left corner and (w, h) being the width and height of a detected face.
