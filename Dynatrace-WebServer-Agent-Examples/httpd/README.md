![Apache HTTPD Logo](https://github.com/dynaTrace/Dynatrace-Docker/blob/images/apache-httpd-logo.png)

# Dynatrace WebServer Agent Example: Apache HTTPD

This project contains exemplary integrations of the [Dynatrace Application Monitoring](http://www.dynatrace.com/en/products/application-monitoring.html) enterprise solution with a [Dockerized Apache HTTPD](https://hub.docker.com/_/httpd/) process for deep end-to-end application monitoring.

**Note**: the Dynatrace Web Server Agent (master agent) process inside `dynatrace/wsagent` is tightly coupled to the web server modules (slave agents) which are loaded into web server processes. Due to high latency sensitivity, these counterparts must always run on the same host.

## How to install Dynatrace?

You can quickly bring up an entire Dockerized Dynatrace environment by using [Docker Compose](https://docs.docker.com/compose/) with the [provided `docker-compose.yml` file](https://github.com/dynaTrace/Dynatrace-Docker/blob/master/docker-compose.yml) like so:

```
DT_SERVER_LICENSE_KEY_FILE_URL=http://repo.internal/dtlicense.key \
docker-compose up
```

**Note**: the example runs an instance of type `dynatrace/server`. This image has been designed to run in low-traffic, resource-constrained **demo and trial environments**. Dynatrace does not support its use in production or pre-production grade environments of any kind.

### Licensing

In the example above, you have to let `DT_SERVER_LICENSE_KEY_FILE_URL` point to a valid Dynatrace License Key file. If you don't have a license yet, you can [obtain a Dynatrace Free Trial License here](http://bit.ly/dttrial-docker-github). However, you don't need to have your license file hosted by a server: if you can run a console, [Netcat](https://en.wikipedia.org/wiki/Netcat) can conveniently serve it for you on port `80` via `sudo nc -l 80 < dtlicense.key`.

## How to instrument a Dockerized Apache HTTPD process?

With the Dockerized Dynatrace environment running, we can now easily instrument an application process without having to alter that process' Docker image. Here is what an examplary integration in `run-container.sh` looks like:

<pre><code>#!/bin/bash
HTTPD_LOAD_MODULE="dtagent_module \${DTWSAGENT_ENV_LIB64}"

echo "Starting Apache HTTPD - Example"
docker run --rm \
  --name httpd-example \
  <strong>--volumes-from dtwsagent</strong> \                                            # <strong>1)</strong>
  <strong>--link dtwsagent</strong> \                                                    # <strong>2)</strong>
  <strong>--ipc container:dtwsagent</strong> \                                           # <strong>3)</strong>
  --publish-all \
  httpd \
  sh -c "<strong>\${DTWSAGENT_ENV_DT}/attach-to-wsagent-master.sh</strong> && \          # <strong>4)</strong>
         (<strong>echo LoadModule ${HTTPD_LOAD_MODULE} >> conf/httpd.conf</strong>) && \ # <strong>5)</strong>
         httpd-foreground"
</code></pre>

### Behind the Scenes

1) We mount the agent installation directory from the `dtwsagent` container into the web server process' container via `--volumes-from dtwsagent`. This is required to share configuration information between the master and the slave agents.

2) **Convenience**: We link the web server process' container against the `dtwsagent` container via `--link dtwsagent`. This way, we inherit the other container's environment variables `DTWSAGENT_ENV_DT` and `DTWSAGENT_ENV_LIB64` and can thus quickly deduce a `LoadModule` declaration without having to know much about the environment.

3) We share the `dtwsagent`'s IPC namespace via `--ipc container:dtwsagent`. This is required to allow the master and the slave agents to communicate via a shared memory segment across containers on the same host.

4) We attach to the master agent by invoking `attach-to-wsagent-master.sh`, which has been shared by the `dtwsagent` container in step **1)**. This is required to allow the master and the slave agents to communicate via UDP.

5) We place a `LoadModule` declaration inside Apache's `httpd.conf` before we invoke the actual web server process ([see here for the original Dockerfile](https://github.com/docker-library/httpd/blob/1f1f7d39d5fe5aebeedea6872786b4e3ce0ebcc9/2.4/Dockerfile)).

## Additional Information

See the following Dockerized Dynatrace components and examples for more information:

- [Dockerized Dynatrace Server](https://github.com/dynaTrace/Dynatrace-Docker/tree/master/Dynatrace-Server)
- [Dockerized Dynatrace Collector](https://github.com/dynaTrace/Dynatrace-Docker/tree/master/Dynatrace-Collector)
- [Dockerized Dynatrace Agent](https://github.com/dynaTrace/Dynatrace-Docker/tree/master/Dynatrace-Agent) and [Examples](https://github.com/dynaTrace/Dynatrace-Docker/tree/master/Dynatrace-Agent-Examples)
- [Dockerized Dynatrace Web Server Agent](https://github.com/dynaTrace/Dynatrace-Docker/tree/master/Dynatrace-WebServer-Agent) and [Examples](https://github.com/dynaTrace/Dynatrace-Docker/tree/master/Dynatrace-WebServer-Agent-Examples)

## Problems? Questions? Suggestions?

This offering is [Dynatrace Community Supported](https://community.dynatrace.com/community/display/DL/Support+Levels#SupportLevels-Communitysupported/NotSupportedbyDynatrace(providedbyacommunitymember)). Feel free to share any problems, questions and suggestions with your peers on the Dynatrace Community's [Application Monitoring & UEM Forum](https://answers.dynatrace.com/spaces/146/index.html).

## License

Licensed under the MIT License. See the LICENSE file for details.
[![analytics](https://www.google-analytics.com/collect?v=1&t=pageview&_s=1&dl=https%3A%2F%2Fgithub.com%2FdynaTrace&dp=%2FDynatrace-Docker%2FDynatrace-WebServer-Agent-Examples%2Fhttpd&dt=Dynatrace-Docker%2FDynatrace-WebServer-Agent-Examples%2Fhttpd&_u=Dynatrace~&cid=github.com%2FdynaTrace&tid=UA-54510554-5&aip=1)]()