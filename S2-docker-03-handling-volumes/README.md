# Hands-on Docker-03 : Handling Docker Volumes

Purpose of the this hands-on training is to teach students how to handle volumes in Docker containers.

## Learning Outcomes 

At the end of the this hands-on training, students will be able to;

- explain what Alpine container is and why it is widely used.

- list available volumes in Docker.

- create a volume in Docker.

- inspect properties of a volume in Docker.

- locate the Docker volume mount point.

- attach a volume to a Docker container.

- attach same volume to different containers.

- delete Docker volumes.

## Outline

- Part 1 - Launch a Docker Machine Instance and Connect with SSH

- Part 2 - Data Persistence in Docker Containers

- Part 3 - Managing Docker Volumes

- Part 4 - Using Same Volume with Different Containers

- Part 5 - Docker Volume Behaviours

- Part 6 - Bind Mounts

- Part 7 - Anonymous Volumes

## Part 1 - Launch a Docker Machine Instance and Connect with SSH

- Launch a Docker machine on Amazon Linux 2023 AMI with security group allowing SSH connections using the [Cloudformation Template for Docker Machine Installation](../S1A-docker-01-installing-on-ec2-linux2/docker-installation-template.yml).

- Connect to your instance with SSH.

```bash
ssh -i .ssh/call-training.pem ec2-user@ec2-3-133-106-98.us-east-2.compute.amazonaws.com
```

## Part 2 - Data Persistence in Docker Containers

- Check if the docker service is up and running.

```bash
sudo systemctl status docker
docker info
```

- Run an `alpine` container with interactive shell open, and add command to run alpine shell. Here, explain what the alpine container is and why it is so popular. (Small size, Secure, Simple, Fast boot)

```bash
docker run -it alpine ash
```

**Note:** `-i` → interactive mode, `-t` → terminal, `-it` → interactive terminal

- Display the os release of the alpine container.

```bash
cat /etc/os-release
```

- Create a file named `short-life.txt` under `/home` folder

```bash
cd home && touch short-life.txt && ls
```

- Exit the container and return to ec2-user bash shell.

```bash
exit
```

- Show the list of all containers available on Docker machine.

```bash
docker ps
docker ps -a
```

- Start the alpine container and connect to it.

```bash
docker start 2ae 
docker ps
docker exec -it 2ae ash
```

**Note:** Container outbound rules are allow anywhere by default but inbound rules are totally deny. You can go from container everywhere but for inbound you must publish the port explicitly.

- Show that the file `short-life.txt` is still there, and explain why it is there. (Container holds it data until removed).

```bash
ls /home 
```

- Exit the container and return to ec2-user bash shell.

```bash
exit
```

- Remove the alpine container.

```bash
docker rm -f 2a
docker ps -a
```

**Alternative:**

```bash
docker stop 2a && docker rm 2a
```

**Note:** `docker rm -f 2a` = `docker stop 2a && docker rm 2a`

- Show the list of all containers, and the alpine container is gone with its data.

```bash
docker ps -a
```

## Part 3 - Managing Docker Volumes

- Explain why we need volumes in Docker.

- List the volumes available in Docker, since not added volume before list should be empty.

```bash
docker volume --help
```

- Create a volume named `my-vol`.

```bash
docker volume create my-vol
```

- List the volumes available in Docker, should see local volume `my-vol` in the list.

```bash
docker volume ls
```

- Show details and explain the volume `my-vol`. Note the mount point: `/var/lib/docker/volumes/my-vol/_data`.

```bash
docker volume inspect my-vol
```

- List all files/folders under the mount point of the volume `my-vol`, should see nothing listed.

https://docs.docker.com/engine/storage/volumes/

```bash
sudo ls -al /var/lib/docker/volumes/my-vol/_data 
```

- Run a `alpine` container with interactive shell open, name the container as `babak`, attach the volume `my-vol` to `/test` mount point in the container, and add command to run alpine shell. Here, explain `--volume` and `-v` flags.

```bash
docker run -it --name babak -v my-vol:/test alpine ash
```

- List files/folder in `babak` container, show mounting point `/test`, and explain the mounted volume `my-vol`.

```bash
ls
```

- Create a file in `babak` container under `/test` folder.

```bash
cd test && echo "This file is created in the container babak" > i-will-persist.txt
```

- List the files in `/test` folder, and show content of `i-will-persist.txt`.

```bash
ls && cat i-will-persist.txt
```

- Exit the `babak` container and return to `ec2-user` bash shell.

```bash
exit
```

- Show the list of all containers available on Docker machine.

```bash
docker ps -a
```

- Remove the `babak` container.

```bash
docker rm babak
```

- Show the list of all containers, and the `babak` container is gone.

```bash
docker ps -a
```

- List all files/folders under the volume `my-vol`, show that the file `i-will-persist.txt` is there.

```bash
sudo ls -al /var/lib/docker/volumes/my-vol/_data && sudo cat /var/lib/docker/volumes/my-vol/_data/i-will-persist.txt
```

## Part 4 - Using Same Volume with Different Containers

- Run a `alpine` container with interactive shell open, name the container as `babak2`, attach the volume `my-vol` to `/test2` mount point in the container, and add command to run alpine shell.

```bash
docker volume ls
docker run -it --name babak2 -v my-vol:/test2 alpine ash
```

- List the files in `/test2` folder, and show that we can reach the file `i-will-persist.txt`.

```bash
ls -l /test2 && cat /test2/i-will-persist.txt
```

- Create an another file in `babak2` container under `/test2` folder.

```bash
cd test2 && echo "This is a file of the container babak2" > loadmore.txt
```

- List the files in `/test2` folder, and show content of `loadmore.txt`.

```bash
ls && cat loadmore.txt
```

- Exit the `babak2` container and return to ec2-user bash shell.

```bash
exit
```

- Run a `ubuntu` container with interactive shell open, name the container as `babak3`, attach the volume `my-vol` to `/test3` mount point in the container, and add command to run bash shell.

```bash
docker run -it --name babak3 -v my-vol:/test3 ubuntu bash
```

- List the files in `/test3` folder, and show that we can reach the all files created earlier.

```bash
ls -l /test3
```

- Create an another file in `babak3` container under `/test3` folder.

```bash
cd test3 && touch file-from-3rd.txt && ls
```

- Exit the `babak3` container and return to ec2-user bash shell.

```bash
exit
```

- Run an another `ubuntu` container with interactive shell open, name the container as `babak4`, attach the volume `my-vol` to `/test4` mount point in the container, and add command to run bash shell.

```bash
docker run -it --name babak4 -v my-vol:/test4 ubuntu bash
```

- List the files in `/test4` folder, and show that we can reach the all files created earlier.

```bash
ls -l /test4
```

- Exit the `babak4` container and return to ec2-user bash shell.

```bash
exit
```

- List all containers.

```bash
docker ps -a
```

- Delete `babak2` and `babak3` containers.

```bash
docker rm babak2 babak3
```

- List all containers.

```bash
docker ps -a
```

- Delete all volumes.

```bash
docker volume prune
```

## Part 5 - Docker Volume Behaviours

There are four situations to determine volume behaviours.

### Situation-1:

|No    | Situation   | Behaviour |
| ---- | ----------- | ------------ |
| 1    | If the volume is empty and the target directory is empty. | The target directory is going to be empty inside the container. |

![situation 1](situation-1.png)

- Create an empty volume named `empty-vol`.

```bash
docker volume create empty-vol
```

- List the volumes and show that `empty-vol` is empty.

```bash
docker volume ls
sudo ls /var/lib/docker/volumes/empty-vol/_data
```

- Run the `ondiacademy/hello-ondia` container with interactive shell open, name the container as `try1`, attach the volume `empty-vol` to `/hello-ondia` mount point in the container, and show that  the target directory is empty.

```bash
docker run -it --name try1 -v empty-vol:/hello-ondia ondiacademy/hello-ondia sh
ls
cd hello-ondia && ls
```

- Exit from the container.

```bash
exit
```

### Situation-2:

|No    | Situation   | Behaviour |
| ---- | ----------- | ------------ |
| 2    | If the volume is empty and the target directory is not empty. | All the content in the target directory is going to be copied to the volume. |

![situation 2](situation-2.png)

- List the volumes and show that `empty-vol` is empty.

```bash
docker volume ls
sudo ls /var/lib/docker/volumes/empty-vol/_data
```

- Run the `ondiacademy/hello-ondia` container with interactive shell open, name the container as `try2`, attach the volume `empty-vol` to `/var` mount point in the container, and show that the content of target directory is copied to the volume.

```bash
docker run -it --name try2 -v empty-vol:/var ondiacademy/hello-ondia sh
ls
cd var && ls
```

- List all files/folders under the volume `empty-vol`, show that all content of `/var` directory is copied to the `empty-vol`.

```bash
exit
sudo ls /var/lib/docker/volumes/empty-vol/_data
```

- Run the following command and show the output. (We create a file in the volume)

```bash
sudo touch /var/lib/docker/volumes/empty-vol/_data/i-am-volume.txt
sudo ls /var/lib/docker/volumes/empty-vol/_data
```

- Run the `ondiacademy/hello-ondia` container with interactive shell open, name the container as `try3`, attach the volume `empty-vol` to `/var` mount point in the container, and show that we can see the file `i-am-volume.txt` in the target directory.

```bash
docker run -it --name try3 -v empty-vol:/var ondiacademy/hello-ondia sh
cd var && ls
```

- Exit from the container.

```bash
exit
```

### Situation-3:

|No    | Situation   | Behaviour |
| ---- | ----------- | ------------ |
| 3    | If the volume is not empty and the target directory is empty. | All the content in the volume is going to be copied to the target directory. |

![situation 3](situation-3.png)

- Remove the all volumes.

```bash
docker volume prune -a
```

- Run the following command and show the output.

```bash
docker run --rm -v empty-vol:/app ondiacademy/hello-ondia ls app
```

- List all files/folders under the volume `empty-vol`, show that the file `app.py` is there.

```bash
sudo ls /var/lib/docker/volumes/empty-vol/_data
```

### Situation-4:

|No    | Situation   | Behaviour |
| ---- | ----------- | ------------ |
| 4    | If the volume is not empty. | There will be just the files inside the volume regardless of the target directory is full or empty. |

![situation 4](situation-4.png)

- Create a volume name `full-vol` and create a file in it.

```bash
docker volume create full-vol
sudo touch /var/lib/docker/volumes/full-vol/_data/full.txt
```

- List all files/folders under the volume `full-vol`, show that the file `full.txt` is there.

```bash
sudo ls /var/lib/docker/volumes/full-vol/_data
```

- Run the `ondiacademy/hello-ondia` container with interactive shell open, name the container as `try4`, attach the volume `full-vol` to `/hello-ondia` mount point in the container, and show that we just see the files inside volume regardless of  the target directory is full or empty.

```bash
docker run -it --name try4 -v full-vol:/hello-ondia ondiacademy/hello-ondia sh
ls
cd hello-ondia && ls
```

- Exit from the container.

```bash
exit
```

- Remove all volumes and containers and list them.

```bash
docker container prune
docker volume prune -a
docker volume ls
docker ps
```

## You can attach multiple volumes to a container

```bash
docker volume create volume1
docker volume create volume2
docker volume create volume3
docker volume ls
```

```bash
docker run -it --name babak-test -v volume1:/hello-ondia -v volume2:/app -v volume3:/logs ondiacademy/hello-ondia sh
exit
```

```bash
sudo ls /var/lib/docker/volumes/volume1/_data
```

```bash
docker container prune
docker volume prune -a
docker volume ls
docker ps
```

## Part 6 - Bind Mounts

We can attach any folder to container not only volume. It is same with volume mounting.

**Note:** Volume always works with "sudo"

- Run the `nginx` container at the detached mode, name the container as `nginx-default`, and open <public-ip> on browser and show the nginx default page.

```bash
docker run -d --name nginx-default -p 80:80 nginx
```

- Add a security rule for protocol HTTP port 80 and show Nginx Web Server is running on Docker Machine.

```text
http://<public-ip>:80
```

- Attach the `nginx` container, show the index.html in the /usr/share/nginx/html directory.

```bash
docker exec -it nginx-default bash
cd /usr/share/nginx/html
ls
cat index.html
exit
```

- Create a folder named webpage, and an index.html file.

```bash
mkdir webpage && cd webpage
echo "<h1>Welcome to Docker Lesson</h1>" > index.html
```

- Run the `nginx` container at the detached mode, name the container as `nginx-new`, attach the directory `/home/ec2-user/webpage` to `/usr/share/nginx/html` mount point in the container, and open <public-ip> on browser and show the web page.

```bash
docker run -d --name nginx-new -p 8080:80 -v /home/ec2-user/webpage:/usr/share/nginx/html nginx
```

- Add a security rule for protocol HTTP port 8080 and show Nginx Web Server is running on Docker Machine.

```text
http://<public-ip>:8080
```

- Attach the `nginx` container, show the index.html in the /usr/share/nginx/html directory.

```bash
docker exec -it nginx-new bash
cd /usr/share/nginx/html
ls
cat index.html
exit
```

- Add `<h2>This is added for docker volume lesson</h2>` line to index.html in the /home/ec2-user/webpage folder and check the web page on browser.

```bash
cd /home/ec2-user/webpage
echo "<h2>This is added for docker volume lesson</h2>" >> index.html
```

- Check the web page on browser.

- Remove the containers.

```bash
docker rm -f nginx-default nginx-new
```

- Remove the volumes.

```bash
docker volume prune -f
```

## Part 7 - Anonymous Volumes

- Explain what an anonymous volume is and why to use it

> Anonymous volumes are given a random name that's guaranteed to be unique within a given Docker host. Just like named volumes, anonymous volumes persist even if you remove the container that uses them, except if you use the `--rm` flag when creating the container, in which case the anonymous volume associated with the container is destroyed.

> Why do we need anonymous volumes?
> - **Avoiding Container Layer Bloat:** The container's writable layer is part of the storage driver, and constantly writing large or temporary files there can slow things down or lead to performance issues.
> - **Write-Heavy Applications in Read-Only Containers:** When running containers with read-only root filesystems (--read-only), some applications still need write access to temporary directories (e.g., /tmp). Instead of making the whole container writable, an anonymous volume provides a temporary write location.
> - **Sharing Data Between Container Restarts (Short-Lived Persistence):** If a container restarts, the writable layer is wiped, but anonymous volumes can persist across restarts within the same container lifecycle.
> - **Ensuring Data Isn't Stored in the Container Image:** If an application writes logs or temp files, you might accidentally include them in the image when committing changes. Using an anonymous volume ensures these files never get stored inside the image layer.

- If you create multiple containers consecutively that each use anonymous volumes, each container creates its own volume. Anonymous volumes aren't reused or shared between containers automatically. To share an anonymous volume between two or more containers, you must mount the anonymous volume using the random volume ID.

- Run the following command to start a **BusyBox** container with an anonymous volume:

```bash
docker run -dit --name cont1 -v /data busybox sh
```

- Check the available volumes and see there is a random-named volume is created.

```bash
docker volume ls
```

```text
DRIVER    VOLUME NAME
local     7cbbc785f43581c3300f0ad0bc0dccb61506582df2e56076eee34ab9d3a9bb63
```

- Attach to the running container and create a file in `/data`

```bash
docker exec -it cont1 sh
```

- Once inside the container, run:

```bash
echo "Hello, anonymous volume!" > /data/message.txt
exit
```

```bash
docker inspect cd0c648575720a69a991c203df304eefe06f901ae36647e1b4205aaf41541d66
sudo ls /var/lib/docker/volumes/cd0c648575720a69a991c203df304eefe06f901ae36647e1b4205aaf41541d66/_data
```

- Restart the container and check if the data persists:

```bash
docker restart cont1 
docker exec -it cont1 sh -c "cat /data/message.txt"
```

- The message should still be there because anonymous volumes persist across container restarts.

- Now, stop and remove the container:

```bash
docker stop cont1 && docker rm cont1
```

- Check if the volume persists:

```bash
docker volume ls
```

```text
DRIVER    VOLUME NAME
local     7cbbc785f43581c3300f0ad0bc0dccb61506582df2e56076eee34ab9d3a9bb63
```

- We still see the volume because anonymous volumes persist even if you remove the container that uses them; however, if you use `--rm` flag when you create the container, the anonymous volume is also deleted when the container stops.

- Create another container that uses an anonymous volume.

```bash
docker run -dit --rm --name cont2 -v /data busybox sh
```

**Note:** `--rm` removes only anonymous volumes automatically, not other volumes.

- Check the available volumes and see if an anonymous volume has been created.

```bash
docker volume ls
```

```text
DRIVER    VOLUME NAME
local     390ed0181d8e0c3c7a37eebbd5c3f3b4471feff146e0858b007cccd2e5634fac
```

- Stop the container and check the volumes.

```bash
docker stop cont2
```

```bash
docker ps -a
```

```text
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

```bash
docker volume ls
```

```text
DRIVER    VOLUME NAME
```

- The anonymous volume is automatically removed along with the container.

## Additional Notes

**stdout:**

```bash
docker run -p 80:80 nginx
docker logs ce
```

**docker exec vs docker attach:**

- `docker exec` → Used to run a new command or open a new terminal inside a running container.

```bash
docker run -p 80:80 nginx
exit
docker ps -a
docker start 5ac
docker exec -it 5ac touch file1
docker exec -it 5ac ls file1
```

- `docker attach` → Used to connect to an existing process of a running container.

```bash
docker attach 5ac
```

**Attention!!** When you exit from attach, the container process finishes so the container also stops.
