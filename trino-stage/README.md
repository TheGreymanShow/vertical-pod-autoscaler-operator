### VPA observations on Trino-Staging namespace

VPA Spec:

```
apiVersion: "autoscaling.k8s.io/v1"
kind: VerticalPodAutoscaler
metadata:
  name: trino-stage-v1
  namespace: opf-trino-stage
  labels:
    app: trino-stage-v1
spec:
  targetRef:
    apiVersion: "apps/v1"
    kind: Deployment
    name: trino-worker
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: "trino-worker"
      maxAllowed:
        cpu: "3000m"
        memory: "4Gi"

```

Initial VPA reco on creation:

​		 ![Screenshot 2021-11-22 at 8.52.08 PM](images/Screenshot%202021-11-22%20at%208.52.08%20PM.png)

3651757469 =~ 3.65 gb upper bound

1938998548 = ~2gb lower bound

2975900105 =~ 3gb target

Usage at the time of creation:

2021-11-22 20:49:40 ![Screenshot 2021-11-22 at 8.55.22 PM](images/Screenshot%202021-11-22%20at%208.55.22%20PM.png)

After VPA is enabled, pods got auto-updated with values immediately:

```
    Limits:
      cpu:     25m
      memory:  2975900105
    Requests:
      cpu:     25m
      memory:  2975900105
```

CPU is reduced from 500m to 25m

Memory is increased from 2gb to ~3gb

![Screenshot 2021-11-23 at 8.45.06 AM](images/Screenshot%202021-11-23%20at%208.45.06%20AM.png)

![Screenshot%202021-11-23%20at%208.45.12%20AM](images/Screenshot%202021-11-23%20at%208.45.12%20AM.png)



![Screenshot%202021-11-23%20at%208.45.55%20AM](images/Screenshot%202021-11-23%20at%208.45.55%20AM.png)

![Screenshot%202021-11-23%20at%208.46.02%20AM](images/Screenshot%202021-11-23%20at%208.46.02%20AM.png)

=====

Recommendation after 15mins of VPA creation:

![Screenshot 2021-11-22 at 9.03.25 PM](images/Screenshot%202021-11-22%20at%209.03.25%20PM.png)

Recommendation after 12hours of VPA creation:

![Screenshot 2021-11-23 at 8.48.32 AM](images/Screenshot%202021-11-23%20at%208.48.32%20AM.png)

=====

After 1 day:

After running some big queries for a period of time, memory limit raised from 3gb to 4gb.

![Screenshot 2021-11-25 at 5.17.13 PM](images/Screenshot%202021-11-25%20at%205.17.13%20PM.png)

![Screenshot 2021-11-25 at 5.19.41 PM](images/Screenshot%202021-11-25%20at%205.19.41%20PM.png)

![Screenshot%202021-11-25%20at%205.20.48%20PM](images/Screenshot%202021-11-25%20at%205.20.48%20PM.png)

