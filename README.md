# test-helm-subchart-uninstall-on-upgrade

What should happen if we `helm upgrade` to a chart version that no longer has one of the subcharts? Answer: the subchart is uninstalled on `helm upgrade`.

```console
$ k3d cluster create

$ helm install parentchart parentchart --wait

$ k get pods
NAME                                     READY   STATUS    RESTARTS   AGE
parentchart-d6fbdbb-95ldl                1/1     Running   0          49s
parentchart-subchart1-6876cff7b7-k5q2m   1/1     Running   0          49s
parentchart-subchart2-64d78f6fb4-srvsq   1/1     Running   0          49s

$ helm upgrade parentchart parentchart --wait --set subchart2.enabled=false
Release "parentchart" has been upgraded. Happy Helming!
NAME: parentchart
LAST DEPLOYED: Wed Nov 13 10:17:28 2024
NAMESPACE: default
STATUS: deployed
REVISION: 2

# subchart2 pod is gone
$ k get pods
NAME                                     READY   STATUS    RESTARTS   AGE
parentchart-d6fbdbb-95ldl                1/1     Running   0          74s
parentchart-subchart1-6876cff7b7-k5q2m   1/1     Running   0          74s

$ helm upgrade parentchart parentchart --wait
Release "parentchart" has been upgraded. Happy Helming!
NAME: parentchart
LAST DEPLOYED: Wed Nov 13 10:17:45 2024
NAMESPACE: default
STATUS: deployed
REVISION: 3

# subchart2 pod is not back because helm retains the values from the previous upgrade
$ k get pods
NAME                                     READY   STATUS    RESTARTS   AGE
parentchart-d6fbdbb-95ldl                1/1     Running   0          90s
parentchart-subchart1-6876cff7b7-k5q2m   1/1     Running   0          90s

$ helm upgrade parentchart parentchart --wait --reset-values
Release "parentchart" has been upgraded. Happy Helming!
NAME: parentchart
LAST DEPLOYED: Wed Nov 13 10:18:03 2024
NAMESPACE: default
STATUS: deployed
REVISION: 4

# subchart2 pod is back
$ k get pods
NAME                                     READY   STATUS    RESTARTS   AGE
parentchart-d6fbdbb-95ldl                1/1     Running   0          110s
parentchart-subchart1-6876cff7b7-k5q2m   1/1     Running   0          110s
parentchart-subchart2-64d78f6fb4-c9vcq   1/1     Running   0          4s

# Delete subchart2
$ rm -rf parentchart/charts/subchart2

# Comment out the subchart2 dependency
$ nano parentchart/Chart.yaml

$ helm upgrade parentchart parentchart --wait
Release "parentchart" has been upgraded. Happy Helming!
NAME: parentchart
LAST DEPLOYED: Wed Nov 13 10:19:08 2024
NAMESPACE: default
STATUS: deployed
REVISION: 5

# subchart2 pod is gone
$ k get pods
NAME                                     READY   STATUS    RESTARTS   AGE
parentchart-d6fbdbb-95ldl                1/1     Running   0          2m54s
parentchart-subchart1-6876cff7b7-k5q2m   1/1     Running   0          2m54s
```
