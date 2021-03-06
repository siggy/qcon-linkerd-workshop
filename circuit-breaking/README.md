# Linkerd Workshop: Circuit-Breaking

In this workshop we will see the benefits of circuit-breaking.  This
directory contains a `docker-compose.yml` file which defines:

* 5 simple backend servers
* Slow Cooker, a load generator which is configured to send requests to the servers
* Prometheus, to collect metrics from the servers and from Slow Cooker
* Grafana, to display the metrics in Prometheus

## Getting a Baseline

Start the above containers by running:

```bash
docker-compose build
docker-compose up -d
```

View the Grafana dashboard

```bash
open http://localhost:3000 # or docker ip address
```

Note the success rate:

* success rate: ____

Notice the distribution of request volume per instance.  Do some servers seem
to be serving more requests than others, or are they all roughly the same?

## Adding Circuit-Breaking with Linkerd

Add this Linkerd service to the `docker-compose.yml` file to add a Linkerd
container:

```yaml
  linkerd:
    image: buoyantio/linkerd:1.3.5
    ports:
      - 4140:4140
      - 9990:9990
    volumes:
      - ./linkerd.yml:/io/buoyant/linkerd/config.yml:ro
      - ./disco:/disco
    command:
      - "/io/buoyant/linkerd/config.yml"
```

Edit the Slow Cooker service in `docker-compose.yml` to send to 
`http://linkerd:4140` instead of `http://server:8501`.

Linkerd reads its configuration from `linkerd.yml`.  Linkerd can be configured
to use success rate based failure accrual.  What this means is that Linkerd will
track the success rate of each server instance over a sliding window of
requests.  If the success rate for a server instance drops below a certain
threshold, Linkerd will trigger the circuit-breaker and stop sending traffic
to that instance for a period of time.

Edit `linkerd.yml` to set the `successRate` threshold to `0.9` and the
`requests` sliding window size to `20`.

Redeploy the containers and look at the Grafana dashboard again:

```bash
docker-compose down
docker-compose build
docker-compose up -d
open http://localhost:3000 # or docker ip address
```

Note the success rate:

* success rate: ____

Notice the distribution of request volume per instance.  Do some servers seem
to be serving more requests than others, or are they all roughly the same?

## Clean up

Stop and remove all running containers:

```bash
docker-compose down
```
