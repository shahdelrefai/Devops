## Docker Build
Builds Docker images from a `Dockerfile` and a `context`.

> Context: set of files located in a specific path or URL.

**Command**

```bash
docker build [Options] PATH | URL | -
```
 
 **Options**
 
- `-t`, `--tag`: Name, version
- `--build-arg`: Set build-time variables
- `-f`, `--file`: Name of Dockerfile (default is PATH/Dockerfile)

**Example**

```bash
docker build -t log-generator:0.1 -f Dockerfile . # . is the path you're standing in
```

---

## Docker Instructions
### 1. FROM
- Initializes a new build stage and sets the base image (valid image) for subsequent instructions.
- For an empty image `FROM scratch`.
- Using more than one FROM is called `multi-stage build`.
- Use `AS` to name stages.

### 2. RUN
- Executes any commands to create a `new layer` on top of the current image.
- The added layer is used in the next steps in the Dockerfile.
- To execute multiple RUN commands:
    - Have several RUN lines, **not recommended** as they will add layers to the image making it heavier.
    - Use `&&` 
    ```bash 
    RUN apt-get update && apr-get install -y curl
    ```
    - Use input file
      ```bash
      RUN <<EOF
      apt-get update
      apt-get install -your curl
      EOF
      ```

### 3. COPY
- Copies new files or directories from `<src>` and adds them to the filesystem of the container at the path `<dest>`.

```bash
COPY [OPTIONS] <src>..<dest>
```

**Options**

- `--from`: Copy files from a previous stage.
- `--chown`: Change the ownership of the objects. (Linux Containers)
- `--chmod`: Change the access permission of the objects. (Linux Containers)


### 4. ENV
- Sets the `environment variable` key to the value.
- Allows for multiple `<key>=<value>` variables to be set at one time.

```bash
ENV <key>=<value>..
```

- ENV variables will persist when a container is run from the resulting image (if you only need a variable that persists during the image build use `ARG`).
- It persists from stage to stage.


### 5. ARG
- Defines a variable that users can pass at build-time.
- It can include a default value.

```bash
ARG <name>[=<default value>]
```

- The only instruction that can precede FROM at the start of the Dockerfile.
  **Example:**
  ```bash
  ARG CODE_VERSION=latest
  FROM base:${CODE_VERSION}
  ```
  
- ARG variables goes out of scope at the end of the stage where it was defined. You must include ARG instruction in each stage.

> ENV variables always override ARG variables of the same name


### 6. WORKDIR
- Sets the working directory for any instruction that follows it.
- If the WORKDIR doesn't exist, it will be created even if it's not used in any subsequent instruction.

```bash
WORKDIR /path/to/workdir
```


### 7. USER
- Sets the `username (UID)` and optionally the user `group (GID)`.
- Persists for the current stage.
- Specified user is used for RUN instructions, and at runtime, runs the relevant ENTRYPOINT and CMD commands.

```bash
USER <user>[:<group>]
```

> The user should be already existing on the system otherwise an error will occur

- If no USER is specified, the commands are executed under `root user`, which isn't recommended from a security point.   


### 8. HEALTHCHECK
- Tells Docker how to test a container to check that it's still working.
- Disabled by default.

**Scenario**
A web server stuck in a infinite loop and unable to handle new connections, even though the server process is still running in Docker.
  
**Options**
  
- `--interval=`: DURATION (default: 30s)
- `--timeout=`: DURATION (default: 30s)
- `--retries=`: N (default: 3)
- `--start-period=`: DURATION (default: 0s)

**Example**
```bash
HEALTHCHECK --interval=5m --timeout=3s \ CMD curl -f http:localhost/ || exit 1 
# call the curl every 5m and if it didn't respond within 3s exit with code 1
```


### 9. EXPOSE
- Informs Docker that the `container listens` on the specified network ports at runtime.
```bash
EXPOSE <port>[<port>/<protocol>]
```

- It assumes TCP by default.
- Doesn't actually publish the port. It functions as a type of documentation between the person who builds the image and the person who runs the container. (To open the ports use `-p` when running)

> A container is only alive as long as the process inside is alive
*What defines the process that executes within a process container?* `CMD` or `ENTRYPOINT`
### 10. CMD and ENTRYPOINT
**CMD**

- Sets the executable files and parameters that are executed during when running a container.
```bash
CMD ["executable", "param1", "param2"]
CMD ["param1", "param2"] # as default parameters to ENTRYPOINT
CMD command param1 param2
```

- There can only be one CMD instruction in a Dockerfile. If you list more than one CMD, only the last one takes effect.
- The purpose of a CMD is to provide defaults for an `executing container`

**ENTRYPOINT**

- Allows you to configure a container that will run as an executable.
```bash
ENTRYPOINT ["executable", "param1", "param2"] # preferred
ENTRYPOINT command param1 param2
```

- Only the last ENTRYPOINT instruction will have an effect.

**CMD vs ENTRYPOINT**

- The ENTRYPOINT instruction is not overriden by any arguments specified at the docker run command line. (if you would like your container to run the same executable every time, then use ENTRYPOINT).
- You can combine ENTRYPOINT with CMD to provide additional default arguments that can be appended or overridden by the user.

**Example**
```bash
FROM ubuntu
ENTRYPOINT["echo"] # the executable
CMD["Hello, world!"] # the parameters of the executable which can be overriden
```

***

<img width="1150" height="790" alt="image" src="https://github.com/user-attachments/assets/513c500f-6219-442a-a6b5-35cbe6c2ae2d" />

## Docker Push
```bash
docker image push [OPTIONS] NAME[:TAG]
docker push [OPTIONS] NAME[:TAG]
```
**Options**

- `-a`, `--all-tags`: Pushes all tags of an image to the registry, not just the `latest` tag if it exists.
- `--disable-content-trust`: Skips content trust verification. By default, Docker attempts to verify content trust, which ensures the integrity and authenticity of images. Disabling it should be done with caution.
- `-q`, `--quiet`: Suppresses verbose output during the push operation. This can be useful in scripting or when you only need to see errors.


## Docker Tag
```bash
docker image tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
docker tag SOURCE_IMAGE[:TAG] TARGET_IMAGE[:TAG]
```
