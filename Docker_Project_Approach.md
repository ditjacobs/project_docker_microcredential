# Project Micro-credential Docker #

## Set-up ##
I forked the github repo to my own GitHub account "ditjacobs".    
Then, I cloned my copy of the repo to my local environment.   
`git clone https://github.com/ditjacobs/project_docker_microcredential.git`

## Containerize and run training the machine learning model ## 
First, the recipe file `Dockerfile.train` was filled in:
* On Dockerhub, I looked for a fitting base image. I searched for python on Dockerhub and selected python:3.9-slim.
* The working directory was set to /app/models because that is where the trained model will be generated.
* The `requirements.txt` file was copied to the working directory '.'. 
* Python dependencies were installed based on the `requirements.txt` file. The option `--no-cache-dir` was added later to ensure that unneccessary temporary files are not stored. 
* The training script `train.py` was copied to the working directory.
* The command to run the training script was filled in. 

In the next step, the image for the `Dockerfile.train` was build.   
`docker build . --tag python_train:v1.0 -f Dockerfile.train`  
I checked if the image was created  
`docker images`  
Then the image was run to train the machine learning model.  
But first i checked if the data was mounted correctly by opening the container interactively.  
`docker run -it -w /app/models -v /mnt/c/Users/ditjacobs/My\ Documents/Courses/Reproducibility_in_Data_Analysis/ Project_Docker_Ditte/project_docker_microcredential:/app/models python_train:v1.0 bash`  
Then I actually run the image.  
`docker run --detach -w /app/models -v /mnt/c/Users/ditjacobs/My\ Documents/Courses/Reproducibility_in_Data_Analysis/Project_Docker_Ditte/project_docker_microcredential:/app/models python_train:v1.0`

### Github ###
Store the Dockerfile on the cloned Github repo.   
`git add Dockerfile.train`  
`git commit -m "completed the dockerfile.train with correct commands"`  
`git push`   

### Dockerhub ###
I created an account and an image repository `project_docker_microcredential` on Dockerhub.  
I logged in on my local computer  
`docker login -u ditjacobs`  
`docker tag python_train:v1.0 ditjacobs/project_docker_microcredential:python_trainv1.0`  
`docker push ditjacobs/project_docker_microcredential:python_trainv1.0`   

## Containerize and serve the machine learning model ##
The order of the instructions was changed: 
* first the base image was imported
* Then the work directory was set.
* The requirements `requirements.txt` were copied to correct work directory and installed. 
* Also, the python script `server.py` was copied to the correct work directory.
* The port that is expected inside the container is port 8080/ 
* Run the `server.py` script once the container is activated. 

The recipe was again build to an image.   
`docker build . --tag python_serve:v1.0 -f Dockerfile.infer`   

The container image was run  
`docker run --detach -w /app -v .:/app -p 8080:8080 python_serve:v1.0`  
`curl localhost:8080`  
returns 'Welcome to Docker lab'  
With some Googling, I found how I could make a prediction for specific input through the port   
For example the following input:   
`curl -X POST http://localhost:8080/predict -H "Content-Type: application/json" -d '{"input": [5.1, 3, 1.4, 5]}`  
returns:   
prediction: versicolor  

### Github ### 
`git add Dockerfile.infer`  
`git commit -m "reordered commands in recipe and tested python script "`  
`git push`  

### Dockerhub ###
I pushed the image for the python_server as well.   
`docker tag python_serve:v1.0 ditjacobs/project_docker_microcredential:python_servev1.0`  
`docker push ditjacobs/project_docker_microcredential:python_servev1.0`   

## Apptainer ## 
Build an Apptainer image on the HPC of your choice.   
I build both a pre-build image and create my own apptainer recipe.   

1. Pre-build image
To build a container from an image from Dockerhub, I searched for samtools and copied the correct path into the shell script.  
After setting to the correct directory, I build the image with the following command.  
`apptainer build --fakeroot /tmp/$USER/samtools-1.20.sif docker://staphb/samtools:1.20`  
And moved the image from the temporary folder to the correct folder in $VSC_SCRATCH.   

2. Creating my own recipe
I created a recipe that is based on the miniconda3 image. 

The def file for the recipe starts from continuumio/miniconda3.  
It uses the environment.yml to install all dependencies in the conda environment.  
The conda package `regenie` , which is used for association testing, is included in the environment.yml. 
The installation instructions to create and activate a conda environment were copied from the example def file mutatex.def. 

The shell script was adapted to contain the correct paths and files ('environment.yml' and 'association_testing.def') to build the apptainer image. 

I copied the log files locally and created a folder `logs` where all of them can be found. 
