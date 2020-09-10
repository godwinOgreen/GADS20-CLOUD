# LAB: Getting Started with Deployment Manager and Cloud Monitoring.

## Objectives:

In this lab, you will learn how to perform the following tasks:

- Create a Deployment Manager deployment.

- Update a Deployment Manager deployment.

- View the load on a VM instance using Cloud Monitoring.

## Steps:

1. Create a Deployment Manager deployment.

- Place the zone that Qwiklabs assigned you to into an environment variable called MY_ZONE.

```
export MY_ZONE=us-central1-a
```
- Download an editable Deployment Manager template:

```
gsutil cp gs://cloud-training/gcpfcoreinfra/mydeploy.yaml mydeploy.yaml
```
- Use the sed command to replace the PROJECT_ID placeholder string with your Google Cloud Platform project ID.

```
sed -i -e "s/PROJECT_ID/$DEVSHELL_PROJECT_ID/" mydeploy.yaml
```
- Use the sed command to replace the ZONE placeholder string with your Google Cloud Platform zone.

```
sed -i -e "s/ZONE/$MY_ZONE/" mydeploy.yaml
```
- View the `mydeploy.yaml file`, with your modifications.

```
cat mydeploy.yaml
```
- Build a deployment from the template.

```
gcloud deployment-manager deployments create my-first-depl --config mydeploy.yaml
```
2. Update a Deployment Manager deployment

-  Launch the nano text editor to edit the mydeploy.yaml file:

```
nano mydeploy.yaml
```
- Find the line that sets the value of the startup script, `value: "apt-get update"`, and edit it so that it looks like this:

```
value: "apt-get update; apt-get install nginx-light -y"
```
- Press Ctrl+O and then press Enter to save your edited file.
- Press Ctrl+X to exit the nano text editor.
- Enter this command to cause Deployment Manager to update your deployment to install the new startup script:

```
gcloud deployment-manager deployments update my-first-depl --config mydeploy.yaml
```
3. View the Load on a VM using Cloud Monitoring

- To open a command prompt on the `my-vm` instance, click SSH in its row in the VM instances list.

```
gcloud compute ssh my-vm
```
- In the ssh session on my-vm, execute this command to create a CPU load:

```
dd if=/dev/urandom | gzip -9 >> /dev/null &
```
This Linux pipeline forces the CPU to work on compressing a continuous stream of random data.

Create a Monitoring workspace.

- Using your VM's open SSH window and the code shown on the Agents page, install both the Monitoring and Logging agents on your project's VM.
- Make sure you have `sudo` access
- Add the agent's package repository:

```
curl -sSO https://dl.google.com/cloudagents/add-monitoring-agent-repo.sh
sudo bash add-monitoring-agent-repo.sh
sudo apt-get update
```
 Install the agent:
- List the available versions of the agent in order to select which version to install:

```
sudo apt-cache madison stackdriver-agent
```
- For production environments, you might want to pin to a major version to avoid pulling in major versions that might include backward incompatible changes. To pin to a major version, run:

```
sudo apt-get install -y 'stackdriver-agent=major-version.*'
```
- To install the latest version of the agent, run:

```
sudo apt-get install stackdriver-agent
```
- Start the agent service.

```
sudo service stackdriver-agent start
```
- To verify that the agent is working as expected, run:

```
sudo service stackdriver-agent status
```
- You can also examine the logs and ensure there are no errors:

```
grep collectd /var/log/{syslog,messages} | tail
```
- Terminate your workload generator. Return to your ssh session on my-vm and enter this command:

```
kill %1
```