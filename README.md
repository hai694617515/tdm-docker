# TDM Docker

A containerized [TDM](http://www.unidata.ucar.edu/software/thredds/current/tds/reference/collections/TDM.html). This container is meant to work in close conjunction with the [TDS Docker container](https://github.com/Unidata/thredds-docker) which has additional information for running the TDS and TDM containers in coordinated fashion.

## Versions

* `unidata/tdm-docker:latest`
* `unidata/tdm-docker:4.6.11`
* `unidata/tdm-docker:5.0-SNAPSHOT`

## Configuration
### docker-compose Parameterization

To run the TDM Docker container, beyond a basic Docker setup, we recommend installing [docker-compose](https://docs.docker.com/compose/). We will assume you have knowledge on how to [configure a TDS](https://www.unidata.ucar.edu/software/thredds/current/tds/tutorial/BasicConfigCatalogs.html).

In the `docker-compose.yml` file, `volumes` mapping section, you will point the TDM to the [TDS content root directory](https://github.com/Unidata/thredds-docker#thredds) (e.g., `~/tdsconfig/`) and the data directory corresponding to the `DataRoots` element in `threddsConfig.xml`.

Because you will most likely run this container alongside the `thredds-docker` container, see the `thredds-docker` project [README](https://github.com/Unidata/thredds-docker) for additional parameterization via the `compose.env` file.

### Configurable TDM UID and GID

[See parent unidata/tomcat container](https://github.com/Unidata/tomcat-docker#configurable-tomcat-uid-and-gid).

Set the UID/GID of the TDM user via the `compose.env` file.  If not set, the default UID/GID is `1000`/`1000`.

### TDM Password and Coordination with the TDS

The TDM will notify the TDS of data changes via an HTTPS port `8443` triggering mechanism. It is important the TDM password (`TDM_PW` environment variable) defined in the [docker-compose.yml](https://github.com/Unidata/thredds-docker/blob/master/docker-compose.yml) file corresponds to the SHA **digested** password in the [tomcat-users.xml](https://github.com/Unidata/thredds-docker/blob/master/files/tomcat-users.xml) file. [See the parent Tomcat container](https://hub.docker.com/r/unidata/tomcat-docker/) for how to create a SHA digested password. Also, because this mechanism works via port `8443`, you will have to get your HTTPS certificates in place. Again [see the parent Tomcat container](https://hub.docker.com/r/unidata/tomcat-docker/) on how to install certificates, self-signed or otherwise.

Not having the Tomcat `tdm` user password and digested password in sync can be a big source of frustration. One way to diagnose this problem is to look at the TDM logs and `grep` for `trigger`. You will find something like:

```sh
fc.NAM-CONUS_80km.log:2016-11-02T16:09:54.305 +0000 WARN  - FAIL send trigger to http://thredds-jetstream.unidata.ucar.edu/thredds/admin/collection/trigger?trigger=never&collection=NAM-CONUS_80km status = 401
```

Enter the trigger URL in your browser:

```sh
http://thredds-jetstream.unidata.ucar.edu/thredds/admin/collection/trigger?trigger=never&collection=NAM-CONUS_80km
```

At this point the browser will prompt you for a `tdm` login and password you defined in the `docker-compose.yml`. If the triggering mechanism is successful, you see a `TRIGGER SENT` message. Otherwise, make sure your HTTPS certificate is present, and ensure the `tdm` password in the `docker-compose.yml`, and digested password in the `tomcat-users.xml` are in sync.

## Running the TDM

    docker-compose up -d tdm

## Capturing TDM Log Files Outside the Container

Until `5.0`, the TDM lacks configurability with respect to the location of log files and the TDM simply logs locally to where the TDM is invoked. In the meantime, to capture TDM log files outside the container, do the usual volume mounting outside the container:

    /path/to/your/tdm/logs:/usr/local/tomcat/content/tdm/

*and* put the `tdm.jar` and `tdm.sh` run script in `/path/to/your/tdm/logs`.

For example, you can get the `tdm.jar`:

    curl -SL  https://artifacts.unidata.ucar.edu/repository/unidata-releases/edu/ucar/tdmFat/4.6.11/tdmFat-4.6.11.jar

The `tdm.sh` script can be found within this repository. Make sure the `tdm.sh` script is executable by the container.

## Citation

In order to cite this project, please simply make use of the [Unidata THREDDS Data Server DOI](https://data.datacite.org/10.5065/D6N014KG).
