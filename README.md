# Employee-Management-App
## Task-1: Conatinerise the Rails App
- Create Dockerfile in the src directory which is used to generate an image of the Rails App (with Base layer of Ruby 3.0.5 and all required dependencies) which listens on port `3000` <br>
![image](https://user-images.githubusercontent.com/95867745/230729517-7490ee0c-c2dc-4908-b138-96aa75cf4be3.png)
- To generate the image run the following command in the src directory: `docker build -t emp_mgmt .`
## Task-2: Launch the application in a docker conatainer. Launch a separate container for the databse and ensure that the wo containers are able to cennect.
- First and foremost create a network within docker network within which the required containers will run. To do so run: `doceker network create <network_name>`
- Run the mysql continer: `docker run --name <container_name> -d --net <network_name> -e MYSQL_ROOT_PASSWORD=<passowrd> mysql:latest`
- Mysql runs on the specified network on the port `3306`
- Configue the Rails app <br>
![image](https://user-images.githubusercontent.com/95867745/230729238-5b5f02ff-51f8-43a6-a1c6-22612bdeddd3.png)
- Build and run the Rails Application image using <br/>`docker build -t emp_mgmt .` and then <br/>`docker run --name Rails -p 8080:3000 emp_mgmt`
- The Application can be accessed on `localhost:8080`
## Tasks 3 to 7:
- Create the `docker-compose.yml` file<br>
![image](https://user-images.githubusercontent.com/95867745/230729570-980c7741-1ffb-4aa6-9833-c03800601dbb.png)
- Here three services are specified the sql conatiner, the rails applicaion container and the nginx container
- For the database persistance a *named volume* `sql-db` is mounted which stores the database changes on the local maching and loads them into the container at startup
- Write the nginx config file: <br>
![image](https://user-images.githubusercontent.com/95867745/230729755-f3e7d700-efde-4e88-a090-e933145e7ddf.png)
- `limit_req_zone` and `limit req` are directives to implement rate limiting
- `$binary_remote_addr zone=limit:10m` spicifies a rate limiting zone `mylimit` which stores 10MB of IP addresses in binary.
- `rate = 10r/s` specifies a limit of 10 requests per second
- The `nginex` server listens on the port 80 for both `IPv4` and `IPv6` addresses.
```
location / {                      # For all requests made to / i.e., all requests 
  limit_req zone=mylimit;         # enforces the rate limiting
  proxy_pass http://Rails:3000/;  # reverse proxy to the Rails Application
  }
``` 
- **Note: When running Multiple instances of the same container `proxy_pass http://Rails:3000` load balances between the different instances in a Round Robin configuration by default**
- To spin up all the containers using docker compose run:<br> `docker-compose -f dockr-compose.yml up --scale Rails=3` <br>This runs all the conatiners with three containers for the Rails Applicaiton
- All requests will now go through the nginx server which acts as a reverse proxy and rate-limiter
- To stop the containers run:<br>`docker-compose -f docker-comopse.yml down`
## Task-8: Backup deamon
![image](https://user-images.githubusercontent.com/95867745/230730707-c3cd9a0a-ad1d-4a19-8ea3-9a6b4de147ac.png)
- The `cornd` deamon is configured to run the backup script every minute
