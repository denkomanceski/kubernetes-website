---
reviewers:
- verb
- yujuhong
title: Debug a Running Pod
content_template: templates/task
---

{{% capture overview %}}

{{< feature-state state="alpha" >}}

This page shows how to investigate problems in a running {{< glossary_tooltip term_id="pod" >}} using a
debugging Ephemeral Container. This is useful when the container being
debugged does not include debugging tools.

{{% /capture %}}

{{% capture prerequisites %}}

{{< include "task-tutorial-prereqs.md" >}} {{< version-check >}}

* You should be familiar with the basics of
  [ephemeral containers](/docs/concepts/workloads/pods/ephemeral-containers/).
* The examples in this section require the `EphemeralContainers` [feature
  gate](/docs/reference/command-line-tools-reference/feature-gates/) to be
  enabled and kubernetes client and server version v1.16 or later.
* Install [kubectl debug](https://github.com/verb/kubectl-debug) using the
  provided instructions.

{{% /capture %}}

{{% capture steps %}}

## Create a sample application

First, create a sample application to be used with the rest of the steps
in this task. This example uses a naked Pod for simplicity.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: sample-app
spec:
  shareProcessNamespace: true
  containers:
    - name: nginx
      image: nginx
```

Create the Pod described above in your cluster:

```shell
kubectl create -f sample-app.yaml
```

## Create an ephemeral container for debugging

Using the `kubectl debug` plugin referenced above, create an ephemeral container
for debugging:

```shell
kubectl debug sample-app
```

The new container now appears in the list of ephemeral containers in a `kubectl
describe`:

```shell
kubectl describe pod sample-app
```

```
Ephemeral Containers:
  debugger:
    Container ID:   docker://208144f2c4bb88e04b3bda7533346c2a606f19d1bd2231d14c236159140ea785
    Image:          busybox
    Image ID:       docker-pullable://busybox@sha256:9f1003c480699be56815db0f8146ad2e22efea85129b5b5983d0e0fb52d9ab70
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 29 Aug 2019 13:25:40 +0000
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:         <none>
```

* You can use an arbitrary container image, for example,
  `kubectl debug --image=$IMAGE_NAME`.
* Each successive ephemeral container needs a new name. `kubectl debug` defaults
  to using the name `debugger`. You must name additional containers differently
  using the `-c` flag to `kubectl debug`.

The busybox container runs `sh` and waits for you to attach using `kubectl`:

```shell
kubectl attach -it sample-app -c debugger
```

Once attached you are able to see processes from the other containers because
of process namespace sharing. For example:

```shell
% kubectl attach -it sample-app -c debugger
If you don't see a command prompt, try pressing enter.
/ # ps auxww
PID   USER     TIME  COMMAND
    1 root      0:00 /pause
    6 root      0:00 nginx: master process nginx -g daemon off;
   11 101       0:00 nginx: worker process
   12 root      0:00 sh
   17 root      0:00 ps auxww
```

The ephemeral container can be attached and re-attached as long as the shell
process is running.

Since ephemeral containers allow you to modify the state of the running
Pod, you should recreate the Pod so that it's in a known good state. When
using a {{< glossary_tooltip text="deployment" term_id="deployment" >}} this
is accomplished by deleting the Pod and allowing the
{{< glossary_tooltip text="replica set" term_id="replica-set" >}} to recreate
it.

```shell
kubectl delete pod sample-app
```

{{% /capture %}}

{{% capture whatsnext %}}

* Learn tips for interacting with processes in other containers in [Share
  Process Namespace between Containers in a Pod](/docs/tasks/configure-pod-container/share-process-namespace/)

{{% /capture %}}
