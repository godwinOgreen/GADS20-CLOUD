# LAB: Introduction to Containers and Docker v1.6

## Overview

Containers are a way of isolating programs or processes from each other. The primary aim of containers is to make programs easy to deploy in a way that doesn't cause them to break.

It's easy to start using containers without being familiar with the technology that makes them work.

In this lab, you learn how to build, run, and distribute an application as a Docker image.

## Steps

In your project, you have a pre-provisioned VM running Ubuntu Xenial and the necessary tools pre-installed. To connect to it:

Your instance is listed as `k8s-workshop-module-1-lab`.

1. SSH to your instance 
```
gcloud compute ssh k8s-workshop-module-1-lab
```
`This copies SSH keys to the host, and logs you in.`

2. Make sure the instance is fully provisioned. To do this, run the following command and look for the kickstart directory.

```
ls /
```
If the directory is not there, give the instance a few minutes to get fully provisioned before continuing. We've seen it take up to 8 minutes sometimes.

3. Run and Distribute Containers With Docker

The source code for this lab is available in the /kickstart folder. 
- Switch to that directory.

```
cd /kickstart
```
- List the contents.

```
ls -lh
```
`You should see a Dockerfile and web-server.py. web-server.py is a simple Python application that runs a web server which responds to HTTP requests on localhost:8888 and outputs the hostname.`

4. Install dependencies.

- Install the latest version of Python and PIP.

```
sudo apt-get install -y python3 python3-pip
```
- Install Tornado library that is required by the application.

```
pip3 install tornado
```

- Run the Python application in the background.

```
python3 web-server.py &
```

- Ensure that the web server is accessible.

```
curl http://localhost:8888
```

-Terminate the web server.

```
kill %1
```
5. Package Using Docker

Now, see how Docker can help. Docker images are described via Dockerfiles. Docker allows the stacking of images. Your Docker image will be built on top of an existing Docker image library/python that has Python pre-installed.

- Look at the Dockerfile.

```
cat Dockerfile
```

- Build a Docker image with the web server.

```
sudo docker build -t py-web-server:v1 .
```
`Be sure to include the '.' at the end of the command. This tells Docker to start looking for the Dockerfile in the current working directory.`


- Run the web server using Docker.

```
sudo docker run -d -p 8888:8888 --name py-web-server -h my-web-server py-web-server:v1
```

- Try accessing the web server again, and then stop the container.

```
curl http://localhost:8888
```
```
sudo docker rm -f py-web-server
```
The web server and all its dependencies, including the python and tornado library, have been packaged into a single Docker image that can now be shared with everyone. The py-web-server:v1 docker image functions the same way on all Docker supported OSes (OS X, Windows, and Linux).

6. Upload the Image to a Registry


The Docker image needs to be uploaded to a Docker registry to be available for use on other machines. Upload the Docker image to your private image repository in Google Cloud Registry (gcr.io).

- Add the signed in user to the Docker group so you can run docker commands without sudo and push the image to the repository as an authenticated user using the Container Registry credential helper.

```
sudo usermod -aG docker $USER
```

- Exit your the SSH session
```
exit
```

- Launch a new SSH session. This action is needed so that the group change you just made will take effect.

```
gcloud compute ssh k8s-workshop-module-1-lab
```
- In the new SSH session, return to the kickstart directory.

```
cd /kickstart
```

- Store your GCP project name in an environment variable.

```
export GCP_PROJECT=`gcloud config list core/project --format='value(core.project)'`
```

- Rebuild the Docker image with a tag that includes the registry name gcr.io and the project ID as a prefix.

```
docker build -t "gcr.io/${GCP_PROJECT}/py-web-server:v1" .
```
`Again, be sure to include the '.' at the end of the command. This tells Docker to store the image in the current working directory.`

Build the Docker image with a registry name

7. Make the Image Publicly Accessible
Google Container Registry stores its images on Google Cloud storage.

-Configure Docker to use gcloud as a Container Registry credential helper (you are only required to do this once).

```
PATH=/usr/lib/google-cloud-sdk/bin:$PATH
gcloud auth configure-docker
```
`When prompted, press ENTER.`


- Push the image to gcr.io.
```
docker push gcr.io/${GCP_PROJECT}/py-web-server:v1
```

8. Update the permissions on Google Cloud Storage to make your image repository publicly accessible.
```
gsutil defacl ch -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
```
```
gsutil acl ch -r -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
```
```
gsutil acl ch -u AllUsers:R gs://artifacts.${GCP_PROJECT}.appspot.com
```
The image is now available to anyone who has access to your GCP project.

9. Make the Image Publicly Accessible

- Run the Web Server From Any Machine
The Docker image can now be run from any machine that has Docker installed by running the following command.

```
docker run -d -p 8888:8888 -h my-web-server gcr.io/${GCP_PROJECT}/py-web-server:v1
```

- Test it on your VM instance using the curl command.

```
curl http://localhost:8888
```

- Exit the lab environment
```
exit
```