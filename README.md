# gitlab-cheat-sheet
Cheat Sheet for gitlab











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

<br><br>
<br><br>

# gitlab-ci.yml
