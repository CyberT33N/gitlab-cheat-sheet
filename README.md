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
    hostname: 'gitlab.example.com'
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.example.com:8929'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - '8929:80'
      - '2224:22'
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
```
sudo docker exec -it gitlab-ce bash
gitlab-rake "gitlab:password:reset[root]"
```



























<br><br>
<br><br>
_________________________________________________________

<br><br>
<br><br>

# gitlab-ci.yml
