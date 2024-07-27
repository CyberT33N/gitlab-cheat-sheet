# Gitlab Cheat Sheet
Cheat Sheet for gitlab with the most needed stuff..





## Guides
- GitLab: Einstieg in die Versionskontrolle (Webinar vom 16. Mai 2018) https://www.youtube.com/watch?t=1751&v=j7OHSghOvjg&feature=youtu.be

### Quick Actions
- https://docs.gitlab.com/ee/user/project/quick_actions.html














<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>


# Helm Chart
- https://github.com/CyberT33N/helm-cheat-sheet/blob/main/README.md#gitlab
























<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>

# Docker
- https://docs.gitlab.com/ee/install/docker.html
- https://hub.docker.com/r/gitlab/gitlab-ee/
```bash
sudo docker pull gitlab/gitlab-ee
```


<br><br>
<br><br>


## docker-compose.yml
```yaml
version: '3.6'
networks:
  default:
    external:
      name: localdev
services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.local.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.local.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'
```

<br><br>

.env
```bash
GITLAB_HOME=/srv/gitlab
```

<br><br>

Next run:
```bash
sudo -E docker-compose up
```

<br><br>

Get your container name:
```bash
sudo docker ps
```

<br><br>


Inspect container to get ip adress where you can visit your local gitlab
```bash
sudo docker inspect containerNameHere
# Will stand anywhere at the buttom
# "IPAddress": "xx.x.x.x"
```
  - Next add to your /etc/hosts
  ```bash
  sudo gedit /etc/hosts
  12.34.56.78 gitlab.local.com
  ```


<br><br>

Get your Gitlab password:
```
sudo docker exec -it gitlabContainerNameHere bash
cd etc/gitlab
cat initial_root_password
```
  - You username is root






<br><br>

If needed you can reset your password of the root with:
```bash
sudo docker exec -it gitlab-ce bash
gitlab-rake "gitlab:password:reset[root]"
```





<br><br>

Next add your SSH Key on Gitlab by clicking on your Avatar > Settings > SSH Keys. Create a sample project and try if you can clone it via ssh
```bash
git clone git@gitlab.local.com:websites/porn-scrap.git

# If it ask "Are you sure you want to continue connecting (yes/no/[fingerprint])?" Use fingerprint 
# In my case I got an error after this "fatal: Could not read from remote repository.". But after this I tried again to git clone and it works.

```









































<br><br>
<br><br>
_________________________________________________________
_________________________________________________________
<br><br>
<br><br>


# External Object Storage

## Minio
- **Gitlab will use for default it's own minio instance. If you want to use an external one you can check it out here**
- https://docs.gitlab.com/charts/advanced/external-object-storage/minio.html
-MinIO is an object storage server that exposes S3-compatible APIs.

MinIO can be deployed to several different platforms. To launch a new MinIO instance, follow their Quickstart Guide. Be sure to secure access to the MinIO server with TLS.

To connect GitLab to an external MinIO instance, first create MinIO buckets for the GitLab application, using the bucket names in this example configuration file.

Using the MinIO client, create the necessary buckets before use: 
```shell
mc mb gitlab-registry-storage
mc mb gitlab-lfs-storage
mc mb gitlab-artifacts-storage
mc mb gitlab-uploads-storage
mc mb gitlab-packages-storage
mc mb gitlab-backup-storage
```




















<br><br>
<br><br>
_________________________________________________________
_________________________________________________________
<br><br>
<br><br>

# Disable LFS
- Settings > General > Visibility, project features, permissions > Expand > Git Large File Storage (LFS)











































<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>


# CI/CD

## .gitlab-ci.yml
- Place this file in the root of your project.
```yaml
# If you use the docker executor on your gitlab-runner then you can set there the base image and do not have to set it like below. For more details check the gitlab-runner register section of this cheat sheet.
image: node:18.2.0

stages:
  - testing

scrap:
  stage: testing
  script:
    - npm i && npm run test
```

<br><br>
In order to use Pipelines you must install gitlab-runner. Read below..





<br><br>
<br><br>

## Properties


### only
- Use specific branch

### artifacts
- Re-use files from previous jobs

### needs 
- Access your artifacts

### image
- Define the docker image where you want to run your stages on

### scripts
- https://docs.gitlab.com/ee/ci/yaml/script.html








<br><br>
<br><br>


## Environment variables
- https://docs.gitlab.com/ee/ci/variables/predefined_variables.html
- https://docs.gitlab.com/ee/ci/variables/

<br><br>

### CI
- See if you are inside of a gitlab job
```
process.env.CI
```


<br><br>

### CI_COMMIT_REF_NAME
- See which branch you at
```
process.env.CI_COMMIT_REF_NAME
```











<br><br>
<br><br>

## Use scripts
- Because gitlab will clone your project you can include .sh files to your project and then execute them in script tag
```yaml
# .gitlab-ci.yml
stages:
  - testing

include:
  - '/gitlab-ci/npm-run-test.yml'
```


```yaml
# /gitlab-ci/npm-run-test.yml
npm-run-test:
  tags:
    - test
  stage: testing
  script:
    - bash gitlab-ci/utils/nvm/install-nvm.sh && npm i && npm run test
```

### longs command
- https://docs.gitlab.com/ee/ci/yaml/script.html#split-long-commands
```yaml
job:
  script:
    - |
      echo "First command line."
      echo "Second command line."
      echo "Third command line."
```
















<br><br>
<br><br>

## Use MongoDB in stage
- Make sure that your tests in your project use the environment variable MONGO_URI
- It is important that we use for host mongo and not localhost
```yaml
stagename:
  tags:
    - test
  stage: testing
  script:
    - node -v
    - npm -v
    - npm i
    - npm run test
  services:
    - mongo:latest
  variables:
    MONGO_URI: mongodb://test:test@mongo:27017/dbnamehere?authSource=admin
    MONGO_INITDB_ROOT_USERNAME: test
    MONGO_INITDB_ROOT_PASSWORD: test
```

















































<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>

# gitlab-runner
- https://docs.gitlab.com/runner/commands/



<br><br>
<br><br>

## docker-compose local
- This will run gitlab-ee and gitlab-runner with docker-compose on localhost
```
# .env
GITLAB_HOME=/srv/gitlab
GITLAB_ROOT_PASSWORD=root
```



```yaml
# docker-compose.yml
networks:
  default:
    external:
      name: localdev

services:
  gitlab:
    extends:
      file: services/gitlab/service.yml
      service: web

  gitlab-runner:
    depends_on:
      - gitlab
    extends:
      file: services/gitlab-runner/service.yml
      service: gitlab-runner
```



```yaml
# services/gitlab/service.yml
version: '3.6'

services:
  web:
    image: 'gitlab/gitlab-ee:latest'
    restart: always
    hostname: 'gitlab.local.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.local.com'
        # Add any other gitlab.rb configuration here, each on its own line
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - '$GITLAB_HOME/config:/etc/gitlab'
      - '$GITLAB_HOME/logs:/var/log/gitlab'
      - '$GITLAB_HOME/data:/var/opt/gitlab'
    shm_size: '256m'

```



```yaml
# services/gitlab-runner/service.yml
version: '3.6'

services:
  gitlab-runner:
    image: gitlab/gitlab-runner:alpine
    restart: unless-stopped
    volumes:
      - ./config/gitlab-runner:/etc/gitlab-runner
      - /var/run/docker.sock:/var/run/docker.sock
```

<br><br>

Then run this command to configure/install the runner:
```bash
sudo docker-compose up -d

echo 'register gitlab-runner..'
sudo docker-compose exec gitlab-runner gitlab-runner register \
--non-interactive \
--url http://gitlab.local.com/ \
--tag-list "test" \
--registration-token xxxxxxxxxxxxx \
--executor docker \
--docker-image node:18.2.0 \
--docker-network-mode localdev

# interactive mode
# sudo docker-compose exec gitlab-runner gitlab-runner register
```

<br><br>

It will ask you for details about the GitLab instance you want to attach to.
You will find this information at http://gitlab.local.com/admin/runners. 
  
You can use aswell Group Runner that will be available for all your repos inside fo your group:
- Go to your group -> CI/CD -> Runners
  - http://gitlab.local.com/groups/websites/-/runners


<br><br>
  
<br><br>
**If you specify tags then you must include them to your service otherwhise your runner will nto be triggered**. To ignore tags and force run the runner you can use:
- Project > Settings > CI/CD > Edit Button of your runner > Check "Indicates whether this runner can pick jobs without tags"
  
  
<br><br>
At this point your runner should be registered+running and gitlab-ee should be running too. 

After you registered your runner you do not have to run again the register command. You can just run this script:
```bash
echo 'docker-compose down init..'
sudo docker-compose down
echo '\n\n'

echo 'docker-compose up init..'
sudo docker-compose up -d
echo '\n\n'
```
  
  
  
  





<br><br>
<br><br>

## config.toml
You can edit this file with:
```shell
sudo docker exec -it dev-environment_gitlab-runner_1 bash
vi /etc/gitlab-runner/config.toml
```












<br><br>
<br><br>

## register
You can use non-interactive mode to register your runner:
- https://docs.gitlab.com/runner/register/#one-line-registration-command
```shell
sudo gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "PROJECT_REGISTRATION_TOKEN" \
  --executor "docker" \
  --docker-image alpine:latest \
  --description "docker-runner" \
  --maintenance-note "Free-form maintainer notes about this runner" \
  --tag-list "docker,aws" \
  --run-untagged="true" \
  --locked="false" \
  --access-level="not_protected"
```

#### executor
- You can choose between those executors which your runner should use. For local docker is recommended. Because then you can use docker base images e.q. node:18.2.0 and directly run npm i
- https://docs.gitlab.com/runner/executors/




















## protected branches
- http://gitlab.local.com/puppeteer/puppeteerservices/-/settings/repository
To protect a branch:
On the top bar, select Menu > Projects and find your project.
On the left sidebar, select Settings > Repository.
Expand Protected branches.
From the Branch dropdown list, select the branch you want to protect.
From the Allowed to merge list, select a role, or group that can merge into this branch. In GitLab Premium, you can also add users.
From the Allowed to push list, select a role, group, or user that can push to this branch. In GitLab Premium, you can also add users.
Select Protect.


