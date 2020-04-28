# Team3 Project

Gitlab+Sonarqube installation and execution

## Requirements:
 - Gitlab running using docker-compose
 - Sonarqube running using docker-compose
 - Access to the VM or Baremetal system (Laptop or WK) IP address. Note this IP you will use it for configuration purposes.
 - Install Gitlab runner - https://docs.gitlab.com/runner/install/linux-manually.html
 - Sonarqube requires NodeJS for the scans, but since we are using a container NodeJS will NOT need to be installed on your system.
 - docker-compose needs to be installed

## Additional Information:
 - Gitlab ports are published at 80:80, 443:443 & 22:22
 - SonarQube port is published at 9000:9000

 - Sonarqube default login:
    - admin:admin

## Data Location:
 - All docker volumes are in default location: /var/lib/docker/volumes/
   NOTE: under each volume directory there is a _data directory which is where the actual data resides.

## Start the stack:
 - Navigate to the root directory of the repository
 - Run 'sudo docker-compose up -d'
 - For viewing services logs execute the following
    - Gitlab - sudo docker logs -f team3_gitlab_1
    - Sonarqube - sudo docker logs -f team3_sonarqube_1
 - It will take a while for both services to be fully functional, verify by trying to access them:
    - Gitlab http://<IP of your server>:80
    - Sonarqube http://<IP of your server>:9000

## Sonarqube Plugin setup:
 - When all the services are up and running copy the sonarqube gitlab plugin to /var/lib/docker/volumes/team3_sonarqube_extensions/_data/plugins
   if the plugin directory does not exists then create it. 
    - cp ./sonarqube-plugins/* /var/lib/docker/volumes/team3_sonarqube_extensions/_data/plugins
 - Restart Sonarqube:
    - Wait for sonarqube to come up, log in, go to Administration->System->Click Restart Server

## Configure Sonarqube Project:
 - Go to the Plus (+) sign on the top right -> new Project
 - Provide a project key, I like to not use spaces, copy this key and add it to your notes
 - Add a display name
 - On the following page click generate token and copy the token to your notes, we will use it later.
 - Done with the setup

## Configure Sonarqube Gitlab plugin:
 - Go to Administration->Configuration->General Settings->Gitlab
 - Add your gitlab url under 'URL to access GitLab.' Example - http://10.103.158.156/
 - Add your Gitlab User Token, to get your Gitlab User Token you need to go back to the Gitlab server.
 - On the gitlab server go to your user profile area (Top right)->Settings->Access Tokens->Create a new token (name it anything you want)
   -> copy and store the key on a scratch notes document
 - Now that you have the access token, go back to sonarqube and finish the Gitlab setup by submitting the token and clicking save.
 - Done configuring Sonarqube

## Configure the Shared Gitlab-runner:
 - The Gitlab-runner on a high level will execute a docker command to run a sonar-scaner container to perform the sonarqube scan for every
   commit based on our soon to be configured Gitlab CI pipeline.
 - Make sure you installed gitlab-runner
 - Lets get a registration token from gitlab so the runner can call back and register to the server.
 - On Gitlab go to Admin Area (Wrench on the top left, you need to be root)->Overview->Runners
 - Under the "Set up a shared Runner manually" section you will find the registration token.
 - Copy it and paste it somewhere you can access it later.
 - SSH to the system where you installed the gitlab-runner and execute the following command. Replace the place holder variables with your 
   respective values (Gitlab URL <GITLAB-URL> and Registration Token <REGISTRATION-TOKEN>).
 - sudo gitlab-runner register --name "Sonarqube Shell Scanner Runner" --url "<GITLAB-URL>" --registration-token "<REGISTRATION-TOKEN>" --executor "shell" --clone-url="<GITLAB-URL>" --tag-list=shell -n
 - The tag 'shell' is used to make sure the runner only works on pipelines that includes such tag.
 - The runner is registered as a shared runner which means that as long as the pipeline for a repository includes the the 'shell' tag then the runner will execute it.

## Perform the Part2 exercise:
 - Create a new repository on Gitlab
 - When you are done creating the repo add a new file over the UI by clicking 'New File'
 - Name the file '.gitlab-ci.yml' make sure you have the dot(.) in front of it to make it hidden.
 - This is the file that indicates Gitlab that the project wants to use the CI component and it contains the pipeline configuration.
 - Navigate our team repository and copy the contents of 'template-gitlab-ci.yml' to the text box on the UI.
 - Replace the place holder variables
     - <SONAR-TOKEN> with your Sonarqube project token created on a previous step
     - <SONARQUBE SERVER URL> with the Sonarqube server url- Example - http://10.103.158.156:9000
     - <SONAR PROJECT KEY> with your Sonarqube project key created on a previous step.
 - Here you have the template-gitlab-ci.yml file contents, just in case:

image:
  name: sonarsource/sonar-scanner-cli:latest
  entrypoint: [""]
variables:
  SONAR_TOKEN: "<SONAR-TOKEN>"
  SONAR_HOST_URL: "<SONARQUBE SERVER URL>"
  SONAR_PROJECT_KEY: "<SONAR PROJECT KEY>"
  GIT_DEPTH: 0
sonarqube-check:
  stage: test
  script:
    - sonar-scanner -Dsonar.qualitygate.wait=true -Dsonar.projectKey=$SONAR_PROJECT_KEY -Dsonar.sources=./ -Dsonar.exclusions="**/*.jpg" -Dsonar.gitlab.commit_sha=$CI_COMMIT_SHA -Dsonar.gitlab.ref_name=$CI_COMMIT_REF_NAME -Dsonar.gitlab.project_id=$CI_PROJECT_ID
  allow_failure: true
  only:
    - merge_requests
    - master
  tags:
    - shell

 - SSH to your VM/System
 - Download the Juice-Shop source code from https://github.com/bkimminich/juice-shop
    - wget https://github.com/bkimminich/juice-shop/archive/master.zip 
 - Clone the recently created repo
    - git clone <Gitlab clone link>
 - Provide your gitlab credentials, it can be root
 - copy uncompress and copy the juide-shop source code to the repo root directory
 - Add it to git
    - git add .
 - Commit your changes
    - git commit -m "Added the juice-shop source code" .
 - Push the changes to the remote master repo branch
    - git push origin master
 - Go back to the created project on the gitlab Web UI
 - Click pipelines under CI/CD
 - Wait for the pipeline to show up and when it does go to the job and observed the console executions!
 - It should fail, and when it does you can go to your commit on the UI and see a little Sonarqube at the bottom of the page
 - Also if you hover the sonarqube stage on the job you will see a little tooltip with report information

Done!


