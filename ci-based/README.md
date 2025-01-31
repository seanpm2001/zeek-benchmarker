# Zeek Benchmarker

This repo contains a set of python and bash scripts for running a remote benchmarking service for Zeek and Broker builds. It is intended for use with the Cirrus CI continuous integration service that Zeek uses for automated build and test, but could be adapted to run against other hosts as well.

It uses Docker for privilege separation when running the benchmark scripts.

## Requirements
- Python 3
- Docker
- Docker-compose

## Setup

1. Create three docker volumes as described at the top of the docker-compose file:
   - grafana_data: A path to store data for grafana
   - test_data: A path containing pcaps for zeek benchmarks
   - broker_test_data: A path containing a cluster configuration data file for broker benchmarks
2. Create the four necessary docker images by running the following. Ignore warnings about unset environment variables.

   sudo docker-compose build zeek-local
   sudo docker-compose build zeek-remote
   sudo docker-compose build broker-local
   sudo docker-compose build broker-remote

3. Edit the config.yml and set the values as described in that file.
4. Create a python virtualenv in the same directory as the script, and use `pip` to install the required python modules listed in `requirements.txt`
5. If systemd support is desired, edit the zeek-benchmarker.service file, update the `<path>` values to the location of the script, and copy it into the proper directory for systemd (usually `/etc/systemd/system`).
6. By default the benchmarker runs on a system with a large amount of CPUs, and we limit the CPUs used to a specific set. This is set in the `app.config['CPU_SET']` variable. If used on a system with fewer CPUs, this needs to be updated.

## Supported Endpoints

### `/zeek`:

This endpoint is used to benchmark builds of the primary Zeek repo based on PRs and marges from the Cirrus CI system.

#### Required header values and arguments:

- Header value `Zeek-HMAC`: This is the HMAC value computed from the combination of the endpoint and a request timestamp, in the form of `zeek-<timestamp>`, using the key provided in the HMAC_KEY variable in the script.
- Header value `Zeek-HMAC-Timestamp`: This is the timestamp used in the above computation.
- Argument `branch`: The full branch name being tested. This will be checked by `git` to ensure that it is a valid branch name.
- Argument `build`: The full URL to the build being tested. By default the script checks to ensure that the URL is coming from the Cirrus infrastructure.
- Argument `build_hash`: A sha256 hash of the build file.

#### Output

The benchmark outputs the amount of time it took to read and process the `DATA_FILE` and the maximum amount of memory used to process it, as such:

```
Time spent: 97.25 seconds
Max memory usage: 2125832 bytes
```

### `/broker`:

This endpoint is used to benchmark builds of the primary Broker repo based on PRs and marges from the Cirrus CI system.

#### Required header values and arguments:

- Header value `Zeek-HMAC`: This is the HMAC value computed from the combination of the endpoint and a request timestamp, in the form of `zeek-<timestamp>`, using the key provided in the HMAC_KEY variable in the script.
- Header value `Zeek-HMAC-Timestamp`: This is the timestamp used in the above computation.
- Argument `branch`: The full branch name being tested. This will be checked by `git` to ensure that it is a valid branch name.
- Argument `build`: The full URL to the build being tested. By default the script checks to ensure that the URL is coming from the Cirrus infrastructure.
- Argument `build_hash`: A sha256 hash of the build file.

#### Output

The benchmark returns the output from the broker-cluster-benchmark included with Broker:

```
zeek-recording-logger (sending): 0.0098869s
zeek-recording-manager (sending): 0.584391s
zeek-recording-proxy (sending): 1.00211s
zeek-recording-worker (receiving): 1.00507s
zeek-recording-proxy (receiving): 14.334s
zeek-recording-manager (receiving): 14.3401s
zeek-recording-worker (sending): 14.7662s
zeek-recording-logger (receiving): 15.2519s
system: 15.2516s
```
