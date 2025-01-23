# OptServer demo

The [OptServer](https://docs.mosek.com/latest/opt-server/index.html) is a tool for submitting and executing MOSEK optimizations remotely. This Docker image sets up the simplest stand-alone OptServer container with latest MOSEK version and with all dependencies installed internally. 

## Building and running

Prepare a folder containing this Dockerfile and a MOSEK license file:

```
Dockerfile
optserver.conf
start.sh
mosek.lic
```

(You can quickly get a trial license file from https://www.mosek.com/try/). Inside that folder execute

```
docker build -t optserver-demo .
docker run -p 30080:30080 optserver-demo
```

## Optimizing

If all went well now you have a demo instance of the MOSEK OptServer accessible via:

```
http://your.machine.name:30080
```

To optimize remotely provide that address in your MOSEK code following instructions in your interface manual. Assuming MOSEK 9.2+:

* Optimizer API: call ``putoptserverhost`` see [C](https://docs.mosek.com/latest/capi/tutorial-remote-optimization.html), [Python](https://docs.mosek.com/latest/pythonapi/tutorial-remote-optimization.html), [C#](https://docs.mosek.com/latest/dotnetapi/tutorial-remote-optimization.html), [Java](https://docs.mosek.com/latest/javaapi/tutorial-remote-optimization.html)
* Fusion API: call ``M.optserverHost``, see [C++](https://docs.mosek.com/latest/cxxfusion/tutorial-remote-optimization.html), [Python](https://docs.mosek.com/latest/pythonfusion/tutorial-remote-optimization.html), [C#](https://docs.mosek.com/latest/dotnetfusion/tutorial-remote-optimization.html), [Java](https://docs.mosek.com/latest/javafusion/tutorial-remote-optimization.html)
* Command line: ``mosek -optserv http://your.machine.name:30080 filename``
* MATLAB: [see here](https://docs.mosek.com/latest/toolbox/tutorial-remote-optimization.html) or [see here](https://docs.mosek.com/latest/matlabapi/tutorial-remote-optimization.html)

It is assumed that you are using the same MOSEK version in the client as on the OptServer, which, if using this ``Dockerfile``, is the last stable version (the first two numbers ``major.minor`` matter). Otherwise modify the file to install MOSEK versions other than ``latest``.

## Web interface

To see if the server works open

```http://your.machine.name:30080/api/v1/version```

To try the web interface open 

```http://your.machine.name:30080```

in a Web browser. Log in with user ``admin``, password ``admin``.

## Other

* If you attach in interactive mode you can start/stop the OptServer using the scripts provided and adjust configuration.
* Documentation is at https://docs.mosek.com/latest/opt-server/index.html
* If you don't provide a license file, comment out the ``COPY`` command in the dockerfile and you can still try the web interface, but without being able to actually optimize anything.
* Contact ``support@mosek.com`` in case of questions.

