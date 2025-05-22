# Understanding Dockerfile Instructions: CMD vs ENTRYPOINT vs RUN

When working with Dockerfiles, three of the most misunderstood and frequently confused instructions are `CMD`, `ENTRYPOINT`, and `RUN`. While they may seem similar at a glance, each plays a distinct role in Docker's build and runtime processes. Here, we'll demystify these instructions, explore their differences, and offer best practices for using them effectively.

***

### 🔹 The Role of `ENTRYPOINT` in Dockerfiles

`ENTRYPOINT` defines the primary command that runs when a container starts. It sets the behavior of the container at runtime and is ideal for ensuring that your container behaves like a specific application.

#### What Happens If You Only Use `ENTRYPOINT`?

If a Dockerfile includes only an `ENTRYPOINT` without a `CMD`, the container will execute the specified command but without any default arguments. You’ll need to provide arguments manually when running the container.

```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
```

Running this image without arguments:

```bash
docker run myimage
```

Will output a blank line because `echo` was called without any arguments.

But if you pass an argument:

```bash
docker run myimage hello
```

It will output:

```
hello
```

W**hat happens?**

* The `echo` command is run **with no arguments**.
* This results in a **blank line** being printed to the console.
* The container then **exits immediately**, since the command finishes.

**No error will occur.**

It's a valid container behavior — it's just that the `ENTRYPOINT` command (`echo`) runs with no input, so it doesn’t do much.

If you want the container to do something meaningful by default, you should either:

*   Provide a `CMD` to supply default arguments:

    ```docker
    ENTRYPOINT ["echo"]
    CMD ["hello"]
    ```
* Or ensure the `ENTRYPOINT` itself includes the necessary logic or arguments.

This behavior makes it crucial to either supply runtime arguments or use `CMD` to provide defaults.

***

### 🔹 Combining `ENTRYPOINT` and `CMD` for Default Behavior

`CMD` serves as the default arguments to `ENTRYPOINT`. When both are used together, `CMD` enhances the container’s flexibility by allowing runtime overrides.

```dockerfile
FROM alpine
ENTRYPOINT ["echo"]
CMD ["Hello from CMD"]
```

Now, both of the following work:

```bash
docker run myimage            # Outputs: Hello from CMD
docker run myimage Hi there   # Outputs: Hi there
```

This combination provides a convenient way to specify defaults that can be easily overridden.

***

### 🔹 Can You Have Multiple `ENTRYPOINT` instructions?

No. Only the **last** `ENTRYPOINT` instruction in a Dockerfile is effective. Any preceding `ENTRYPOINT` will be overridden.

```dockerfile
ENTRYPOINT ["echo", "Hello"]
ENTRYPOINT ["ls", "-l"]  # This one takes effect
```

If you need to perform multiple actions at container startup, consider:

*   Using a shell form (less portable, but concise):

    ```dockerfile
    ENTRYPOINT sh -c "echo Hello && ls -l"
    ```
*   Or a startup script (recommended for complex logic):

    ```dockerfile
    COPY start.sh /start.sh
    RUN chmod +x /start.sh
    ENTRYPOINT ["/start.sh"]
    ```

    where script file `start.sh` may contain &#x20;

    ```sh
    # start.sh
    #!/bin/sh
    echo "Hello"
    ls -l
    ```

This way, you can perform multiple tasks at startup in a controlled way.

***

### 🔹 What are the different ways to define ENTRYPOINT instruction and how does it behave?

> **An `ENTRYPOINT` doesn’t need arguments, but its usefulness depends on whether it expects them.**

***

1.  **No arguments needed (self-contained ENTRYPOINT)**\
    If the `ENTRYPOINT` command is complete by itself and doesn’t require additional input, then **you don't need to pass arguments** when running the container.\
    **Example:**

    ```docker
    FROM alpine
    ENTRYPOINT ["echo", "Hello, world!"]
    ```

    Running this:

    ```bash
    docker run myimage
    ```

    Prints:

    ```
    Hello, world!
    ```

    No arguments passed — still works.
2.  &#x20;**Arguments are expected (ENTRYPOINT as a command)**

    If the `ENTRYPOINT` is just a command like `echo` or a script that expects arguments, then **you must provide them**, or it may run with no useful result.

    **Example:**

    ```docker
    FROM alpine
    ENTRYPOINT ["echo"]
    ```

    Running this:

    ```bash
    docker run myimage
    ```

    Prints a **blank line** — because `echo` was called without arguments.

    But running:

    ```bash
    docker run myimage "Hi there"
    ```

    Prints:

    ```
    Hi there
    ```
3.  **Combine with CMD for default arguments**

    You can provide **default arguments** with `CMD`, which are passed to `ENTRYPOINT` unless overridden.

    ```docker
    FROM alpine
    ENTRYPOINT ["echo"]
    CMD ["Hello from CMD"]
    ```

    Then:

    ```bash
    docker run myimage
    ```

    Prints:

    ```
    Hello from CMD
    ```

    And:

    ```bash
    docker run myimage "Custom message"
    ```

    Prints:

    ```
    Custom message
    ```

**Summary:**

* `ENTRYPOINT` **can** work without arguments — if it doesn't need any.
* If it **expects arguments**, you must supply them at runtime or via `CMD`.
* Use `CMD` to provide default arguments to `ENTRYPOINT`.

***

### 🔹 What About `RUN`?

Unlike `ENTRYPOINT`, the `RUN` instruction is used **during the image build process**, not at runtime. It’s for installing packages, setting up configurations, and other tasks needed to prepare the image.

#### Key Differences

| Feature      | `RUN` (Build-time)                                                                      | `ENTRYPOINT` (**Container** Runtime)                                                                            |
| ------------ | --------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Purpose      | Executes commands during the **image build process**(Install/setup/config during build) | Specifies the **default command** to run **when a container starts**(Define container default runtime behavior) |
| When it runs | **Once**, while the image is being built(During `docker build`)                         | **Every time** a container is started from the image(During `docker run`)                                       |
| Effect       | The command's results are baked into the **image layer**.                               | Defines how the container behaves when launched.                                                                |
| Output       | A new image layer                                                                       | A container process                                                                                             |

Think of `RUN` as baking a cake, and `ENTRYPOINT` as instructions for eating it.

`RUN` instruction example:

```docker
FROM ubuntu
RUN apt-get update && apt-get install -y curl
```

* This installs `curl` into the image during the build.
* It does **not run again** when you start a container from the image.

**`ENTRYPOINT`** instruction example:

```docker
FROM ubuntu
ENTRYPOINT ["curl"]
```

* Every time a container is started from this image, it will run the `curl` command.

**Summary:**

* Use **`RUN`** to prepare the image (install software, configure).
* Use **`ENTRYPOINT`** to define what happens **when the container is run**.

***

### 🔹 Multiple `RUN` Instructions: Should You Do It?

Yes, you can. But it’s not always efficient.

Each `RUN` command creates a **new layer** in the Docker image. These layers are cached and reusable, but they also increase the image size if not handled carefully.

Example:

```dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get install -y git
```

This works fine, but it's **not optimal**.

**Best Practice: Combine `RUN` commands when possible**

Instead, write

```docker
FROM ubuntu
RUN apt-get update && \
    apt-get install -y curl git && \
    rm -rf /var/lib/apt/lists/*
```

**Why this is better:**

* ✅ Reduces the number of image layers.
* ✅ Avoids caching outdated package indexes (`apt-get update` + install in same step).
* ✅ Reduces final image size by cleaning up.
* ✅ Better performance.

***

## 🔍 What does **"clean up after install"** mean in Docker? and how to do that?

When you install software in a Docker image — especially in Linux-based images using package managers like `apt`, `yum`, or `apk` — the installation process often leaves behind **temporary files**, **caches**, or **metadata** that:

* Are not needed at runtime
* **Increase the image size**
* Can sometimes even expose unnecessary info or security risks

So "cleaning up" means: **delete those unneeded files immediately after the install** — in the same `RUN` step — to keep the image **small and efficient**.

***

#### 🧼 Example for Debian/Ubuntu (`apt`)

```dockerfile
FROM ubuntu

RUN apt-get update && \
    apt-get install -y curl && \
    rm -rf /var/lib/apt/lists/*
```

**What's happening:**

* `apt-get update` downloads package lists
* `apt-get install` installs `curl`
* `rm -rf /var/lib/apt/lists/*` deletes package list cache

**Why it works:** We're cleaning **in the same `RUN` instruction**, so the temp files never get saved in the image layer.

***

#### 🧼 Example for Alpine (`apk`)

```dockerfile
FROM alpine

RUN apk add --no-cache curl
```

**What’s different:**

* Alpine’s `apk` has a `--no-cache` flag that **avoids caching the package index at all** — no need to clean it up manually.

***

#### 🧼 Example for RHEL/CentOS (`yum`)

```dockerfile
FROM centos

RUN yum install -y curl && \
    yum clean all && \
    rm -rf /var/cache/yum
```

***

#### ✅ Summary: Clean-Up Best Practices

| Base OS       | How to clean up after install                               |
| ------------- | ----------------------------------------------------------- |
| Debian/Ubuntu | `rm -rf /var/lib/apt/lists/*`                               |
| Alpine        | Use `--no-cache` flag with `apk add`                        |
| RHEL/CentOS   | `yum clean all` and `rm -rf /var/cache/yum`                 |
| General       | Do cleanup **in the same RUN command** to avoid image bloat |

***

### 🔚 Conclusion

Understanding the differences between `RUN`, `CMD`, and `ENTRYPOINT` is essential to writing efficient, maintainable Dockerfiles:

* Use `RUN` for image setup and package installation.
* Use `ENTRYPOINT` to define default container behavior.
* Use `CMD` to provide default arguments for `ENTRYPOINT`.
* Combine `RUN` steps when possible and always clean up to reduce image size.

With these best practices, you'll build cleaner, faster, and more predictable Docker images.
