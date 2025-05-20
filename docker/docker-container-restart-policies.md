# Docker Container Restart Policies

Docker’s restart policies ensure high availability by automatically restarting containers after failures or restarts, reducing manual intervention. In essence, they **keep services running through crashes or reboots**[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs) [\[2\]](./#id-2.-difference-in-docker-restart-policy-between-on-failure-and-unless-stopped-or-stack-overflow)**.** Docker even recommends using restart policies instead of manual process managers[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). By configuring `--restart`, you can control if and how a container is brought back up after it stops or when the Docker daemon restarts.

### Restart Policy Options (`--restart` flag)

Docker supports four restart policies. Each policy determines under what conditions a container is restarted:

* **`no`** (default): _Do not restart the container._ If the container stops for any reason, Docker will not attempt to restart it[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). This is useful for one-off or batch jobs where you don’t want automatic retries.
* **`on-failure[:max-retries]`**: _Restart only on errors._ Docker will restart the container **only if** it exits with a **non-zero** exit code (indicating a failure)[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). You can append `:N` (for example `on-failure:5`) to limit the number of restart attempts. Without a limit, Docker will keep trying indefinitely on each failure[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). This policy is handy for applications that should auto-retry on crashes but not on clean exits.
* **`always`**: _Always restart the container whenever it stops_[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). No matter what caused the container to stop (even a clean exit), Docker will restart it. Additionally, if the Docker daemon or host machine restarts, Docker will bring the container back up[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs) [\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean). Note that if you **manually stop** the container (`docker stop`), it will stay stopped until you manually restart it or the Docker daemon restarts[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
* **`unless-stopped`**: _Always restart the container except when it has been stopped by the user_[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). This is like `always`, but with one key difference: once you manually stop the container, Docker will **not** restart it again, even after a Docker daemon or system reboot[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). This gives you manual control: the container auto-restarts on failures or reboots, but stays down once explicitly stopped.

Each policy’s behavior can be summarized from the official docs: Docker will _“restart the container if it exits due to an error”_ for `on-failure`, _“always restart the container if it stops”_ for `always`, and _“similar to `always`, except that when the container is stopped (manually or otherwise), it isn’t restarted even after Docker daemon restarts”_ for `unless-stopped` [\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).

### When to Use Each Policy

Choosing a policy depends on your application’s needs:

* **`no`**: Use when you want **no automatic restarts**. This is the default and is suitable for short-lived tasks or when you prefer to manage restarts manually.
* **`on-failure`**: Good for services that may occasionally crash but should retry, while avoiding endless loops. For example, development/testing or non-critical tasks where a crash should trigger a retry but you don’t want a constantly failing container to consume resources. You can also limit retries (e.g. `on-failure:3`) to avoid infinite restarts[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
* **`always`**: Ideal for **critical, long-running services** that must stay up 24/7 (like web servers, databases, etc.). Even if the service exits normally or after a crash, Docker will bring it back. This also ensures the service restarts on Docker or host reboot[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean).
* **`unless-stopped`**: Similar to `always`, but gives administrators more control. Use it for critical services where you want auto-restart on failures and reboots, _except_ you may want to stop them manually without them coming back automatically[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean). For example, for a service you only want down for planned maintenance; once manually stopped, it won’t restart until you start it again.

In production, a common pattern is to use `always` or `unless-stopped` for essential services (so they recover from crashes or reboots), and `on-failure` for less critical or finicky services to prevent endless crash loops[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean). In fact, experts note: _“Critical applications: use `always` or `unless-stopped` … Non-critical or testing: use `on-failure` to limit restarts …”_[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean).

### Behavior in Key Scenarios

The table below summarizes what each restart policy does in different events:

* **Container exits unexpectedly (crash or non-zero exit):**
  * `no`: Container stops and is not restarted.
  * `on-failure`: Docker **restarts** the container (up to any specified retry limit) _only if_ it exited with an error (non-zero exit code)[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs). If the container exits cleanly (zero status), Docker does nothing.
  * `always`: Docker **always restarts** the container, regardless of exit status[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
  * `unless-stopped`: Same as `always` for unexpected exits; Docker restarts it as long as it wasn’t manually stopped before[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
* **Docker daemon restarts (or service restart):**
  * `no`: Container remains stopped; Docker does not restart it automatically.
  * `on-failure`: Container remains stopped. The `on-failure` policy **does not** restart containers simply because the Docker daemon came back up[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
  * `always`: Docker restarts the container after the daemon restarts, unless you had stopped it manually before the restart[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
  * `unless-stopped`: Docker restarts the container (just like `always`), _unless_ the container had been manually stopped earlier. If you stopped the container before the daemon restart, it stays stopped[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).
* **Host machine reboots:** _(This is equivalent to a Docker daemon restart on boot.)_
  * `no`: Container remains stopped after reboot.
  * `on-failure`: Container remains stopped (no auto-start on boot).
  * `always`: Docker starts the container when the host boots[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean).
  * `unless-stopped`: Docker starts the container on boot _if_ it was not manually stopped before; if you had stopped it, it stays down[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean).

In summary, **`always` and `unless-stopped`** will resurrect your container after host or Docker restarts (keeping the service running), whereas **`no` and `on-failure`** leave it down. The official documentation emphasizes that `on-failure` will _not_ revive a container on daemon restart[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs), whereas `always` will[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs).

### Examples of Using Restart Policies

#### `docker run` examples

Here are some `docker run` commands showing restart flags:

```bash
# Always restart the nginx container on stops or reboots
docker run -d --restart=always --name webserver nginx

# Restart my-app up to 5 times if it crashes (non-zero exit)
docker run -d --restart=on-failure:5 --name my-app my-image

# Restart redis container unless it is manually stopped
docker run -d --restart=unless-stopped --name redis redis
```

Each command uses `-d` to run detached and sets the desired policy after `--restart=`. For instance, the second line tells Docker to restart `my-app` only on crashes, retrying up to 5 times[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean). The first line will keep `nginx` running continuously (even after a reboot).

#### `docker-compose.yml` examples

You can also set restart policies in Docker Compose. In a `docker-compose.yml` file, use the `restart` key under a service. For example:

```yaml
version: '3'
services:
  web:
    image: nginx
    restart: always

  worker:
    image: my-worker-image
    restart: on-failure:3
```

In this snippet, the `web` service will always restart on failure or system reboot, while the `worker` service will restart up to 3 times if it exits with an error. This follows the same semantics as the `docker run` flags[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs) [\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean).

By choosing the appropriate policy, you ensure your containers **recover automatically** from crashes or reboots. For example, running critical services with `restart: always` (or `unless-stopped`) ensures they stay up through Docker or host restarts[\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean), while less critical tasks can use `on-failure` to avoid endless restart loops.

**Sources:** Official Docker documentation and community guides outline these policies and behaviors in detail[\[1\]](./#id-1.-start-containers-automatically-or-docker-docs) [\[3\]](./#id-3.-how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet-or-digitalocean).

## Citations:

#### 1. [Start containers automatically | Docker Docs](https://docs.docker.com/engine/containers/start-containers-automatically/)

#### 2. [Difference in docker restart policy between on-failure and unless-stopped? | Stack Overflow](https://stackoverflow.com/a/67335047)

#### 3. [How to Set Up Docker Container Auto-Restart on Failure in a DigitalOcean Droplet? | DigitalOcean](https://www.digitalocean.com/community/questions/how-to-set-up-docker-container-auto-restart-on-failure-in-a-digitalocean-droplet)
