## Dune Checker

In this repository you will find a Dockerfile that creates a [CodeChecker](https://github.com/Ericsson/codechecker/tree/master) client that will analyze a branch from [LSTS' DUNE](https://github.com/LSTS/dune) repository. After analysis, it attempts to publish a report to a CodeChecker server. 
Below, I will break down the usage of this repository. 

## Dockerfile

The container created by this docker file will install all needed dependencies to build DUNE's source code. It also contains an installation of codechecker. 
After environment is set up, it builds DUNE while generating a .json file that contains all compilation commands that happen during said build.

```
  CodeChecker log --build "make -j8" --output /opt/codechecker/compile_commands.json
```

This is what CodeChecker uses to then analyze the source code. 

```
  CodeChecker analyze compile_commands.json --config client_config.json -o output
```

The output of the above command will be a folder that includes reports on all the files that were analyzed during the build process. 

