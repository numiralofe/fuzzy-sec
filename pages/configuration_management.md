# Configuration Management
[back](../README.md)

To manage configurations and deployments there are two main repositories involved in the process:

**configurations** - This git repo would contains all application configurations and its main goal is that we  versioning applications configurations and tight them with the release process, making that on each release the current status of the repo is tagged allowing us this way to go back and forward with applications configurations.

1. Centralize all the configuration in a common place (Consul)
2. Change properties on the fly without the need of redeploying/restarting services (It depends on the usage of the properties within the code).
3. Makes configuration more portable and suitable for different environments.
4. Define properties common to every service in one place

The main idea is:

1. All the services will ask to consul for its configuration.
2. This configuration is fetched from consul on application startup based on the application name + env profiles
3. Properties are refreshed periodically to check for new changes and broadcasted across services.

We would then use gonsul to sync all changes happening on the git repo and make them available in consul k/v, Gonsul will recursively parse all the files in the git directory. Whenever Gonsul moves one level deep into a folder, the folder name is added as a Consul KV path part and as soon as it finds a file (either .json, .txt or .ini - or any other given in parameters) it will take the file name (without the extension) and append to the Consul KV path, making it a key and the file content is added as the value.

Example: Take this repository folder structure:

````
+-- prod
|   |
|   +--eu-dc1
|    |
|    +-- fuzz-sec-webservices
|    |	  |-- config.json
|    |    |-- extra-conf.yaml
|    |
|    +-- fuzz-sec-controllers
|        |-- config.json
|   +--us-dc2
|    |
|    +-- fuzz-sec-webservices
|    |	  |-- config.json
|    |    |-- extra-conf.yaml
|    |
|    +-- fuzz-sec-controllers
|        |-- config.json
|
+-- dev
    |
    +-- fuzz-sec-webservices
    |	|-- config.json
    |
    +-- fuzz-sec-controllers
        |-- config.json
````        


**deployments** - This repo contains several jobs that describe where and how applications should be deployed, delegating on the job scheduler allocations and how to deploy based on rules defined on the job.


As so the flow consist in 3 steps only:

1. Merge whatever required configuration changes to a MASTER branch on configurations repo and create create a TAG with the description of the changes to the configurations repo.

2. On the deployments repo we must set new application and configurations versions of the job files and also the desired environments/datacenter where applications should be deployed.

3. Last step (by committing the desired changes on the job available on deployments repo) will automatically trigger job execution and the deployment / release process, please bare that depending on the job stanza, it can be a canary or blue / green deployment or multi environment, on all cases we will always have a final step that will be promoting (or not) the new version of the deployed application.


Alternatively we can also setup a process that when the building step (drone / jenkins ) is completed (artifact compiling / docker image creation) call a web hook that will trigger the deployment of the builded application according to pre defined steps. 

