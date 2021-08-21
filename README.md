# Docker tutorial

## Useful commands

### Final app commands
- Start app container:
  ```sh
  docker-compose up -d
  ```

- Tear down container:
  *Note: use the `--volumes` flag to also remove volumes*
  ```sh
  docker-compose down
  ```


### Tutorial
- Run tutorial image:  
  *Note: No need to build - this image is downloaded*
  ```sh
  docker run -dp 80:80 docker/getting-started
  ```

- Access tutorial at port 80:  
  http://localhost

### Building/running app
- Build container image:
  ```sh
  docker build -t docker/tutorial .
  ```

- Run container image:
  ```sh
  docker run -dp 3000:3000 docker/tutorial
  ```

- Access app at port 3000:
  http://localhost:3000

- Change an image name:
  ```sh
  docker tag docker/tutorial YOUR-USER-NAME/tutorial
  ```

### Uploading/demoing app
- Log in to docker cli:
  ```sh
  docker login -u YOUR-USER-NAME
  ```

- Push an image:
  ```sh
  docker push YOUR-USER-NAME/tutorial
  ```

- Use https://labs.play-with-docker.com/ to demo the image.
  ```sh
  docker run -dp 8081:80 YOUR-USER-NAME/tutorial
  ```

### Persisting a volume
- Create volume:
  ```sh
  docker volume create todo-db
  ```

- Running a container image with a persisted volume:
  ```sh
  docker run -dp 3000:3000 -v todo-db:/etc/todos YOUR-USER-NAME/tutorial
  ```

- Inspecting a volume to see where the data is actually stored on the host:
  ```sh
  docker volume inspect todo-db
  ```

### Multi-container apps
- Create network:
  ```sh
  docker network create todo-app
  ```

- Start a MySQL container and attach to the network:
  *Note: The `--network-alias` flag is used for other apps to easily connect to the network*
  ```sh
  docker run -d \
    --network todo-app --network-alias mysql \
    -v todo-mysql-data:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=secret \
    -e MYSQL_DATABASE=todos \
    mysql:5.7
  ```

- Connect to the database and verify it connects:
  *Note: use the password from the MYSQL_ROOT_PASSWORD environment variable*
  ```sh
  docker exec -it <mysql-container-id> mysql -p
  ```

- Use the `nicolaka/netshoot` container to troubleshoot/debug network issues:
  ```sh
  docker run -it --network todo-app nicolaka/netshoot
  ```
  ```sh
  dig mysql
  ```

- Start the app in dev mode, connected to the `todo-app` network:
  ```sh
  docker run -dp 3000:3000 \
    --name todo-app-dev \
    --rm -it --detach \
    -w /app -v "$(pwd):/app" \
    --network todo-app \
    -e MYSQL_HOST=mysql \
    -e MYSQL_USER=root \
    -e MYSQL_PASSWORD=secret \
    -e MYSQL_DB=todos \
    node:12-alpine \
    sh -c "yarn install && yarn run dev"
  ```