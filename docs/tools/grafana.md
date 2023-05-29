# Grafana

To intialize the Grafana container, run the following compose file:

```yml
version: '3'

services:
  grafana:
    image: grafana/grafana-oss:latest
    ports:
      - 3000:3000
    volumes:
      - grafana-data:/var/lib/grafana
```

Go to `http://localhost:3000` and login with the default credentials `admin:admin`.

Add first datasource to show out prometheus URL (9090).  
Click dashboard and show custom metrics in there.  
For testing import cadvisor's dashboard 14282 and select prometheus.  
In prometheus go to status -> targets to check targets are up.

Grafana-data folder should be have `420:0` permissions.
