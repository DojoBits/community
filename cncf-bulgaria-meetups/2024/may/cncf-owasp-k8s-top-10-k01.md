
![CNCF Sofia Meetup](https://www.cncf.io/wp-content/uploads/2022/07/cncf-color-bg.svg)

# CNCF OWASP Top 10 Kubernetes K01

In `CNCF Sofia meetup 2024 May edition` you will have access to cloud based `Ubuntu 22.04` VM which is provided only
for the conference. The only thing that you need is computer with `ssh client` like
[mobaxterm](https://mobaxterm.mobatek.net/download.html).

# 0.0 Pre-requisites home setup ( skip at CNCF Sofia meetup )

For home setup you need a `Ubuntu 22.04` Virtual Machine. It's up to you to bring it up and you can use for example
[VirtualBox](https://www.virtualbox.org). We are lazy here and we use [vagrant](https://www.vagrantup.com) to bring up.

Keep in mind this is not the focus of the lecture and we will not go into details. `Vagrant` its a CLI tool to help you
bring your development env for seconds. You can even go crazy and setup more complicated setups. You can start with
`Vagrant` from [here](https://developer.hashicorp.com/vagrant/tutorials/getting-started)

## 0.1 High level steps to bring up the VM using Vagrant:

- Install `Vagrant`
- Install `VirtualBox` or `VMware` - for Virtual Machine provider
- Create a `Vagrantfile` with the following content: [Vagrantfile](https://app.vagrantup.com/generic/boxes/ubuntu2204)
- Run `vagrant up` and wait for the VM to be up and running.

Here is an example of the output of your `Vagrantfile`

```bash
~$ cat Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2204"
end
```


Here is our example ( yours will be different ) ours is using `VMware` as provider on Mac with M1 arm based processor

```bash
~$ vagrant up
Bringing machine 'default' up with 'vmware_desktop' provider...
==> default: Cloning VMware VM: 'hajowieland/ubuntu-jammy-arm'. This can take some time...
==> default: Checking if box 'hajowieland/ubuntu-jammy-arm' version '1.0.0' is up to date...
==> default: Verifying vmnet devices are healthy...
==> default: Preparing network adapters...
==> default: Starting the VMware VM...
==> default: Waiting for the VM to receive an address...
==> default: Forwarding ports...
    default: -- 22 => 2222
==> default: Waiting for machine to boot. This may take a few minutes...
    default: SSH address: 192.168.59.142:22
    default: SSH username: vagrant
SNIP ....
SNIP ....
==> default: Enabling and configuring shared folders...
    default: -- /Users/vhristev/Documents/gitlab_home/vagrant/rxm-labvm: /vagrant

```

Lets get into the VM:

```bash
~$ vagrant ssh
Welcome to Ubuntu 22.04.1 LTS (GNU/Linux 5.15.0-56-generic aarch64)

  System load:  0.080078125        Processes:               283
  Usage of /:   13.3% of 35.27GB   Users logged in:         0
  Memory usage: 3%                 IPv4 address for ens160: 192.168.59.142
  Swap usage:   0%


To check for new updates run: sudo apt update

vagrant@ubuntu:~$
```
## 0.2 Pre-requisites CNCF Sofia Meetup

At the conference you will receive a physical paper sheet with all the details you need for your VM. Please talk with
your instructor if you have any questions.

### 1. Install Docker

Lets start by installing Docker.

We will use Docker to run our containers. Here is a script to install it:

`cat install-docker.sh`

```bash
vagrant@ubuntu:~$ bash install-docker.sh
```


## Defeating OWASP Security Risks

The OWASP Kubernetes Top 10 was created to help SecDevOps, DevOps, SREs, and Developers to prioritize addressing risks
related to Kubernetes. The Top 10 risks range from workload misconfigurations, permissive RBAC policies, network
segmentation, and others. We are going to examine some of the Top 10 Kubernetes security risks of 2022 (the latest
available list) which are documented [here](https://owasp.org/www-project-kubernetes-top-ten/)


### 0. Pre-requisites

If you already know how to login to your lab system, `Docker` and `Kubernetes` are installed you can skip this section
and jump directly to `1.` . Otherwise, follow the instructions


### 0.1. Login to your lab system

Login to your machine using an ssh client:

```
@laptop:~$ chmod 400 key.pem

@laptop:~$ ssh -i key.pem ubuntu@55.55.55.55

Welcome to Ubuntu 22.04.2 LTS (GNU/Linux 5.15.0-1031-aws x86_64)

...

~$
```


### 0.2. Install Kubernetes

Before proceeding, make sure to have an active Kubernetes cluster on your VM. We provide a script to install Kubernetes
on your VM.

```
~$ wget -qO - https://raw.githubusercontent.com/cncf/classfiles/master/k8s.sh | sh

...

~$
```

We also add our account to the `docker` group to minimize use of `sudo`:

```
~$ sudo usermod -aG docker $(whoami)     # add yourself to the group

~$
```

Even though the _docker_ group was added to your user's group list, your login shell maintains the old groups. After
updating your user groups you will need to restart your login shell to ensure the changes take effect.

In the lab system the easiest approach is to use the `newgrp` command but you can also logout and login back in which
will refresh your user's group list.

```
~$ newgrp docker

~$
```

Let's check to see that your user shell session is now a part of the _docker_ group. If you don't see the docker group
please log out and log back in.

```
~$ groups

ubuntu adm dialout cdrom floppy sudo audio dip video plugdev netdev lxd docker

~$
```


### 0.3. Validation

Let's try to run `docker` without `sudo`:

```
~/$ docker version | head -2
Client: Docker Engine - Community
 Version:           24.0.4
```

Great. Your current shell, and any new shell sessions, can now use the `docker` command without `sudo`.


Validate the k8s cluster is up and running. It may take some time cluster to be `Ready` state but it should be up and
running very shortly.

```
~$ kubectl get node

NAME              STATUS   ROLES           AGE   VERSION
ip-172-31-20-48   Ready    control-plane   50s   v1.29.0

~$
```


### 0.4. Auto completion ( optional )

It is not just a good idea, but rather an essential requirement to have auto completion a.k.a. `<TAB>`. This could be
one of the reasons why Kubernetes prioritizes it as the foremost feature in their
[cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/).

Add autocomplete permanently to your bash shell.

```
~$ echo "source <(kubectl completion bash)" >> ~/.bashrc
```

Many user will use `k` as an alias for `kubectl`. Other use `kc`, `kctl` its up to you. The example below uses `k`.
Again this is a personal preference.

```
~$ echo "alias k=kubectl" >> ~/.bashrc

~$ echo "complete -o default -F __start_kubectl k" >> ~/.bashrc
```

Refresh the shell to make sure the changes take effect:

```
~$ source ~/.bashrc
```

Try it out. You just need to type some command and pres `<TAB>`. In the example below the tab key is pressed after
typing `k get pods --all<TAB>` and it provides suggestions:

```
~$ k get pods  --all

--all-namespaces               (If present, list the requested object(s) across all namespace…)
--allow-missing-template-keys  (If true, ignore any errors in templates when a field or map k…)

~$
```


### 1. OWASP K01 - insecure workload configurations

The overview of the OWASP K01 risk states:

> The security context of a workload in Kubernetes is highly configurable which can lead to serious security
> misconfigurations propagating across an organization’s workloads and clusters. The Kubernetes adoption, security, and
> market trends report 2022 from Redhat stated that nearly 53% of respondents have experienced a misconfiguration
> incident in their Kubernetes environments in the last 12 months.

Let's create a container which has some bad security practices to illustrate this risk:

```
~$ mkdir k01 && cd $_

~/k01$ nano priv-pod.yaml && cat $_
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: root-user
spec:
  # Bad host volume mount
  volumes:
  - name: nicehost
    hostPath:
      path: /
  # Host PID can lead to container breakouts
  hostPID: true
  containers:
  - name: root-user
    image: busybox
    command: ["sleep", "3600"]
    volumeMounts:
    - name: nicehost
      mountPath: /host
    securityContext:
      # priviliged enables all system calls
      privileged: true
      # root user - obvious problem
      runAsUser: 0
```
```
~/k01$
```

Let's review the security issues with this configuration:

- **Host volume mount**: The container mounts the host's root filesystem (`path: /`). This provides the container full
  read-write access to the host's file system, which is a significant security risk.

- **Host PID namespace**: The configuration specifies that the container should share the host's PID namespace
  (`hostPID: true`). This allows processes within the container to see and interact with all processes on the host, as
  well as the `/proc` filesystem where secrets exist!

- **Privileged mode**: The container is running in privileged mode (`privileged: true`). This means it has access to all
  devices on the host and can make any system call to the kernel.

- **Running as Root**: In the `securityContext`, the container is configured to run as root (`runAsUser: 0`). Running
  containers as root can potentially escalate to root on the host, especially if combined with `privileged: true`.

All of these settings significantly reduce the isolation between the container and the host, and they should be avoided
unless absolutely necessary. As a general rule, containers should be run with the least amount of privilege necessary to
accomplish their tasks.

```
~/k01$ kubectl apply -f priv-pod.yaml

pod/root-user created

~/k01$ kubectl get po

NAME        READY   STATUS    RESTARTS   AGE
root-user   1/1     Running   0          3s

~/k01$
```

What you can do is absolutely powerful. You can run a container as root, and then mount the host filesystem into the
container. Let's see what we can do with this container:

```
~/k01$ kubectl exec -it root-user -- sh

/ # id

uid=0(root) gid=0(root) groups=0(root),10(wheel)

/ #
```

We mounted the host filesystem into the container, and we are running as root. We can check the `shadow` file which
contains the password hashes of all the users on the host:

```
/ # head -5 /host/etc/shadow

root:*:19516:0:99999:7:::
daemon:*:19516:0:99999:7:::
bin:*:19516:0:99999:7:::
sys:*:19516:0:99999:7:::
sync:*:19516:0:99999:7:::

/ #
```

We can list our entire host filesystem:

```
/ # ls /host/

bin         home        libx32      opt         sbin        tmp
boot        lib         lost+found  proc        snap        usr
dev         lib32       media       root        srv         var
etc         lib64       mnt         run         sys

/ #
```

What you are looking is the host root ( `/` ) filesystem.

You can create files:

```
/ # echo 'Sneaky' >> /host/gothacked.txt

/ # ls -l /host/gothacked.txt

-rw-r--r--    1 root     root             7 Jun 24 20:49 /host/gothacked.txt

/ # cat /host/gothacked.txt

Sneaky

/ #
```

You can see all host processes as well:

```
/ # ps aux

PID   USER     TIME  COMMAND
    1 root      0:11 /lib/systemd/systemd --system --deserialize 35
    2 root      0:00 [kthreadd]
SNIP...
SNIP...
  237 root      0:00 [cryptd]
  498 root      0:00 /usr/sbin/acpid
  501 root      0:00 /usr/sbin/cron -f -P
  503 102       0:01 {dbus-daemon} @dbus-daemon --system --address=systemd: --nofork --nopidfi
-567 65535     0:00 /pause
-571 root      0:00 /usr/bin/containerd-shim-runc-v2 -namespace k8s.io -id d24131ac8ea18aba90
-601 65535     0:00 /pause
-629 root      0:00 /usr/local/bin/kube-proxy --config=/var/lib/kube-proxy/config.conf --host
-836 root      0:00 bpfilter_umh
-119 root      0:00 /usr/bin/weave-npc
-152 root      0:00 /usr/sbin/ulogd -v
-234 root      0:00 {launch.sh} /bin/sh /home/weave/launch.sh
-309 root      0:00 /home/weave/weaver --port=6783 --datapath=datapath --name=8a:f8:ab:42:d9:
-455 root      0:00 /home/weave/kube-utils -run-reclaim-daemon -node-name=ip-172-31-16-208 -p
-523 root      0:00 /usr/bin/containerd-shim-runc-v2 -namespace k8s.io -id c57d04f08b1467c008
-542 65535     0:00 /pause
-575 root      0:01 /coredns -conf /etc/coredns/Corefile
-651 root      0:00 /usr/bin/containerd-shim-runc-v2 -namespace k8s.io -id 7572754e4dcdc81f04
-672 65535     0:00 /pause
-708 root      0:01 /coredns -conf /etc/coredns/Corefile
-944 1000      0:00 bash
-236 root      0:00 [kworker/u4:2-ev]
-284 root      0:00 [kworker/0:3-rcu]
-329 root      0:00 [kworker/1:0-eve]
-420 root      0:00 /usr/bin/containerd-shim-runc-v2 -namespace k8s.io -id 633b6db3ecbd83380a
-439 65535     0:00 /pause
-509 root      0:00 sleep 3600
-582 1000      0:00 kubectl exec -it root-user -- /bin/sh
-598 root      0:00 /bin/sh
-703 root      0:00 ps aux

...

/ #
```

Another attack is to use the _container runtime socket_ to gain access to other containers running on this host. We can
use two approaches to find it:

- Leveraging our access to the host PID namespace and getting info from running processes
- Using our access to the root filesystem on the host and searching for config file(s)

List processes that are referencing a unix socket file:

```
/ # ps aux | grep sock

...

  446 root     32:50 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --container-runtime-endpoint=unix:///var/run/containerd/containerd.sock --pod-infra-container-image=registry.k8s.io/pause:3.9

/ #
```

Among our results should be an entry like the example above; the Kubernetes kubelet daemon communicates to a container
runtime using the `--container-runtime-endpoint=` configuration flag, in our case it points to
`/var/run/containerd/containerd.sock`

We have learned 2 things: what the container runtime is: `containerd` and where it is listening:
`/var/run/containerd/containerd.sock`.

Our 2nd approach, we can find containerd's config typically found in the `/etc` directory on a host:

```
/ # ls -l /host/etc/containerd/

total 12
-rw-r--r--    1 root     root           886 Jul  6 01:52 config.bak
-rw-r--r--    1 root     root          6993 Jul  6 01:52 config.toml

/ #
```

There is its config (`.toml`) file! We can confirm that it is configured to listen on a unix socket by searching the config:

```
/ # cat /host/etc/containerd/config.toml  | grep sock

  address = "/run/containerd/containerd.sock"

/ #
```

Confirmed! The socket is located at `/run/containerd/containerd.sock`.

```
/ # exit

~/k01$
```

Since our runtime is `containerd` we can use the containerd cli (`crictl`) to control the runtime.

Create a new container which has the `crictl` binary installed:

```
~/k01$ nano crictl-pod.yaml && cat $_
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: root-user-crictl
spec:
  # Bad host volumene mount
  volumes:
  - name: nicehost
    hostPath:
      path: /
  # Host PID
  hostPID: true
  containers:
  - name: root-user-crictl
    image: docker.io/rancher/crictl:v1.19.0
    command: ["sleep", "3600"]
    volumeMounts:
    - name: nicehost
      mountPath: /host
    securityContext:
      # priviliged
      privileged: true
      # root user:
      runAsUser: 0
```
```
~/k01$
```

Create our crictl pod.

```
~/k01$ kubectl apply -f crictl-pod.yaml

pod/root-user-crictl created

~/k01$
```

Check if our pod is running:

```
~/k01$ kubectl get po

NAME               READY   STATUS    RESTARTS   AGE
root-user          1/1     Running   0          3m37s
root-user-crictl   1/1     Running   0          10s

~/k01$
```

`exec` into the crictl pod:

```
~/k01$ kubectl exec -it root-user-crictl -- sh

sh-4.2#
```

Use our `clictl` cli tool to connect to the containerd socket with the following options:

- `--runtime-endpoint` is the containerd socket on our host which is why we put `/host` in front of it (we mounted our
  host filesystem under `/host`` inside our container)
- `pods` - command to list all pods


```
sh-4.2# crictl --runtime-endpoint="unix:///host/run/containerd/containerd.sock" pods

POD ID              CREATED             STATE               NAME                                       NAMESPACE           ATTEMPT             RUNTIME
8059109da767b       4 minutes ago       Ready               root-user-crictl                           default             0                   (default)
9c71ddc705548       30 minutes ago      Ready               root-user                                  default             0                   (default)
695a7de2098b6       30 hours ago        Ready               coredns-5d78c9869d-8gnsl                   kube-system         2                   (default)
d7ca185d926cf       30 hours ago        Ready               coredns-5d78c9869d-hfrtv                   kube-system         2                   (default)
633345bc67b49       30 hours ago        Ready               weave-net-h5srj                            kube-system         1                   (default)
4c2192f8b4edd       30 hours ago        Ready               kube-proxy-6s59q                           kube-system         1                   (default)
281e24c298c45       30 hours ago        Ready               etcd-ip-172-31-24-124                      kube-system         1                   (default)
0dc6a9e8c99d6       30 hours ago        Ready               kube-scheduler-ip-172-31-24-124            kube-system         1                   (default)
14c3a7172b6d3       30 hours ago        Ready               kube-apiserver-ip-172-31-24-124            kube-system         1                   (default)
8e074ee007710       30 hours ago        Ready               kube-controller-manager-ip-172-31-24-124   kube-system         1                   (default)
e28584987f70b       2 days ago          NotReady            coredns-5d78c9869d-hfrtv                   kube-system         1                   (default)
7a7d2310cca8a       2 days ago          NotReady            coredns-5d78c9869d-8gnsl                   kube-system         1                   (default)
4b57f6b3e0931       2 days ago          NotReady            kube-proxy-6s59q                           kube-system         0                   (default)
251bb54dc8977       2 days ago          NotReady            weave-net-h5srj                            kube-system         0                   (default)
f9a794a5cca51       2 days ago          NotReady            kube-scheduler-ip-172-31-24-124            kube-system         0                   (default)
50c69e6dd09c0       2 days ago          NotReady            kube-apiserver-ip-172-31-24-124            kube-system         0                   (default)
56bae0d26919f       2 days ago          NotReady            kube-controller-manager-ip-172-31-24-124   kube-system         0                   (default)
6f6b622982081       2 days ago          NotReady            etcd-ip-172-31-24-124                      kube-system         0                   (default)

sh-4.2#
```

From here you can use your imagination about what you can do because you are `root` and you have full control over the host
and its containers.

Exit the pod

```
sh-4.2# exit

~/k01$
```


### 2. OWASP K01 - Prevention

The "How to Prevent" section of the OWASP K01 risk states:

> In order to prevent misconfigurations, they must first be detected in both runtime and in code.

Let's fix our pod to be more secure; for this job we will use the static code analysis approach to find the issues in
our podSpec ("in code" as mentioned in the risk text). Ideally we would use these tools in our **CI/CD** pipeline to
prevent insecure pods from being deployed.

We can also use Kubernetes Admission Control as a second line of defense (at "runtime" as mentioned in the OWASP text)
to prevent insecure pods from being deployed if our linter(s) and/or static analysis tooling fails (beyond the scope of
this lab).


### 2.1. OWASP K01 - Kube-score

We will install a tool called **kube-score** which scans Kubernetes manifests for improved security and resiliency based
on kube-score's [checks](https://github.com/zegl/kube-score/blob/master/README_CHECKS.md). Some of these checks include:

- Pods have resource limits and requests set
- Pods have the same requests as limits on resources set
- Container images uses an explicit non-latest tag
- Ingresses targets a Service
- CronJobs has a configured deadline
- NetworkPolicies targets at least one Pod

Download the archive, extract it, and move it to your path:

```
~/k01$ wget -q https://github.com/zegl/kube-score/releases/download/v1.18.0/kube-score_1.17.0_linux_amd64.tar.gz

~/k01$ tar -zxf kube-score_*_linux_amd64.tar.gz

~/k01$ sudo mv kube-score /usr/local/bin

~/k01$ kube-score version

kube-score version: 1.18.0, commit: 0fb5f668e153c22696aa75ec769b080c41b5dd3d, built: 2024-02-05T14:13:15Z

~/k01$
```

Let's run `kube-score` against our pod definition and see what it says:

```
~/k01$ kube-score score priv-pod.yaml

v1/Pod root-user                                                              💥
    [CRITICAL] Pod NetworkPolicy
        · The pod does not have a matching NetworkPolicy
            Create a NetworkPolicy that targets this pod to control who/what can communicate
            with this pod. Note, this feature needs to be supported by the CNI implementation
            used in the Kubernetes cluster to have an effect.
    [CRITICAL] Container Security Context User Group ID
        · root-user -> The container is running with a low user ID
            A userid above 10 000 is recommended to avoid conflicts with the host. Set
            securityContext.runAsUser to a value > 10000
        · root-user -> The container running with a low group ID
            A groupid above 10 000 is recommended to avoid conflicts with the host. Set
            securityContext.runAsGroup to a value > 10000
    [CRITICAL] Container Security Context ReadOnlyRootFilesystem
        · root-user -> The pod has a container with a writable root filesystem
            Set securityContext.readOnlyRootFilesystem to true
    [CRITICAL] Container Image Tag
        · root-user -> Image with latest tag
            Using a fixed tag is recommended to avoid accidental upgrades
    [CRITICAL] Container Resources
        · root-user -> CPU limit is not set
            Resource limits are recommended to avoid resource DDOS. Set resources.limits.cpu
        · root-user -> Memory limit is not set
            Resource limits are recommended to avoid resource DDOS. Set
            resources.limits.memory
        · root-user -> CPU request is not set
            Resource requests are recommended to make sure that the application can start and
            run without crashing. Set resources.requests.cpu
        · root-user -> Memory request is not set
            Resource requests are recommended to make sure that the application can start and
            run without crashing. Set resources.requests.memory
    [CRITICAL] Container Ephemeral Storage Request and Limit
        · root-user -> Ephemeral Storage limit is not set
            Resource limits are recommended to avoid resource DDOS. Set
            resources.limits.ephemeral-storage
    [CRITICAL] Container Security Context Privileged
        · root-user -> The container is privileged
            Set securityContext.privileged to false. Privileged containers can access all
            devices on the host, and grants almost the same access as non-containerized
            processes on the host.

~/k01$
```

Let's discuss the issues and how we can fix them:

- Set the `hostPID` to false, so the Pod won't share the host's PID namespace
- Remove `privileged` and change the value of `runAsUser` in the `securityContext` so the container will now as a
  `non-root` user with the user ID and group ID of `10001`.
- Add `readOnlyRootFilesystem: true` to make the container's filesystem read-only
- Pull an image by FQIN with a pinned version `docker.io/busybox:1.36.1` instead of using the latest tag (commented out).
  As example of image we will still use the cri-tools to show you what is the result of the changes.
- Set both resource requests and limits for CPU, memory, and ephemeral storage to mitigate potential resource
  exhaustion.
- Set the `imagePullPolicy` to Always. This ensures that Kubernetes always pulls the image from the registry to make
  sure it's using the correct version.
- Remove the `hostPath` volume and set ephemeral-storage ( or other volumes ) with a limit. This ensures that your
  application has enough ephemeral storage to start and run without crashing.

Let's fix the issues by creating a new pod definition file called `good-pod.yaml`:

```
~/k01$ nano good-pod.yaml && cat $_
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: safer-pod
spec:
  hostPID: false
  volumes:
  - name: safer-volume
    emptyDir: {}
  containers:
  - name: safer-container
    # image: docker.io/busybox:1.36.1
    image: docker.io/rancher/crictl:v1.19.0
    imagePullPolicy: Always
    command: ["sleep", "3600"]
    volumeMounts:
    - name: safer-volume
      mountPath: /host
    securityContext:
      privileged: false
      runAsUser: 10001
      runAsGroup: 10001
      readOnlyRootFilesystem: true
    resources:
      limits:
        cpu: "1"
        memory: "1Gi"
        ephemeral-storage: "1Gi"
      requests:
        cpu: "500m"
        memory: "500Mi"
        ephemeral-storage: "500Mi"
```
```
~/k01$
```

Run `kube-score` again and see what it says after our changes.

```
~/k01$ kube-score score good-pod.yaml

v1/Pod safer-pod                                                              💥
    [CRITICAL] Pod NetworkPolicy
        · The pod does not have a matching NetworkPolicy
            Create a NetworkPolicy that targets this pod to control who/what can communicate
            with this pod. Note, this feature needs to be supported by the CNI implementation
            used in the Kubernetes cluster to have an effect.

~/k01$
```

As we can see all the critical issues are gone. The only one left is the `NetworkPolicy` which is beyond the scope of
this lab.


### 2.2. OWASP K01 - KubeLinter

Let's install another static code analysis tool called **KubeLinter**. KubeLinter is a Go-based static analsys tool from
StackRox.

Download the archive, extract it, and move it to your path:

```
~/k01$ wget -q https://github.com/stackrox/kube-linter/releases/download/v0.6.8/kube-linter-linux.tar.gz

~/k01$ tar zxf kube-linter-linux.tar.gz

~/k01$ sudo mv kube-linter /usr/local/bin/

~/k01$ kube-linter version

v0.6.8

~/k01$
```

The default KubeLinter checks can be summarized as:

- Service selectors should match one deployment labels
- Deployments that use the deprecated serviceAccount field instead of the serviceAccountName field
- Secrets used in an environment variable instead of as a file or secretKeyRef
- Deployments  selector doesn't match the pod template labels
- Deployments with multiple replicas that don't specify inter pod anti-affinity
- Objects that use deprecated API versions under extensions v1beta
- Containers not running with a read-only root filesystem
- Pods that reference a non-existing service account
- Deployments with containers that run in privileged mode
- Containers not set to runAsNonRoot
- Deployments that expose port 22, commonly reserved for SSH access
- Containers without CPU requests and limits
- Containers without memory requests and limits

Let's run `kube-linter` on our `priv` and `good` pod definitions. First we will run it against the `priv-pod.yaml` file:

```
~/k01$ kube-linter lint priv-pod.yaml

KubeLinter v0.6.8

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) object shares the host's process namespace (via hostPID=true). (check: host-pid, remediation: Ensure the host's process namespace is not shared.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) The container "root-user" is using an invalid container image, "busybox". Please use images that are not blocked by the `BlockList` criteria : [".*:(latest)$" "^[^:]*$" "(.*/[^:]+)$"] (check: latest-tag, remediation: Use a container image with a specific tag other than latest.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" does not have a read-only root file system (check: no-read-only-root-fs, remediation: Set readOnlyRootFilesystem to true in the container securityContext.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" is Privileged hence allows privilege escalation. (check: privilege-escalation-container, remediation: Ensure containers do not allow privilege escalation by setting allowPrivilegeEscalation=false, privileged=false and removing CAP_SYS_ADMIN capability. See https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ for more details.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" is privileged (check: privileged-container, remediation: Do not run your container as privileged unless it is required.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" is not set to runAsNonRoot (check: run-as-non-root, remediation: Set runAsUser to a non-zero number and runAsNonRoot to true in your pod or container securityContext. Refer to https://kubernetes.io/docs/tasks/configure-pod-container/security-context/ for details.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) host system directory "/" is mounted on container "root-user" (check: sensitive-host-mounts, remediation: Ensure sensitive host system directories are not mounted in containers by removing those Volumes and VolumeMounts.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" has cpu request 0 (check: unset-cpu-requirements, remediation: Set CPU requests and limits for your container based on its requirements. Refer to https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits for details.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" has cpu limit 0 (check: unset-cpu-requirements, remediation: Set CPU requests and limits for your container based on its requirements. Refer to https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits for details.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" has memory request 0 (check: unset-memory-requirements, remediation: Set memory requests and limits for your container based on its requirements. Refer to https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits for details.)

/home/ubuntu/k01/priv-pod.yaml: (object: <no namespace>/root-user /v1, Kind=Pod) container "root-user" has memory limit 0 (check: unset-memory-requirements, remediation: Set memory requests and limits for your container based on its requirements. Refer to https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#requests-and-limits for details.)

Error: found 11 lint errors

~/k01$
```

We can see that `kube-linter` also give us a lot of issues and also give us some remediation steps. Let's check our
`good-pod.yaml` file.

```
~/k01$ kube-linter lint good-pod.yaml

KubeLinter v0.6.8

No lint errors found!

~/k01$
```

KubeLinter doesn't complain about a missing network policy like kube-score did. Not all linting tools are created equal.

Deploy the `good-pod.yaml` file and see if it works on our cluster

```
~/k01$ kubectl apply -f good-pod.yaml

pod/safer-pod created

~/k01$
```

Let's check if the pod is running.

```
~/k01$ kubectl get po

NAME               READY   STATUS    RESTARTS   AGE
root-user          1/1     Running   0          15m
root-user-crictl   1/1     Running   0          4m26s
safer-pod          1/1     Running   0          6s
```

`exec` inside the pod and check if we are root:

```
~/k01$ kubectl exec -it safer-pod -- /bin/sh

sh-4.2$ id

uid=10001 gid=10001 groups=10001

sh-4.2$
```

As you can see we are not root inside the container

Check if we have access to the host's containerd socket. We will use the same command as we did before:

```
sh-4.2$ crictl --runtime-endpoint="unix:///host/run/containerd/containerd.sock" pods

FATA[0002] connect: connect endpoint 'unix:///host/run/containerd/containerd.sock', make sure you are running as root and the endpoint has been started: context deadline exceeded

sh-4.2$
```

We don't have the `/host` volume mounted in the container which is why we are not able to access the host's containerd
socket. Also to be able to do that we need the `HostPID` to be set to true. The `ps` command (if available) will just
output container level processes.

```
sh-4.2$ exit

exit

~/k01$
```

Maintaining robust security configurations across a sprawling Kubernetes environment can pose a considerable challenge.
A major portion of these configurations are often established in the manifest's `securityContext`. However, there's
also a range of other potential misconfigurations that may be found elsewhere. To prevent these misconfigurations, it's
crucial to detect them both in real-time operation *and* within the code itself. Consequently, we should enforce the
following policies for our applications:

- Operations should be run as a non-root user
- Applications should be run in non-privileged mode
- We should ensure `AllowPrivilegeEscalation` is set to False to prevent child processes from obtaining higher
  privileges than their parent processes

Various tools like the Open Policy Agent can serve as a policy engine to identify these prevalent misconfigurations.
Additionally, the CIS Benchmark for Kubernetes can provide an initial reference point for identifying potential
misconfigurations.


### 3. OWASP K02 - Supply chain vulnerabilities

The overview of the OWASP K02 risk states:

> Containers take on many forms at different phases of the development lifecycle supply chain; each of them presenting
> unique security challenges. A single container alone can rely on hundreds of third-party components and dependencies
> making trust of origin at each phase extremely difficult. These challenges include but are not limited to image
> integrity, image composition, and known software vulnerabilities.

In this section of the lab we will scan our container images for known vulnerabilities. Let's start with creating a
simple Python container image that prints a message. We will deliberately use an older version of Python to potentially
introduce vulnerabilities that can be detected by a scanner.


### 3.1. OWASP K02 - Create a Python Dockerfile

Create our working directory and move into it:

```
~/k01$ mkdir ~/k02 && cd $_

~/k02$
```

First, let's create a Dockerfile for our Python application:

```
~/k02$ nano Dockerfile && cat $_
```
```Dockerfile
# We are deliberately using an older version of Python
FROM docker.io/python:3.9
WORKDIR /app
COPY app.py .
CMD ["python", "app.py"]
```
```
~/k02$
```

Next, we'll create a very simple Python script that just prints a message. Save this as `app.py`:

```
~/k02$ nano app.py && cat $_
```
```python
print("CNCF old and vulnerable python image!")
```
```
~/k02$
```


### 3.2.  OWASP K02 - Build the image

We can build this image using the following command:

```
~/k02$ docker image build -t cncf-vuln:1.0.0 .

[+] Building 28.0s (9/9) FINISHED                                              docker:default

...

 => => naming to docker.io/library/cncf-vuln:1.0.0                                       0.0s


~/k02$
```

At this point, we have a container image for a simple Python app based on an older Python version. Now let's move on to
installing and using Trivy to scan this image for vulnerabilities


### 3.3. OWASP K02 - Install Trivy

Aqua Security's Trivy is a container image (and other artifacts) vulnerability scanner.

Trivy detects vulnerabilities in:

- OS packages (e.g. Alpine, Ubuntu, RHEL, CentOS)
- Application dependencies (e.g. npm, pipenv, Composer, Bundler)

Trivy can scan container images:

- on the local machine in Docker Engine
- on a remote Docker registry e.g. Docker Hub, ECR, GCR, ACR
- tarball stored in the `docker save` formatted file
- an OCI Image Format compliant image directory
- local filesystem
- in a remote git repo

On Ubuntu, you can install Trivy with the following commands:

> N.B. if you receive a message like
> `Warning: apt-key is deprecated. Manage keyring files in trusted.gpg.d instead (see apt-key(8)).` it can safely be
> ignored.

```
~/k02$ sudo apt update && sudo apt install wget apt-transport-https gnupg lsb-release

~/k02$ wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

OK

~/k02$ echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

deb https://aquasecurity.github.io/trivy-repo/deb jammy main

~/k02$ sudo apt update && sudo apt install trivy -y

Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  trivy

...

No user sessions are running outdated binaries.

No VM guests are running outdated hypervisor (qemu) binaries on this host.

~/k02$
```


### 3.4. OWASP K02 - Scan the image with Trivy

Now we can scan our image using Trivy (_be patient as this may take some time_):

```
~/k20$ trivy image cncf-vuln:1.0.0

2024-01-27T02:36:33.590Z	INFO	Need to update DB
2024-01-27T02:36:33.590Z	INFO	DB Repository: ghcr.io/aquasecurity/trivy-db
2024-01-27T02:36:33.590Z	INFO	Downloading DB...
42.56 MiB / 42.56 MiB [--------------------------------------------------------------------------------------------------------------------------------------] 100.00% 14.60 MiB p/s 3.1s
2024-01-27T02:36:36.908Z	INFO	Vulnerability scanning is enabled
2024-01-27T02:36:36.908Z	INFO	Secret scanning is enabled
2024-01-27T02:36:36.908Z	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-01-27T02:36:36.908Z	INFO	Please see also https://aquasecurity.github.io/trivy/v0.48/docs/scanner/secret/#recommendation for faster secret detection
2024-01-27T02:37:41.304Z	INFO	Detected OS: debian
2024-01-27T02:37:41.304Z	INFO	Detecting Debian vulnerabilities...
2024-01-27T02:37:41.694Z	INFO	Number of language-specific files: 1
2024-01-27T02:37:41.694Z	INFO	Detecting python-pkg vulnerabilities...

cncf-vuln:1.0.0 (debian 12.4)

Total: 817 (UNKNOWN: 3, LOW: 510, MEDIUM: 236, HIGH: 65, CRITICAL: 3)

┌──────────────────────────────┬─────────────────────┬──────────┬──────────────┬─────────────────────────┬───────────────┬──────────────────────────────────────────────────────────────┐
│           Library            │    Vulnerability    │ Severity │    Status    │    Installed Version    │ Fixed Version │                            Title                             │
├──────────────────────────────┼─────────────────────┼──────────┼──────────────┼─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ apt                          │ CVE-2011-3374       │ LOW      │ affected     │ 2.6.1                   │               │ It was found that apt-key in apt, all versions, do not       │
│                              │                     │          │              │                         │               │ correctly...                                                 │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2011-3374                    │
├──────────────────────────────┼─────────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ bash                         │ TEMP-0841856-B18BAF │          │              │ 5.2.15-2+b2             │               │ [Privilege escalation possible to other user than root]      │
│                              │                     │          │              │                         │               │ https://security-tracker.debian.org/tracker/TEMP-0841856-B1- │
│                              │                     │          │              │                         │               │ 8BAF                                                         │
├──────────────────────────────┼─────────────────────┤          │              ├─────────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ binutils                     │ CVE-2017-13716      │          │              │ 2.40-2                  │               │ binutils: Memory leak with the C++ symbol demangler routine  │
│                              │                     │          │              │                         │               │ in libiberty                                                 │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2017-13716                   │
│                              ├─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                              │ CVE-2018-18483      │          │              │                         │               │ binutils: Integer overflow in cplus-dem.c:get_count() allows │
│                              │                     │          │              │                         │               │ for denial of service                                        │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2018-18483                   │
│                              ├─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                              │ CVE-2018-20673      │          │              │                         │               │ libiberty: Integer overflow in demangle_template() function  │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2018-20673                   │
│                              ├─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                              │ CVE-2018-20712      │          │              │                         │               │ libiberty: heap-based buffer over-read in d_expression_1     │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2018-20712                   │
│                              ├─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                              │ CVE-2018-9996       │          │              │                         │               │ binutils: Stack-overflow in libiberty/cplus-dem.c causes     │
│                              │                     │          │              │                         │               │ crash                                                        │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2018-9996                    │
│                              ├─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                              │ CVE-2021-32256      │          │              │                         │               │ stack-overflow issue in demangle_type in rust-demangle.c.    │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2021-32256                   │
│                              ├─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│                              │ CVE-2023-1972       │          │              │                         │               │ binutils: Illegal memory access when accessing a             │
│                              │                     │          │              │                         │               │ zer0-lengthverdef table                                      │
│                              │                     │          │              │                         │               │ https://avd.aquasec.com/nvd/cve-2023-1972                    │
├──────────────────────────────┼─────────────────────┤          │              │                         ├───────────────┼──────────────────────────────────────────────────────────────┤
│ binutils-common              │ CVE-2017-13716      │          │              │                         │               │ binutils: Memory leak with the C++ symbol demangler routine  │

...

~/k02$
```

Here you will see a lot of output because of the huge amount of vulnerabilities:
`Total: 817 (UNKNOWN: 3, LOW: 510, MEDIUM: 236, HIGH: 65, CRITICAL: 3)`

Just for reference if we change our application's base python image from `python:3.9` to `python:3.10-slim` we end up
with 94 vulnerabilities. (Keep in mind that you dont have that image built--if you want to test it you have to build it
first.)

Here's an example of scanning with the `python:3.10-slim` image:

```
~/k02$ trivy image cncf-vuln:2.0.0

2024-01-27T02:40:15.665Z	INFO	Vulnerability scanning is enabled
2024-01-27T02:40:15.666Z	INFO	Secret scanning is enabled
2024-01-27T02:40:15.666Z	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-01-27T02:40:15.666Z	INFO	Please see also https://aquasecurity.github.io/trivy/v0.48/docs/scanner/secret/#recommendation for faster secret detection
2024-01-27T02:40:26.543Z	INFO	Detected OS: debian
2024-01-27T02:40:26.543Z	INFO	Detecting Debian vulnerabilities...
2024-01-27T02:40:26.580Z	INFO	Number of language-specific files: 1
2024-01-27T02:40:26.580Z	INFO	Detecting python-pkg vulnerabilities...

cncf-vuln:2.0.0 (debian 12.4)

Total: 94 (UNKNOWN: 0, LOW: 67, MEDIUM: 22, HIGH: 4, CRITICAL: 1)

┌────────────────────┬─────────────────────┬──────────┬──────────────┬───────────────────┬───────────────┬──────────────────────────────────────────────────────────────┐
│      Library       │    Vulnerability    │ Severity │    Status    │ Installed Version │ Fixed Version │                            Title                             │
├────────────────────┼─────────────────────┼──────────┼──────────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ apt                │ CVE-2011-3374       │ LOW      │ affected     │ 2.6.1             │               │ It was found that apt-key in apt, all versions, do not       │
│                    │                     │          │              │                   │               │ correctly...                                                 │
│                    │                     │          │              │                   │               │ https://avd.aquasec.com/nvd/cve-2011-3374                    │
├────────────────────┼─────────────────────┤          │              ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ bash               │ TEMP-0841856-B18BAF │          │              │ 5.2.15-2+b2       │               │ [Privilege escalation possible to other user than root]      │
│                    │                     │          │              │                   │               │ https://security-tracker.debian.org/tracker/TEMP-0841856-B1- │
│                    │                     │          │              │                   │               │ 8BAF                                                         │
├────────────────────┼─────────────────────┤          │              ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ bsdutils           │ CVE-2022-0563       │          │              │ 1:2.38.1-5+b1     │               │ util-linux: partial disclosure of arbitrary files in chfn    │
│                    │                     │          │              │                   │               │ and chsh when compiled...                                    │
│                    │                     │          │              │                   │               │ https://avd.aquasec.com/nvd/cve-2022-0563                    │
├────────────────────┼─────────────────────┤          ├──────────────┼───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ coreutils          │ CVE-2016-2781       │          │ will_not_fix │ 9.1-1             │               │ coreutils: Non-privileged session can escape to the parent   │
│                    │                     │          │              │                   │               │ session in chroot                                            │
│                    │                     │          │              │                   │               │ https://avd.aquasec.com/nvd/cve-2016-2781                    │
│                    ├─────────────────────┤          ├──────────────┤                   ├───────────────┼──────────────────────────────────────────────────────────────┤
│                    │ CVE-2017-18018      │          │ affected     │                   │               │ coreutils: race condition vulnerability in chown and chgrp   │
│                    │                     │          │              │                   │               │ https://avd.aquasec.com/nvd/cve-2017-18018                   │
├────────────────────┼─────────────────────┼──────────┤              ├───────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ gcc-12-base        │ CVE-2023-4039       │ MEDIUM   │              │ 12.2.0-14         │               │ gcc: -fstack-protector fails to guard dynamic stack          │

...

~/k02$
```

Trivy will analyze the image and report any found vulnerabilities. You can use this report to understand the risks
associated with the outdated Python version and take appropriate actions.

You can also filter the `severity` to minimize the output and focus the report:

```
~/k20$ trivy --severity HIGH,CRITICAL image cncf-vuln:1.0.0

2024-01-27T02:41:40.238Z	INFO	Vulnerability scanning is enabled
2024-01-27T02:41:40.238Z	INFO	Secret scanning is enabled
2024-01-27T02:41:40.238Z	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-01-27T02:41:40.238Z	INFO	Please see also https://aquasecurity.github.io/trivy/v0.48/docs/scanner/secret/#recommendation for faster secret detection
2024-01-27T02:41:40.327Z	INFO	Detected OS: debian
2024-01-27T02:41:40.328Z	INFO	Detecting Debian vulnerabilities...
2024-01-27T02:41:40.595Z	INFO	Number of language-specific files: 1
2024-01-27T02:41:40.595Z	INFO	Detecting python-pkg vulnerabilities...

cncf-vuln:1.0.0 (debian 12.4)

Total: 68 (HIGH: 65, CRITICAL: 3)

┌─────────────────────────────┬────────────────┬──────────┬──────────────┬──────────────────────┬───────────────┬──────────────────────────────────────────────────────────────┐
│           Library           │ Vulnerability  │ Severity │    Status    │  Installed Version   │ Fixed Version │                            Title                             │
├─────────────────────────────┼────────────────┼──────────┼──────────────┼──────────────────────┼───────────────┼──────────────────────────────────────────────────────────────┤
│ git                         │ CVE-2023-25652 │ HIGH     │ affected     │ 1:2.39.2-1.1         │               │ git: by feeding specially crafted input to `git apply        │

~/k02$
```


### 3.5. OWASP K02 - Scan public images

You can scan public or private images which you got from somewhere. Let's scan the latest official `alpine` image:

```
~/k02$ trivy image docker.io/alpine:latest

2024-01-27T02:42:16.250Z	INFO	Vulnerability scanning is enabled
2024-01-27T02:42:16.251Z	INFO	Secret scanning is enabled
2024-01-27T02:42:16.251Z	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-01-27T02:42:16.251Z	INFO	Please see also https://aquasecurity.github.io/trivy/v0.48/docs/scanner/secret/#recommendation for faster secret detection
2024-01-27T02:42:17.091Z	INFO	Detected OS: alpine
2024-01-27T02:42:17.091Z	WARN	This OS version is not on the EOL list: alpine 3.19
2024-01-27T02:42:17.091Z	INFO	Detecting Alpine vulnerabilities...
2024-01-27T02:42:17.114Z	INFO	Number of language-specific files: 0

docker.io/alpine:latest (alpine 3.19.1)

Total: 0 (UNKNOWN: 0, LOW: 0, MEDIUM: 0, HIGH: 0, CRITICAL: 0)

~/k02$
```

Will you look at that, no vulnerabilities found!

However, the moment we move down a version (3.19 is "latest" as of publication) we can see vulnerabilities come to
light, try it:

```
~/k02$ trivy image docker.io/alpine:3.18.5

2024-01-27T02:43:56.237Z	INFO	Vulnerability scanning is enabled
2024-01-27T02:43:56.238Z	INFO	Secret scanning is enabled
2024-01-27T02:43:56.238Z	INFO	If your scanning is slow, please try '--scanners vuln' to disable secret scanning
2024-01-27T02:43:56.238Z	INFO	Please see also https://aquasecurity.github.io/trivy/v0.48/docs/scanner/secret/#recommendation for faster secret detection
2024-01-27T02:43:57.026Z	INFO	Detected OS: alpine
2024-01-27T02:43:57.026Z	INFO	Detecting Alpine vulnerabilities...
2024-01-27T02:43:57.028Z	INFO	Number of language-specific files: 0

docker.io/alpine:3.18.5 (alpine 3.18.5)

Total: 6 (UNKNOWN: 0, LOW: 2, MEDIUM: 4, HIGH: 0, CRITICAL: 0)

┌────────────┬───────────────┬──────────┬────────┬───────────────────┬───────────────┬───────────────────────────────────────────────────────────┐
│  Library   │ Vulnerability │ Severity │ Status │ Installed Version │ Fixed Version │                           Title                           │
├────────────┼───────────────┼──────────┼────────┼───────────────────┼───────────────┼───────────────────────────────────────────────────────────┤
│ libcrypto3 │ CVE-2023-6129 │ MEDIUM   │ fixed  │ 3.1.4-r1          │ 3.1.4-r3      │ openssl: POLY1305 MAC implementation corrupts vector      │
│            │               │          │        │                   │               │ registers on PowerPC                                      │
│            │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2023-6129                 │
│            ├───────────────┤          │        │                   ├───────────────┼───────────────────────────────────────────────────────────┤
│            │ CVE-2023-6237 │          │        │                   │ 3.1.4-r4      │ openssl: Excessive time spent checking invalid RSA public │
│            │               │          │        │                   │               │ keys                                                      │
│            │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2023-6237                 │
│            ├───────────────┼──────────┤        │                   ├───────────────┼───────────────────────────────────────────────────────────┤
│            │ CVE-2024-0727 │ LOW      │        │                   │ 3.1.4-r5      │ openssl: denial of service via null dereference           │
│            │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2024-0727                 │
├────────────┼───────────────┼──────────┤        │                   ├───────────────┼───────────────────────────────────────────────────────────┤
│ libssl3    │ CVE-2023-6129 │ MEDIUM   │        │                   │ 3.1.4-r3      │ openssl: POLY1305 MAC implementation corrupts vector      │
│            │               │          │        │                   │               │ registers on PowerPC                                      │
│            │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2023-6129                 │
│            ├───────────────┤          │        │                   ├───────────────┼───────────────────────────────────────────────────────────┤
│            │ CVE-2023-6237 │          │        │                   │ 3.1.4-r4      │ openssl: Excessive time spent checking invalid RSA public │
│            │               │          │        │                   │               │ keys                                                      │
│            │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2023-6237                 │
│            ├───────────────┼──────────┤        │                   ├───────────────┼───────────────────────────────────────────────────────────┤
│            │ CVE-2024-0727 │ LOW      │        │                   │ 3.1.4-r5      │ openssl: denial of service via null dereference           │
│            │               │          │        │                   │               │ https://avd.aquasec.com/nvd/cve-2024-0727                 │
└────────────┴───────────────┴──────────┴────────┴───────────────────┴───────────────┴───────────────────────────────────────────────────────────┘

~/k02$
```

Just one version below latest (`alpine:3.18.5`) we get 6 vulnerabilities!

You can use `trivy` to scan your own images, or images you plan on using as base images. You can also use `trivy` to
scan your Kubernetes cluster for vulnerable images and use Kubernetes Admission Control to block deployments if images
are not scanned.


### 4. OWASP K02 - Prevention

The "How to Prevent" section of the OWASP K02 risk states:

> Image Composition: Container images should be created using minimal OS packages and dependencies to reduce the attack
> surface if the workload should be compromised. Consider utilizing alternative base images such as Distroless or
> Scratch to not only improve security posture but also drastically reduce the noise generated by vulnerability
> scanners. Using distroless images also reduces the image size which ultimately helps in faster CI/CD build. It is also
> important to ensure your base images are up-to-date with the latest security patches. Tools such as Docker Slim are
> available to optimize your image footprint for performance and security reasons.

Using scanners we can disover out-of-date base images with vulnerabilities and update them whenever possible but
according to the OWASP prevention advice we can also mitigate attacks using "distroless" images.


### 4.1. OWASP K02 - Distroless images

"Distroless" images are a type of container image that Google created. They contain only your application and its runtime
dependencies. They do not contain package managers, shells, or any other programs you would expect to find in a standard
Linux distribution.

Why to use them and what are the benefits?

- Distroless images are minimal because they exclude unnecessary binaries and files, resulting in a smaller attack
  surface and less overhead

- By removing unnecessary components typically found in a Linux distribution, these images reduce the risk of security
  vulnerabilities; if a shell or package manager isn't present, they can't be exploited

- They contain only your application and its direct dependencies, which can make them simpler to understand and manage.

- The smaller size of the images can result in faster deploy times, less wasted CPU time, and other efficiencies.

However, the minimalist nature of Distroless images can also be a drawback, particularly when debugging. For example,
you won't have a shell to log into or package manager to install debugging tools. Therefore, you'll need to think
differently about how to debug applications that are running in `Distroless` containers.

Let's check our previews image `cncf-vuln:1.0.0` and see what is inside.

```

~/k02$ docker container run --rm -it --name myapp5 cncf-vuln:1.0.0 /bin/sh

# id

uid=0(root) gid=0(root) groups=0(root)

# ls -l /

total 52
drwxr-xr-x   1 root root 4096 Jul  8 23:43 app
lrwxrwxrwx   1 root root    7 Jul  3 00:00 bin -> usr/bin
drwxr-xr-x   2 root root 4096 Mar  2 13:55 boot
drwxr-xr-x   5 root root  360 Jul  9 00:06 dev
drwxr-xr-x   1 root root 4096 Jul  9 00:06 etc
drwxr-xr-x   2 root root 4096 Mar  2 13:55 home
lrwxrwxrwx   1 root root    7 Jul  3 00:00 lib -> usr/lib
lrwxrwxrwx   1 root root    9 Jul  3 00:00 lib32 -> usr/lib32
lrwxrwxrwx   1 root root    9 Jul  3 00:00 lib64 -> usr/lib64
lrwxrwxrwx   1 root root   10 Jul  3 00:00 libx32 -> usr/libx32
drwxr-xr-x   2 root root 4096 Jul  3 00:00 media
drwxr-xr-x   2 root root 4096 Jul  3 00:00 mnt
drwxr-xr-x   2 root root 4096 Jul  3 00:00 opt
dr-xr-xr-x 206 root root    0 Jul  9 00:06 proc
drwx------   1 root root 4096 Jul  4 06:17 root
drwxr-xr-x   1 root root 4096 Jul  4 03:28 run
lrwxrwxrwx   1 root root    8 Jul  3 00:00 sbin -> usr/sbin
drwxr-xr-x   2 root root 4096 Jul  3 00:00 srv
dr-xr-xr-x  13 root root    0 Jul  9 00:06 sys
drwxrwxrwt   1 root root 4096 Jul  4 06:19 tmp
drwxr-xr-x   1 root root 4096 Jul  3 00:00 usr
drwxr-xr-x   1 root root 4096 Jul  3 00:00 var

# ps

    PID TTY          TIME CMD
      1 pts/0    00:00:00 sh
     10 pts/0    00:00:00 ps

# exit

~/k02$
```

As you can see we have a filesystem including a shell where we can execute commands. Now let's build our Python
application as a "distroless" image.

Create our working directory and copy our Python app from the previews example:

```
~/k02$ mkdir ~/k02/distroless && cd $_

~/k02/distroless$ cp ~/k02/app.py .

~/k02/distroless$
```

We need to make some changes for our application to work.
- `FROM` - We need to change the base image to use a distroless image
- `CMD` - We need to change the command to run our application

In our previous image we used `CMD` as: `CMD ["python", "app.py"]`. The distroless image will evaluate this as two
separate arguments: `"python"` and `"app.py"`. In the distroless image the `ENTRYPOINT` is set to `python3`, so whatever
is in `CMD` will be passed as arguments to the Python interpreter so we need to remove `"python"` and leave only the
`"app.py"` argument.


```
~/k02/distroless$ nano Dockerfile && cat $_
```
```Dockerfile
FROM gcr.io/distroless/python3-debian11
WORKDIR /app
COPY app.py .
CMD ["app.py"]
```
```
~/k02/distroless$
```

Build the image:

```
~/k02/distroless$ docker image build -t cncf-distroless:1.0.0 .

[+] Building 6.6s (8/8) FINISHED                                                     docker:default

...

 => => naming to docker.io/library/cncf-distroless:1.0.0                                       0.0s


~/k02$
```

We can check if our image is built:

```
~/k02/distroless$ docker image ls | grep distro

cncf-distroless                  1.0.0     a8b52de842dd   12 minutes ago      54.5MB

~/k02$
```

Let's run it and see what is inside:

```
~/k02/distroless$ docker container run --rm -it --name distroless cncf-distroless:1.0.0 /bin/sh

SyntaxError: Non-UTF-8 code starting with '\xac' in file /bin/sh on line 2, but no encoding declared; see http://python.org/dev/peps/pep-0263/ for details

~/k02$
```

We are getting this error because distroless images do not include a shell (`/bin/sh` in our case). The idea behind
distroless is to minimize the attack surface of the image (and resultant containers) by not including any utilities or
shells that aren't needed for your application to run.

We attempt to run `/bin/sh` inside the container, there's no such file. Since the image is based on a minimal Python
image, it tries to execute `/bin/sh` as a Python script, hence the Python-related syntax error.

But don't worry if we run our image without `/bin/sh` it will work because the main focus of distroless images is to run
your application not a shell.

```
~/k02/distroless$ docker container run --rm --name distroless cncf-distroless:1.0.0

cncf old and vulnerable python image!

~/k02/distroless$
```

We should probably change that message now that the image less vulnerable!


### 5. CHALLENGE

Below is a faulty Kubernetes pod specification (`pod.yaml`) that contains several insecure and incorrect configurations.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: faulty-pod
spec:
  containers:
  - name: insecure-container
    image: alpine:latest
    command: ["/bin/sh", "-c", "sleep infinity"]
    securityContext:
      runAsUser: 0
      privileged: true
    volumeMounts:
    - name: host-volume
      mountPath: /data
  volumes:
  - name: host-volume
    hostPath:
      path: /var/run
      type: Directory
  hostPID: true
  imagePullPolicy: IfNotPresent
```

Your task is to identify the issues using kube-score and fix them. Try to fix as much as you can. We didn't learn about
`NetworkPolicy`, so you can ignore that but we include it in the solution just for your reference.


### 6. Clean up

Delete the pods you created during this lab:

```
~/k02/distroless$ kubectl delete po/root-user po/root-user-crictl po/safer-pod --now

pod "root-user" deleted
pod "root-user-crictl" deleted
pod "safer-pod" deleted

~/k02/distroless$
```

<br>

Congratulations you have completed the lab!

<br>

