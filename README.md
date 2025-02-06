## Dune Checker

In this repository you will find a Dockerfile that creates a [CodeChecker](https://github.com/Ericsson/codechecker/tree/master) client that will analyze a branch from [LSTS' DUNE](https://github.com/LSTS/dune) repository. After analysis, it attempts to publish a report to a CodeChecker server. 
Below, I will break down how to use this repository.

## Dockerfile (Server)

The first piece of puzzle come as a [CodeChecker Server](https://github.com/Ericsson/codechecker/blob/master/docs/web/user_guide.md#server). This server will contain a database that will store each analysis report you send it.
To access them you simply need to open the correct port of the host server on an html page (if your machine https://localhost:port or 10.0.0.15:8001). Thankfully a Dockerfile with a working server already exists on [Docker Hub](https://hub.docker.com/r/codechecker/codechecker-web). Simply pull the image:
```
  docker pull codechecker/codechecker-web
```
And run it:
```
  docker run -d \
  -p 8001:8001 \
  -v /home/$USER/codechecker_workspace:/workspace \
  codechecker/codechecker-web:latest
```
The `-p` argument will expose your local port 8001 to the container'sls 8001 port and `-v` you will attach a local diretory to a directory inside the container. This will be the container that the database with the reports will be stored. This way your reports won't be lost if the container, for whatever reason, shuts down. 

If you're inside LSTS's lab network, there should be a CodeChecker instance running on the Jenkins Server.

## Dockerfile (Client)

For the next piece, the container created by this docker file will install all needed dependencies to build DUNE's source code. Similar to the previous Dockerfile this will also contain a complete CodeChecker installation. Except, in this case we won't be hosting a server, instead, we will be running the actual analyzer and publishing the report to the server.  
After the environment is set up, it builds DUNE while generating a .json file that contains all compilation commands that happen during said build.

```
  CodeChecker log --build "make -j8" --output /opt/codechecker/compile_commands.json
```

This is what CodeChecker uses to then analyze the source code. 

```
  CodeChecker analyze compile_commands.json --config client_config.json -o output
```

The output of the above command will be a folder that includes reports on all the files that were analyzed during the build process. 

Finally we will upload this report to the server through the following command.

```
  CodeChecker store output --name=run_name --url=http://host_server_ip:port_server/product_name
```
These arguments are important so pay attention! The way reports are divided and related to each other are through [products](https://codechecker.readthedocs.io/en/latest/web/products/) which have different runs. Let's say you are analyzing different branches from DUNE. Ideally you would divide each branch into different produts. In the case you are analyzing the same branch but with new changes made you would upload the reports to the same product with a different name. 
To choose the product name and server location you should edit the `--url` and to differentiate between runs you should change the `--name` parameter.  

