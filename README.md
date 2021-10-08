# Apache-superset container with remote MySQL and Redis
Use superset in a docker with a remote MySQL Database (instead of postgres) and a remote redis URL.
The reason is simple - 
For production deployments, it may useful to setup a dedicated remote MySQL RDS, and a dedicated Redis cluster.
The usual superset image comes with a bundled docker image of redis and postgresql. 

**Step 1 - Prepare a MySQL RDS**

Set it up with credentials and create an empty DB for superset.
Make sure the connectivity exists ((192.168.10.11:3306)).

> create database superset;

**Step 2 - Prepare a Redis (Instance on localhost, but ideally have a dedicated remote host)**

>sudo amazon-linux-extras install redis6
>sudo systemctl start redis

add  "bind 0.0.0.0" to /etc/redis/redis.conf to make it listen to a public ip (192.168.10.10). If you are using 127.0.0.1 or localhost as redis host url, you may see connectivity problems from superset docker container because the container will try to connect to itself instead of the host.
 
>sudo systemctl restart redis

Test
>redis-cli -h 192.168.10.10 -p 6379 ping
will result in PONG

**Step 3 - Pull the latest superset code**

>git clone https://github.com/apache/incubator-superset.git

>cd incubator-superset/

Change the following sections for DATABASE and REDIS in ./docker/.env-non-dev to

\#database configurations (do not modify)

DATABASE_DB=superset

DATABASE_HOST=192.168.10.11

DATABASE_PASSWORD=Password1234

DATABASE_USER=admin

DATABASE_PORT=3306

DATABASE_DIALECT=mysql


and Redis

REDIS_HOST=192.168.10.10

REDIS_PORT=6379


In docker-compose-non-dev.yml, remove the following lines as we will not be using docker images for db and redis -

>   redis:
>   
>     image: redis:latest
>     
>     container_name: superset_cache
>     
>     restart: unless-stopped
>     
>     volumes:
>     
>       - redis:/data
>       
> 
>   db:
>   
>     env_file: docker/.env-non-dev
>     
>     image: postgres:10
>     
>     container_name: superset_db
>     
>     restart: unless-stopped
>     
>     volumes:
>     
>       - db_home:/var/lib/postgresql/data
>       
>  db_home:
>  
>    external: false
>    
>  redis:
>  
>    external: false
>    

**Step 4 - Run and Troubleshoot**

Run the command to bring superset, it will initalise MySQL DB, Redis cache, load examples and will run the UI on port 8088. In case there are any errors, try resolving them. With every attempt, drop the superset database, recreate it and run the docker-compose again.

> sudo /usr/local/bin/docker-compose -f docker-compose-non-dev.yml up

**Few Additional Notes**
For actual prod, you may need to change the default values like the default admin password for superset, and create a non-admin user for Mysql. 
If your remote DB and Redis are created in AWS, check your connectivity from the host machine. Use mysql cli and redis-cli for the verification.
If you face connectivity problems, then make sure the security groups (aws) for mysql rds and redis cache allow traffic from your host ip. 

