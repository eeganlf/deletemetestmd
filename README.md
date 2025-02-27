# Install Kubernetes

# Overview 

There are several Kubernetes installation tools provided by various
vendors. In this lab we will learn to use **kubeadm**. As a
community-supported independent tool, it is planned to become the
primary manner to build a Kubernetes cluster.

::: info
The labs were written using **Ubuntu 24.04** instances running on
**G**oogle **C**loud **P**latform (**GCP**). They have been written to
be vendor-agnostic so could run on AWS, local hardware, or inside of
virtualization to give you the most flexibility and options. Each
platform will have different access methods and considerations. As of
v1.21.0 the minimum (as in barely works) size for **VirtualBox** is
3vCPU/4G memory/5G minimal OS for `cp` and 1vCPU/2G memory/5G minimal OS
for worker node. Most other providers work with 2CPU/7.5G.
:::

If using your own equipment you will have to disable swap on every node,
and ensure there is only one network interface. Multiple interfaces are
supported but require extra configuration. There may be other
requirements which will be shown as warnings or errors when using the
**kubeadm** command. While most commands are run as a regular user,
there are some which require root privilege. You If you are accessing
the nodes remotely, such as with **GCP** or **AWS**, you will need to
use an SSH client such as a local terminal or **PuTTY** if not using
**Linux** or a Mac. You can download **PuTTY** from
[www.putty.org](www.putty.org){.uri}. You would also require a or file
to access the nodes. Each cloud provider will have a process to download
or create this file. If attending in-person instructor led training the
file will be made available during class.

::: important
Please disable any firewalls while learning Kubernetes. While there is a
list of required ports for communication between components, the list
may not be as complete as necessary. If using **GCP** you can add a rule
to the project which `allows all traffic to all ports`. Should you be
using **VirtualBox** be aware that inter-VM networking will need to be
set to `promiscuous mode`.
:::

In the following exercise we will install Kubernetes on a single node
then grow the cluster, adding more compute resources. Both nodes used
are the same size, providing 2 vCPUs and 7.5G of memory. Smaller nodes
could be used, but would run slower, and may have strange errors.

::: info
Various exercises will use YAML files, which are included in the text.
You are encouraged to write some of the files as time permits, as the
syntax of YAML has white space indentation requirements that are
important to learn. An important note, **do not** use tabs in your YAML
files, **white space only. Indentation matters.**
:::

If using a PDF the use of copy and paste often does not paste the single
quote correctly. It pastes as a back-quote instead. You will need to
modify it by hand. The files mentioned in labs have also been made
available as a compressed **tar** file. You can view the resources by
navigating to this URL:

[https://cm.lf.training/\\course](https://cm.lf.training/\course){.uri}

To login use user: `LFtraining` and a password of: `Penguin2014`

Once you find the name and link of the current file, which will change
as the course updates, use **wget** to download the file into your node
from the command line then expand it like this:

::: cmdtt
$wget https://cm.lf.training/\course/\course\_V\version\_SOLUTIONS.tar.xz \textbackslash
              --user=LFtraining --password=Penguin2014$ tar -xvf
\_V_SOLUTIONS.tar.xz
:::

(**Note**: depending on your PDF viewer, if you are cutting and pasting
the above instructions, the underscores may disappear and be replaced by
spaces, so you may have to edit the command line by hand!)

# Install Kubernetes {#install-kubernetes .unnumbered}

::: lfbox
Log into your control plane (cp) and worker nodes. If attending
in-person instructor led training the node IP addresses will be provided
by the instructor. You will need to use a or key for access, depending
on if you are using **ssh** from a terminal or **PuTTY**. The instructor
will provide this to you.
:::

1.  Open a terminal session on your first node. For example, connect via
    **PuTTY** or **SSH** session to the first **GCP** node. The user
    name may be different than the one shown, `student`. Create a
    non-root user if one is not present. The IP used in the example will
    be different than the one you will use. You may need to adjust the
    access mode of your pem or ppk key. The example shows how a Mac or
    Linux system would change mode. Windows may have a similar process.

    ::: cmdtt
    \$ chmod 400 .pem \[student@laptop  \]\$ ssh -i .pem
    student@35.226.100.87
    :::

    ::: out
    The authenticity of host '54.214.214.156 (35.226.100.87)' can't be
    established. ECDSA key fingerprint is
    SHA256:IPvznbkx93/Wc+ACwXrCcDDgvBwmvEXC9vmYhk2Wo1E. ECDSA key
    fingerprint is MD5:d8:c9:4b:b0:b0:82:d3:95:08:08:4a:74:1b:f6:e1:9f.
    Are you sure you want to continue connecting (yes/no)? yes Warning:
    Permanently added '35.226.100.87' (ECDSA) to the list of known
    hosts. \<output_omitted\>
    :::

2.  Use the **wget** command above to download and extract the course
    tarball to your node. Again copy and paste won't always paste the
    underscore characters.

3.  You are encouraged to type out commands, if using a PDF or
    eLearning, instead of copy and paste. By typing the commands you
    have a better chance to remember both the command and the concept.
    There are a few exceptions, such as when a long hash or output is
    much easier to copy over, and does not offer a learning opportunity.

4.  Become `root` and update and upgrade the system. You may be asked a
    few questions. If so, allow restarts and keep the local version
    currently installed. Which would be a `yes` then a `2`.

    ::: cmd
    student@cp: $sudo -i

    root@cp:~# apt-get update && apt-get upgrade -y
             \end{cmd}
             
             \begin{out}[]
    <output_omitted>

    You can choose this option to avoid being prompted; instead,
    all necessary restarts will be done for you automatically
    so you can avoid being asked questions on each library upgrade.
         \end{out}

             \begin{cmd}
    Restart services during package upgrades without asking? [yes/no] yes
         \end{cmd}
             
             \begin{out}[]
    <output_omitted>

    A new version (/tmp/fileEbke6q) of configuration file /etc/ssh/sshd_config is
    available, but the version installed currently has been locally modified.

      1. install the package maintainer's version
      2. keep the local version currently installed
      3. show the differences between the versions
      4. show a side-by-side difference between the versions
      5. show a 3-way difference between available versions
      6. do a 3-way merge between available versions
      7. start a new shell to examine the situation
              \end{out}

             \begin{cmd}
    What do you want to do about modified configuration file sshd_config? 2
              \end{cmd}
             
             \begin{out}[]
    <output_omitted>
          \end{out}

             \item
             Install a text editor like \textbf{nano} (an easy to use editor),
             \textbf{vim}, or \textbf{emacs}. Any will do,
             the labs use a popular option, \textbf{vim}.
             \begin{cmd}
    root@cp:~# apt-get install -y vim
                  \end{cmd}
             
             \begin{out}[]
    <output-omitted>
                  \end{out}

             \item
             The main choices for a container environment are
             \textbf{containerd}, \textbf{cri-o}, and \textbf{Docker} 
             on older clusters.
             We suggest \textbf{containerd} for class, as it is easy
             to deploy and commonly used by cloud providers. 

             Please note, install one engine only. If more than one
             are installed the \textbf{kubeadm} init process search
             pattern will use Docker at the moment. Also be aware that
             engines other than \textbf{containerd} may show different
             output on some commands.


             \item
             There are several packages we should install to ensure we have
             all dependencies take care of. Please note the backslash is
             not necessary and can be removed if typing on a single line.
             \begin{cmd}
    root@cp:~# apt install curl apt-transport-https git wget  \
    software-properties-common lsb-release ca-certificates socat -y 
                  \end{cmd}
             
             \begin{out}[]
    <output-omitted>
                  \end{out}
              
             \item
             
             Disable swap if not already done. Cloud providers disable swap 
             on their images.
             \begin{cmd}
    root@cp:~#  swapoff -a
                  \end{cmd}

             \item
             Load modules to ensure they are available for following steps.
             \begin{cmd}
    root@cp:~# modprobe overlay
    root@cp:~# modprobe br_netfilter
                  \end{cmd}
             

             \item
             Update kernel networking to allow necessary traffic. Be aware the 
             shell will add a greater than sign (\textgreater) to indicate the command continues after
             a carriage return. 
             \begin{cmd}
    root@cp:~# cat << EOF | tee /etc/sysctl.d/kubernetes.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
    EOF
                  \end{cmd}
             
             \begin{out}[]
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1
    net.ipv4.ip_forward = 1
                  \end{out}     

             \item
             
             Ensure the changes are used by the current kernel as well
             \begin{cmd}
    root@cp:~# sysctl --system
                  \end{cmd}
             
             \begin{out}[]
    * Applying /etc/sysctl.d/10-console-messages.conf ...
    kernel.printk = 4 4 1 7
    * Applying /etc/sysctl.d/10-ipv6-privacy.conf ...
    net.ipv6.conf.all.use_tempaddr = 2
    net.ipv6.conf.default.use_tempaddr = 2
    * Applying /etc/sysctl.d/10-kernel-hardening.conf ...
    kernel.kptr_restrict = 1
    <output_omitted>
                  \end{out}          

             \item
             Install the necessary key for the software to install
             \begin{cmd}
    root@cp:~# mkdir -p /etc/apt/keyrings
    root@cp:~# curl -fsSL https://download.docker.com/linux/ubuntu/gpg \
    | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

    root@cp:~# echo \
      "deb [arch=$(dpkg --print-architecture)
    signed-by=/etc/apt/keyrings/docker.gpg\]
     https://download.docker.com/linux/ubuntu
     $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
                  \end{cmd}     


             \item
             Install the containerd software.

             \begin{cmd}
    root@cp:~# apt-get update &&  apt-get install containerd.io -y
    root@cp:~# containerd config default | tee /etc/containerd/config.toml
    root@cp:~# sed -e 's/SystemdCgroup = false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
    root@cp:~# systemctl restart containerd
                  \end{cmd}
             \begin{out}[]
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following packages were automatically installed and are no longer required:
    <output_omitted>
                  \end{out}



             \item

             Download the public signing key for the Kubernetes package repositories

             \begin{cmd}
    root@cp:~# mkdir -p -m 755 /etc/apt/keyrings
    root@cp:~# curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
           | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
          \end{cmd}
             

             \item

             Add the appropriate Kubernetes apt repository. Please note that this repository 
         have packages only for Kubernetes 1.30; for other Kubernetes minor versions, 
         you need to change the Kubernetes minor version in the URL to match your 
         desired minor version


             \begin{cmd}
    root@cp:~# echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] \
               https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /" \
           | sudo tee /etc/apt/sources.list.d/kubernetes.list
             \end{cmd}
             
             \begin{out}[]
    deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ / 
             \end{out}

             \item
             Update with the new repo declared, which will download updated repo
             information.

             \begin{cmd}
    root@cp:~# apt-get update
             \end{cmd}
             
             \begin{out}[]
    <output-omitted>
             \end{out}


             \item
             Install the Kubernetes software. There are regular releases, the
             newest of which can be used by omitting the equal sign and
             version information on the command line. Historically
             new versions have lots of changes and a good chance of
             a bug or five. As a result we will hold the software at
             the recent but stable version we install. In a later lab
             we will update the cluster to a newer version.
             \begin{cmd}
    root@cp:~# apt-get install -y kubeadm=1.30.1-1.1 kubelet=1.30.1-1.1 kubectl=1.30.1-1.1 
             \end{cmd}
             
             \begin{out}[]
    <output-omitted>
             \end{out}
             \begin{cmd}
    root@cp:~# apt-mark hold kubelet kubeadm kubectl
             \end{cmd}
             
             \begin{out}[]
    kubelet set on hold.
    kubeadm set on hold.
    kubectl set on hold.
             \end{out}
    \item

             Find the IP address of the primary interface of the cp server. 
             The example below would be the \verb:ens4: interface and an IP
             of \verb:10.128.0.3:, yours may be different. There are two ways of looking at your IP addresses.


             \begin{cmd}
    root@cp:~# hostname -i
                     \end{cmd}
             
             \begin{out}[]
    10.128.0.3
             \end{out}

             \begin{cmd}
    root@cp:~# ip addr show
             \end{cmd}
             
             \begin{out}[]
    ....
    2: ens4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1460 qdisc mq state UP group default qlen 1000
        link/ether 42:01:0a:80:00:18 brd ff:ff:ff:ff:ff:ff
        inet 10.128.0.3/32 brd 10.128.0.3 scope global ens4
           valid_lft forever preferred_lft forever
        inet6 fe80::4001:aff:fe80:18/64 scope link
           valid_lft forever preferred_lft forever
    ....
             \end{out}


             \item
             Add an local DNS alias for our cp server.
             Edit the \file{/etc/hosts} file and add the
             above IP address and assign a name
             \verb:k8scp:.
             \begin{cmd}
    root@cp:~# vim /etc/hosts
             \end{cmd}
             \begin{kcode}
    10.128.0.3 k8scp    #<-- Add this line
    127.0.0.1 localhost
    ....
             \end{kcode}

             \item

             Create a configuration file for the cluster. There
             are many options we could include, and they differ
             for \textbf{containerd}, \textbf{Docker}, and \textbf{cri-o}.
             Use the file included in the course tarball. After
             our cluster is initialized we will view other default
             values used. Be sure to use the node alias we added to
             \file{/etc/hosts}, not the IP so the network certificates
             will continue to work when we deploy a load balancer
             in a future lab. The file is also in the course tarball.
         \begin{cmdtt}
    root@cp:~# cp /home/student/\course/SOLUTIONS/s_03/kubeadm-config.yaml /root/
       \end{cmdtt}
                \begin{cmd}
    root@cp:~# vim kubeadm-config.yaml
                     \end{cmd}
                \begin{yamllst}[\texttt{kubeadm-config.yaml}]
    apiVersion: kubeadm.k8s.io/v1beta3
    kind: ClusterConfiguration
    kubernetesVersion: 1.30.1               #<-- Use the word stable for newest version
    controlPlaneEndpoint: "k8scp:6443"      #<-- Use the alias we put in /etc/hosts not the IP
    networking:
      podSubnet: 192.168.0.0/16
                     \end{yamllst}



             \item

             Initialize the cp. Scan through the output.
             Expect the output to change as the software matures.
             At the end are configuration directions to run as a
             non-root user. The token is mentioned as well. This
             information can be found later with the
             \textbf{kubeadm token list} command. The output also
             directs you to create a pod network to the cluster,
             which will be our next step. Pass the network settings
             \textbf{Cilium} has in its configuration file. 
             \textbf{Please note:} the output
             lists several commands which following exercise steps will
             complete.

             \begin{cmd}
    root@cp:~# kubeadm init --config=kubeadm-config.yaml --upload-certs --node-name=cp \
          | tee kubeadm-init.out                 #<-- Save output for future review
          \end{cmd}
             
             \begin{out}[]
    [init] Using Kubernetes version: v1.30.1
    [preflight] Running pre-flight checks



    <output_omitted>

    You can now join any number of the control-plane node
    running the following command on each as root:

    kubeadm join k8scp:6443 --token vapzqi.et2p9zbkzk29wwth \
     --discovery-token-ca-cert-hash sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd \
     --control-plane --certificate-key 911d41fcada89a18210489afaa036cd8e192b1f122ebb1b79cce1818f642fab8

    Please note that the certificate-key gives access to cluster sensitive
    data, keep it secret!
    As a safeguard, uploaded-certs will be deleted in two hours; If
    necessary, you can use
    "kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

    Then you can join any number of worker nodes by running the following
    on each as root:

    kubeadm join k8scp:6443 --token vapzqi.et2p9zbkzk29wwth \
     --discovery-token-ca-cert-hash sha256:f62bf97d4fba6876e4c3ff645df3fca969c06169dee3865aab9d0bca8ec9f8cd
          \end{out}

             \item

             As suggested in the directions at the end of the previous output
             we will allow a \texttt{non-root} user admin level access to the
             cluster. Take a quick look at the configuration file once it has
             been copied and the permissions fixed.

             \begin{cmd}
    root@cp:~# exit
          \end{cmd}
             
             \begin{out}[]
    logout
          \end{out}
             \begin{cmd}
    student@cp:~$ mkdir -p $HOME/.kube

    student@cp:~$ sudo cp -i /etc/kubernetes/admin.conf
    $HOME/.kube/config

    student@cp:~$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

    student@cp:~$ less .kube/config
    :::

    ::: out
    apiVersion: v1 clusters: - cluster: \<output_omitted\>
    :::

5.  Deciding which pod network to use for Container Networking Interface
    (**CNI**) should take into account the expected demands on the
    cluster. There can be only one pod network per cluster, although the
    **CNI-Genie** project is trying to change this.

    The network must allow container-to-container, pod-to-pod,
    pod-to-service, and external-to-service communications. We will use
    **Cilium** as a network plugin which will allow us to use
    `Network Policies` later in the course. Currently **Cilium** does
    not deploy using CNI by default.

    ::: lfbox
    Cilium is genereally installed using \"cilium install\" or using
    \"helm install\" commands. We have generated the cilium-cni.yaml
    file using the below commands for your convenience. **Note**: You
    dont need to execute the commands in this box, they are just for
    reference.
    :::

    ::: out
    serviceaccount/cilium created serviceaccount/cilium-operator created
    secret/cilium-ca created configmap/cilium-config created
    \<output_omitted\>
    :::

6.  While many objects have short names, a **kubectl** command can be a
    lot to type. We will enable **bash** auto-completion. Begin by
    adding the settings to the current shell. Then update the file to
    make it persistent. Ensure the `bash-completion` package is
    installed. If it was not installed, log out then back in for the
    shell completion to work.

    ::: cmd
    student@cp: $sudo apt-get install bash-completion -y

    <exit and log back in>

    student@cp:~$ source \<(kubectl completion bash)

    student@cp: $echo "source <(kubectl completion bash)" >>$HOME/.bashrc
    :::

7.  Test by describing the node again. Type the first three letters of
    the sub-command then type the **Tab** key. Auto-completion assumes
    the `default` namespace. Pass the namespace first to use
    auto-completion with a different namespace. By pressing **Tab**
    multiple times you will see a list of possible values. Continue
    typing until a unique name is used. First look at the current node
    (your node name may not start with `cp`), then look at pods in the
    `kube-system` namespace. If you see an error instead such as
    `-bash: _get_comp_words_by_ref: command not found` revisit the
    previous step, install the software, log out and back in.

    ::: cmd
    student@cp: $kubectl des<Tab> n<Tab><Tab> cp<Tab>

    student@cp:~$ kubectl -n kube-s\<Tab\> g\<Tab\> po\<Tab\>
    :::

8.  Explore the **kubectl help** command. The output has been omitted
    from commands. Take a moment to review help topics.

    ::: cmd
    student@cp: $kubectl help
    student@cp:~$ kubectl help create
    :::

9.  View other values we could have included in the file when creating
    the cluster.

    ::: cmd
    student@cp: $sudo kubeadm config print init-defaults
              \end{cmd}
             
             \begin{out}[]
    apiVersion: kubeadm.k8s.io/v1beta3
    bootstrapTokens:
    - groups:
      - system:bootstrappers:kubeadm:default-node-token
      token: abcdef.0123456789abcdef
      ttl: 24h0m0s
      usages:
      - signing
      - authentication
    kind: InitConfiguration
    <output_omitted>

              \end{out}


          \end{enumerate}

       \end{exe}

       \begin{exe} {Grow the Cluster}
          \begin{lfbox}
             Open another terminal and connect into a your second
             node. Install \textbf{containerd} and Kubernetes
             software. These are the many, but not all, of the steps
             we did on the cp node.

             This book will use the \textbf{worker} prompt
             for the node being added to help keep track of the
             proper node for each command. Note that the prompt
             indicates both the user and system upon which
             run the command. It can be helpful to change the
             colors and fonts of your terminal session to keep
             track of the correct node.
          \end{lfbox}
          \begin{enumerate}


             \item
             Using the same process as before connect to a
             second node. If attending an instructor-led class
             session, use the same \file{.pem} key and a new IP
             provided by the instructor to access the new
             node. Giving a different title or color to the new terminal
             window is probably a good idea to keep track of the
             two systems. The prompts can look very similar.

                \item
                \begin{cmd}
    student@worker:~$ sudo -i
    :::

10. ::: cmd
    root@worker: # apt-get update && apt-get upgrade -y
    :::

    ::: out
    \<If asked allow services to restart and keep the local version of
    software\>
    :::

11. Install the containerd engine, starting with dependent software.

    ::: cmd
    root@worker: # apt install curl apt-transport-https vim git wget
     software-properties-common lsb-release ca-certificates socat -y

    root@worker: # swapoff -a

    root@worker:˜# modprobe overlay

    root@worker:˜# modprobe br_netfilter

    root@worker:˜# cat \<\< EOF \| tee /etc/sysctl.d/kubernetes.conf
    net.bridge.bridge-nf-call-ip6tables = 1
    net.bridge.bridge-nf-call-iptables = 1 net.ipv4.ip_forward = 1 EOF

    root@worker:˜# sysctl --system

    root@worker: # mkdir -p /etc/apt/keyrings root@worker: # curl -fsSL
    https://download.docker.com/linux/ubuntu/gpg  \| sudo gpg --dearmor
    -o /etc/apt/keyrings/docker.gpg

    root@worker: # echo  \"deb
    \[arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
      https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) stable\" \| sudo tee
    /etc/apt/sources.list.d/docker.list \> /dev/null

    root@worker: # apt-get update && apt-get install containerd.io -y
    root@worker: # containerd config default \| tee
    /etc/containerd/config.toml root@worker: # sed -e 's/SystemdCgroup =
    false/SystemdCgroup = true/g' -i /etc/containerd/config.toml
    root@worker: # systemctl restart containerd
    :::

12. Get the GPG key for the software

    ::: cmd
    root@worker: # curl -fsSL
    https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key  \| sudo gpg
    --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
    :::

13. Add Kubernetes repo

    ::: cmd
    root@worker: # echo \"deb
    \[signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg\]
     https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /\"  \| sudo tee
    /etc/apt/sources.list.d/kubernetes.list
    :::

14. Update repos then install the Kubernetes software. Be sure to match
    the version on the cp.

    ::: cmd
    root@worker: # apt-get update
    :::

15. ::: cmd
    root@worker: # apt-get install -y kubeadm=1.30.1-1.1
    kubelet=1.30.1-1.1 kubectl=1.30.1-1.1
    :::

16. Ensure the version remains if the system is updated.

    ::: cmd
    root@worker: # apt-mark hold kubeadm kubelet kubectl
    :::

17. Find the IP address of your **cp** server. The interface name will
    be different depending on where the node is running. Currently
    inside of **GCE** the primary interface for this node type is
    `ens4`. Your interfaces names may be different. From the output we
    know our cp node IP is `10.128.0.3`.

    ::: cmd
    student@cp: $hostname -i
                     \end{cmd}
             
             \begin{out}[]
    10.128.0.3
             \end{out}

             \begin{cmd}
    student@cp:~$ ip addr show ens4 \| grep inet
    :::

    ::: out
    inet 10.128.0.3/32 brd 10.128.0.3 scope global ens4 inet6
    fe80::4001:aff:fe8e:2/64 scope link
    :::

18. At this point we could copy and paste the **join** command from the
    cp node. That command only works for 2 hours, so we will build our
    own **join** should we want to add nodes in the future. Find the
    token on the cp node. The token lasts 2 hours by default. If it has
    been longer, and no token is present you can generate a new one with
    the **sudo kubeadm token create** command, seen in the following
    command.

    ::: cmd
    student@cp: $sudo kubeadm token create --print-join-command
          \end{cmd}
             
             \begin{out}[]
    kubeadm join k8scp:6443 --token kcu55w.7jso85i0e2dsn05y \
     --discovery-token-ca-cert-hash sha256:0fb62b3c47bfd3af3c15d21f2ab6082fad1f913b244d5980816f8147ce9936ef
          \end{out}

             \item
             On the \textbf{worker node} add a local DNS alias for 
             the cp server. Edit the \file{/etc/hosts} file and add the
             cp IP address and assign the name \verb:k8scp:. The entry
             should be exactly the same as the edit on the cp.
             \begin{cmd}
    root@worker:~# vim /etc/hosts
                     \end{cmd}
             \begin{kcode}
    10.128.0.3 k8scp    #<-- Add this line
    127.0.0.1 localhost
    ....
                     \end{kcode}

             \item
             Use the token and hash, in this case as \verb^sha256:long-hash^
             to join the cluster from the \textbf{second/worker} node. Use the
             \textbf{private} IP address of the cp server and port
             6443. The output of the \textbf{kubeadm init} on the cp
             also has an example to use, should it still be available.
             \begin{cmd}
    root@worker:~# kubeadm join \
         k8scp:6443 --token wcal99.hxv9v0gtnz42g6dr \
         --discovery-token-ca-cert-hash \ 
         sha256:0fb62b3c47bfd3af3c15d21f2ab6082fad1f913b244d5980816f8147ce9936ef --node-name=worker
          \end{cmd}
             
             \begin{out}[]
    [preflight] Running pre-flight checks


    [preflight] Reading configuration from the cluster...
    [preflight] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
    [kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
    [kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
    [kubelet-start] Activating the kubelet service
    [kubelet-start] Waiting for the kubelet to perform the TLS Bootstrap...

    This node has joined the cluster:
    * Certificate signing request was sent to apiserver and a response was received.
    * The Kubelet was informed of the new secure connection details.

    Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
          \end{out}


             \item
             Try to run the \textbf{kubectl} command on the
             secondary system. It should fail. You do not have
             the cluster or authentication keys in your local
             \file{.kube/config} file.
             \begin{cmd}
    root@worker:~# exit

    student@worker:~$ kubectl get nodes
    :::

    ::: out
    The connection to the server localhost:8080 was refused - did you
    specify the right host or port?
    :::

    ::: cmd
    student@worker: $ls -l .kube
         \end{cmd}
             
             \begin{out}[]
    ls: cannot access '.kube': No such file or directory
          \end{out}

          \end{enumerate}

       \end{exe}

       \begin{exe} {Finish Cluster Setup}

          \begin{enumerate}

             \item
             View the available nodes of the cluster. It can take a
             minute or two for the status to change from
             \texttt{NotReady} to \texttt{Ready}. The \verb:NAME:
             field can be used to look at the details. Your node
             name may be different, use YOUR \verb:control-plane: name
             in future commands, if different than the book.
             \begin{cmd}
    student@cp:~$ kubectl get node
    :::

    ::: out
    NAME STATUS ROLES AGE VERSION cp Ready control-plane 28m v1.30.1
    worker Ready \<none\> 50s v1.30.1
    :::

19. Look at the details of the node. Work line by line to view the
    resources and their current status. Notice the status of `Taints`.
    The cp won't allow non-infrastructure pods by default for security
    and resource contention reasons. Take a moment to read each line of
    output, some appear to be an error until you notice the status shows
    `False`.

    ::: cmd
    student@cp: $kubectl describe node cp
          \end{cmd}
             
             \begin{out}[]
    Name:               cp
    Roles:              control-plane
    Labels:             beta.kubernetes.io/arch=amd64
                        beta.kubernetes.io/os=linux
                        kubernetes.io/arch=amd64
                        kubernetes.io/hostname=cp
                        kubernetes.io/os=linux
                        node-role.kubernetes.io/control-plane=
                        node.kubernetes.io/exclude-from-external-load-balancers=
    Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: unix:///var/run/containerd/containerd.sock
                        node.alpha.kubernetes.io/ttl: 0
                        volumes.kubernetes.io/controller-managed-attach-detach: true
    CreationTimestamp:  Mon, 19 Sep 2024 15:00:23 +0000
    Taints:             node-role.kubernetes.io/control-plane:NoSchedule
    <output_omitted>
          \end{out}


             \item
             Allow the cp server to run non-infrastructure pods. The
             cp node begins tainted for security and performance reasons.
             We will allow usage of the node in the training environment, but
             this step may be skipped in a production environment. Note the
             \textbf{minus sign (-)} at the end, which is the syntax to
             remove a taint. As the second node does not have the taint you
             will get a \verb:not found: error. There may be more than
             one taint. Keep checking and removing them until all are
             removed.
             \begin{cmd}
    student@cp:~$ kubectl describe node \| grep -i taint
    :::

    ::: out
    Taints: node-role.kubernetes.io/control-plane:NoSchedule Taints:
    \<none\>
    :::

    ::: cmd
    student@cp: $kubectl taint nodes --all node-role.kubernetes.io/control-plane-
          \end{cmd}
             
             \begin{out}[]
    node/cp untainted
    error: taint "node-role.kubernetes.io/control-plane" not found
          \end{out}

             \begin{cmd}
    student@cp:~$ kubectl describe node \| grep -i taint
    :::

    ::: out
    Taints: \<none\> Taints: \<none\>
    :::

20. Determine if the DNS and Cilium pods are ready for use. They should
    all show a status of `Running`. It may take a minute or two to
    transition from `Pending`.

    ::: cmd
    student@cp: $kubectl get pods --all-namespaces
          \end{cmd}
             
    \begin{out}[]
    NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
    kube-system   cilium-operator-788c7d7585-tnsph   1/1     Running   0          95m
    kube-system   cilium-swjsj                       1/1     Running   0          95m
    kube-system   coredns-5d78c9869d-dwds8           1/1     Running   0          100m
    kube-system   coredns-5d78c9869d-t24p5           1/1     Running   0          100m

    <output_omitted>
          \end{out}

             \item
             \textbf{Only if} you notice the \texttt{coredns-} pods
             are stuck in \verb:ContainerCreating: status you may have
             to delete them, causing new ones to be generated. Delete
             both pods and check to see they show a \verb:Running:
             state. Your pod names will be different.

             \begin{cmd}
    student@cp:~$ kubectl get pods --all-namespaces
    :::

    ::: out
    NAMESPACE NAME READY STATUS RESTARTS AGE kube-system cilium-swjsj
    2/2 Running 0 12m kube-system coredns-576cbf47c7-rn6v4 0/1
    ContainerCreating 0 3s kube-system coredns-576cbf47c7-vq5dz 0/1
    ContainerCreating 0 94m \<output_omitted\>
    :::

    ::: cmd
    student@cp: $kubectl -n kube-system delete \
        pod coredns-576cbf47c7-vq5dz coredns-576cbf47c7-rn6v4
            \end{cmd}
             
             \begin{out}[]
    pod "coredns-576cbf47c7-vq5dz" deleted
    pod "coredns-576cbf47c7-rn6v4" deleted
          \end{out}

             \item
             When it finished you should see more interfaces will be created 
             . It may take up to a minute
             to be created. You will notice interfaces 
             such as \verb:cilium: interfaces when you deploy
             pods, as shown in the output below.
             \begin{cmd}
    student@cp:~$ ip a
    :::

    ::: out
    \<output_omitted\> 3: cilium_net@cilium_host:
    \<BROADCAST,MULTICAST,NOARP,UP,LOWER_UP\> mtu 1460 qdisc noqueue
    state UP group default qlen 1000 link/ether be:19:22:da:62:ac brd
    ff:ff:ff:ff:ff:ff inet6 fe80::bc19:22ff:feda:62ac/64 scope link
    valid_lft forever preferred_lft forever 5: cilium_vxlan:
    \<BROADCAST,MULTICAST,UP,LOWER_UP\> mtu 1460 qdisc noqueue state
    UNKNOWN group default qlen 1000 link/ether ca:22:e7:23:42:89 brd
    ff:ff:ff:ff:ff:ff inet6 fe80::c822:e7ff:fe23:4289/64 scope link
    valid_lft forever preferred_lft forever \<output_omitted\>
    :::

21. Containerd may still be using an out of date notation for the
    `runtime-endpoint`. You may see errors about an undeclared resource
    type such as `unix`//:. We will update the **crictl** configuration.
    There are many possible configuration options. We will set one, and
    view the configuration file that is created. We will also set this
    configuration on worker node as well for our convenience.

    ::: cmd
    student@cp: $sudo crictl config --set \
    runtime-endpoint=unix:///run/containerd/containerd.sock \
    --set image-endpoint=unix:///run/containerd/containerd.sock

    student@worker:~$ sudo crictl config --set
     runtime-endpoint=unix:///run/containerd/containerd.sock  --set
    image-endpoint=unix:///run/containerd/containerd.sock

    student@cp: $sudo cat /etc/crictl.yaml
           \end{cmd}
           \begin{out}[]
    runtime-endpoint: "unix:///run/containerd/containerd.sock"
    image-endpoint: "unix:///run/containerd/containerd.sock"
    timeout: 0
    debug: false
    pull-image-on-create: false
    disable-pull-on-run: false
            \end{out}

          \end{enumerate}

       \end{exe}


       \begin{exe} {Deploy A Simple Application}

          We will test to see if we can deploy a simple
          application, in this case the \textbf{nginx} web
          server.

          \begin{enumerate}

             \item
             Create a new \verb:deployment:, which is a Kubernetes
             object, which will deploy an application in a
             container. Verify it is running and the desired number
             of containers matches the available.
             \begin{cmd}
    student@cp:~$ kubectl create deployment nginx --image=nginx
    :::

    ::: out
    deployment.apps/nginx created
    :::

    ::: cmd
    student@cp: $kubectl get deployments
          \end{cmd}
             
             \begin{out}[]
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           8s
          \end{out}


             \item
             View the details of the deployment. Remember
             auto-completion will work for sub-commands and
             resources as well.
             \begin{cmd}
    student@cp:~$ kubectl describe deployment nginx
    :::

    ::: out
    Name: nginx Namespace: default CreationTimestamp: Wed, 23 Sep 2024
    22:38:32 +0000 Labels: app=nginx Annotations:
    deployment.kubernetes.io/revision: 1 Selector: app=nginx Replicas: 1
    desired \| 1 updated \| 1 total \| 1 ava\.... StrategyType:
    RollingUpdate MinReadySeconds: 0 RollingUpdateStrategy: 25
    \<output_omitted\>
    :::

22. View the basic steps the cluster took in order to pull and deploy
    the new application. You should see several lines of output. The
    first column shows the age of each message, note that due to JSON
    lack of order the time `LAST SEEN` time does not print out
    chronologically. Eventually older messages will be removed.

    ::: cmd
    student@cp: $kubectl get events
          \end{cmd}
             
             \begin{out}[]
    <output_omitted>
          \end{out}


             \item
             You can also view the output in \textbf{yaml} format, which
             could be used to create this deployment again or new
             deployments. Get the information but change the output
             to yaml. Note that halfway down there is status information
             of the current deployment.
             \begin{cmd}
    student@cp:~$ kubectl get deployment nginx -o yaml
    :::

    ::: yamllst
    apiVersion: apps/v1 kind: Deployment metadata: annotations:
    deployment.kubernetes.io/revision: \"1\" creationTimestamp:
    2024-09-24T18:21:25Z \<output_omitted\>
    :::

23. Run the command again and redirect the output to a file. Then edit
    the file. Remove the `creationTimestamp`, `resourceVersion`, and
    `uid` lines. Also remove all the lines including and after
    `status:`, which should be somewhere around line 120, if others have
    already been removed.

    ::: cmd
    student@cp: $kubectl get deployment nginx -o yaml > first.yaml

    student@cp:~$ vim first.yaml
    :::

    ::: out
    \<Remove the lines mentioned above\>
    :::

24. Delete the existing deployment.

    ::: cmd
    student@cp: $kubectl delete deployment nginx
          \end{cmd}
             
             \begin{out}[]
    deployment.apps "nginx" deleted
          \end{out}


             \item
             Create the deployment again this time using the file.
             \begin{cmd}
    student@cp:~$ kubectl create -f first.yaml
    :::

    ::: out
    deployment.apps/nginx created
    :::

25. Look at the yaml output of this iteration and compare it against the
    first. The creation time stamp, resource version and unique ID we
    had deleted are in the new file. These are generated for each
    resource we create, so we may need to delete them from yaml files to
    avoid conflicts or false information. You may notice some time stamp
    differences as well. The `status` should not be hard-coded either.

    ::: cmd
    student@cp: $kubectl get deployment nginx -o yaml > second.yaml

    student@cp:~$ diff first.yaml second.yaml
    :::

    ::: out
    \<output_omitted\>
    :::

26. Now that we have worked with the raw output we will explore two
    other ways of generating useful YAML or JSON. Use the `--dry-run`
    option and verify no object was created. Only the prior `nginx`
    deployment should be found. The output lacks the unique information
    we removed before, but does have the same essential values.

    ::: cmd
    student@cp: $kubectl create deployment two --image=nginx --dry-run=client -o yaml
          \end{cmd}
             \begin{yamllst}
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        app: two
      name: two
    spec:
    <output_omitted>
        \end{yamllst}

             \begin{cmd}
    student@cp:~$ kubectl get deployment
    :::

    ::: out
    NAME READY UP-TO-DATE AVAILABLE AGE nginx 1/1 1 1 7m
    :::

27. Existing objects can be viewed in a ready to use YAML output. Take a
    look at the existing **nginx** deployment.

    ::: cmd
    student@cp: $kubectl get deployments nginx -o yaml
          \end{cmd}
             \begin{yamllst}
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      annotations:
        deployment.kubernetes.io/revision: "1"
      creationTimestamp: null
      generation: 1
      labels:
        run: nginx
    <output_omitted>
          \end{yamllst}


             \item
             The output can also be viewed in JSON output.
             \begin{cmd}
    student@cp:~$ kubectl get deployment nginx -o json
    :::

28. The newly deployed **nginx** container is a light weight web server.
    We will need to create a `service` to view the default welcome page.
    Begin by looking at the help output. Note that there are several
    examples given, about halfway through the output.

    ::: cmd
    student@cp: $kubectl expose -h
          \end{cmd}
             
             \begin{out}[]
    <output_omitted>
          \end{out}


             \item
             Now try to gain access to the web server. As we have not
             declared a port to use you will receive an error.
             \begin{cmd}
    student@cp:~$ kubectl expose deployment/nginx
    :::

    ::: out
    error: couldn't find port via --port flag or introspection See
    'kubectl expose -h' for help and examples.
    :::

29. To change an object configuration one can use subcommands `apply`,
    `edit` or `patch` for non-disruptive updates. The `apply` command
    does a three-way diff of previous, current, and supplied input to
    determine modifications to make. Fields not mentioned are
    unaffected. The `edit` function performs a `get`, opens an editor,
    then an `apply`. You can update API objects in place with JSON patch
    and merge patch or strategic merge patch functionality.

    If the configuration has resource fields which cannot be updated
    once initialized then a disruptive update could be done using the
    `replace --force` option. This deletes first then re-creates a
    resource.

    Edit the file. Find the container name, somewhere around line 31 and
    add the port information as shown below.

    ::: cmd
    student@cp: $vim first.yaml
          \end{cmd}
             \begin{yamllst}[\texttt{first.yaml}]
    ....
       spec:
          containers:
          - image: nginx
            imagePullPolicy: Always
            name: nginx
            ports:                               # Add these
            - containerPort: 80                  # three
              protocol: TCP                      # lines
            resources: {}
    ....
          \end{yamllst}

             \item
             Due to how the object was created we will need to use
             \verb:replace: to terminate and create a new deployment.
             \begin{cmd}
    student@cp:~$ kubectl replace -f first.yaml --force
    :::

    ::: out
    deployment.apps/nginx replaced
    :::

30. View the Pod and Deployment. Note the `AGE` shows the Pod was
    re-created.

    ::: cmd
    student@cp: $kubectl get deploy,pod
          \end{cmd}
             
             \begin{out}[]
    NAME                          READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/nginx   1/1     1            1           2m4s

    NAME                         READY   STATUS    RESTARTS   AGE
    pod/nginx-7db75b8b78-qjffm   1/1     Running   0          8s
          \end{out}
             \item
             Try to expose the resource again. This time it should work.
             \begin{cmd}
    student@cp:~$ kubectl expose deployment/nginx
    :::

    ::: out
    service/nginx exposed
    :::

31. Verify the service configuration. First look at the service, then
    the endpoint information. Note the `ClusterIP` is not the current
    endpoint. Cilium provides the `ClusterIP`. The `Endpoint` is
    provided by `kubelet` and `kube-proxy`. Take note of the current
    endpoint IP. In the example below it is `192.168.1.5:80`. We will
    use this information in a few steps.

    ::: cmd
    student@cp: $kubectl get svc nginx
          \end{cmd}
             
             \begin{out}[]
    NAME      TYPE         CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
    nginx     ClusterIP    10.100.61.122   <none>        80/TCP    3m
          \end{out}
             \begin{cmd}
    student@cp:~$ kubectl get ep nginx
    :::

    ::: out
    NAME ENDPOINTS AGE nginx 192.168.1.5:80 26s
    :::

32. Determine which node the container is running on. Log into that node
    and use **tcpdump**, which you may need to install using **apt-get
    install**, to view traffic on the `tunl0`, as in tunnel zero,
    interface. The second node in this example. You may also see traffic
    on an interface which starts with `cili` and some string. Leave that
    command running while you run **curl** in the following step. You
    should see several messages go back and forth, including a `HTTP`
    HTTP/1.1 200 OK: and a `ack` response to the same sequence.

    ::: cmd
    student@cp: $kubectl describe pod nginx-7cbc4b4d9c-d27xw \
                      | grep Node:
          \end{cmd}
             
             \begin{out}[]
    Node:   worker/10.128.0.5
       \end{out}
             \begin{cmd}
    student@worker:~$ sudo tcpdump -i cilium_vxlan
    :::

    ::: out
    tcpdump: verbose output suppressed, use -v or -vv for full protocol
    decode listening on cilium_vxlan, link-type EN10MB (Ethernet),
    capture size 262144 bytes \<output_omitted\>
    :::

33. Test access to the Cluster IP, port 80. You should see the generic
    `nginx` installed and working page. The output should be the same
    when you look at the `ENDPOINTS` IP address. If the **curl** command
    times out the pod may be running on the other node. Run the same
    command on that node and it should work.

    ::: cmd
    student@cp: $curl 10.100.61.122:80
          \end{cmd}
             
             \begin{out}[]
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
    <output_omitted>
          \end{out}
             \begin{cmd}
    student@cp:~$ curl 192.168.1.5:80
    :::

34. Now scale up the deployment from one to three web servers.

    ::: cmd
    student@cp: $kubectl get deployment nginx
          \end{cmd}
             
             \begin{out}[]
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   1/1     1            1           12m
          \end{out}
             \begin{cmd}
    student@cp:~$ kubectl scale deployment nginx --replicas=3
    :::

    ::: out
    deployment.apps/nginx scaled
    :::

    ::: cmd
    student@cp: $kubectl get deployment nginx
          \end{cmd}
             
             \begin{out}[]
    NAME    READY   UP-TO-DATE   AVAILABLE   AGE
    nginx   3/3     3            3           12m

          \end{out}

             \item
             View the current endpoints. There now should be three. If
             the \verb:UP-TO-DATE: above said three, but \verb:AVAILABLE:
             said two wait a few seconds and try again, it could be slow
             to fully deploy.
             \begin{cmd}
    student@cp:~$ kubectl get ep nginx
    :::

    ::: out
    NAME ENDPOINTS AGE nginx
    192.168.0.3:80,192.168.1.5:80,192.168.1.6:80 7m40s
    :::

35. Find the oldest pod of the **nginx** deployment and delete it. The
    `Tab` key can be helpful for the long names. Use the `AGE` field to
    determine which was running the longest. You may notice activity in
    the other terminal where **tcpdump** is running, when you delete the
    pod. The pods with `192.168.0` addresses are probably on the cp and
    the `192.168.1` addresses are probably on the worker

    ::: cmd
    student@cp: $kubectl get pod -o wide
          \end{cmd}
             
             \begin{out}[]
    NAME                     READY     STATUS    RESTARTS   AGE   IP
    nginx-1423793266-7f1qw   1/1       Running   0          14m   192.168.1.5
    nginx-1423793266-8w2nk   1/1       Running   0          86s   192.168.1.6
    nginx-1423793266-fbt4b   1/1       Running   0          86s   192.168.0.3
         \end{out}
             \begin{cmd}
    student@cp:~$ kubectl delete pod nginx-1423793266-7f1qw
    :::

    ::: out
    pod \"nginx-1423793266-7f1qw\" deleted
    :::

36. Wait a minute or two then view the pods again. One should be newer
    than the others. In the following example nine seconds instead of
    four minutes. If your **tcpdump** was using the `veth` interface of
    that container it will error out. Also note we are using a short
    name for the object.

    ::: cmd
    student@cp: $kubectl get po
          \end{cmd}
             
             \begin{out}[]
    NAME                     READY     STATUS    RESTARTS   AGE
    nginx-1423793266-13p69   1/1       Running   0          9s
    nginx-1423793266-8w2nk   1/1       Running   0          4m1s
    nginx-1423793266-fbt4b   1/1       Running   0          4m1s
          \end{out}

             \item
             View the endpoints again. The original endpoint IP is no
             longer in use. You can delete any of the pods and the
             \texttt{service} will forward traffic to the existing
             backend pods.
             \begin{cmd}
    student@cp:~$ kubectl get ep nginx
    :::

    ::: out
    NAME ENDPOINTS AGE nginx
    192.168.0.3:80,192.168.1.6:80,192.168.1.7:80 12m
    :::

37. Test access to the web server again, using the `ClusterIP` address,
    then any of the endpoint IP addresses. Even though the endpoints
    have changed you still have access to the web server. This access is
    only from within the cluster. When done use **ctrl-c** to stop the
    **tcpdump** command.

    ::: cmd
    student@cp: $curl 10.100.61.122:80
          \end{cmd}
             
             \begin{out}[]
    <!DOCTYPE html>
    <html>
    <head>
    <title>Welcome to nginx!</title>
    <style>
        body
    <output_omitted>
          \end{out}
          \end{enumerate}
       \end{exe}

       \begin{exe} {Access from Outside the Cluster}

          \begin{lfbox}
             You can access a Service from outside the cluster
             using a DNS add-on or environment variables.  We
             will use environment variables to gain access to a Pod.
          \end{lfbox}
          \begin{enumerate}
             \item
             Begin by getting a list of the pods.
             \begin{cmd}
    student@cp:~$ kubectl get po
    :::

    ::: out
    NAME READY STATUS RESTARTS AGE nginx-1423793266-13p69 1/1 Running 0
    4m10s nginx-1423793266-8w2nk 1/1 Running 0 8m2s
    nginx-1423793266-fbt4b 1/1 Running 0 8m2s
    :::

38. Choose one of the pods and use the exec command to run **printenv**
    inside the pod. The following example uses the first pod listed
    above.

    ::: cmd
    student@cp: $kubectl exec nginx-1423793266-13p69 \
        -- printenv |grep KUBERNETES
          \end{cmd}
             
             \begin{out}[]
    KUBERNETES_SERVICE_PORT=443
    KUBERNETES_SERVICE_HOST=10.96.0.1
    KUBERNETES_SERVICE_PORT_HTTPS=443
    KUBERNETES_PORT=tcp://10.96.0.1:443
    <output_omitted>
          \end{out}

             \item
             Find and then delete the existing service for \textbf{nginx}.
             \begin{cmd}
    student@cp:~$ kubectl get svc
    :::

    ::: out
    NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE kubernetes ClusterIP
    10.96.0.1 \<none\> 443/TCP 4h nginx ClusterIP 10.100.61.122 \<none\>
    80/TCP 17m
    :::

39. Delete the service.

    ::: cmd
    student@cp: $kubectl delete svc nginx
          \end{cmd}
             
             \begin{out}[]
    service "nginx" deleted
          \end{out}
             \item
             Create the service again, but this time pass the
             \texttt{LoadBalancer} type. Check to see the status and
             note the external ports mentioned. The output will show the
             \texttt{External-IP} as \texttt{pending}. Unless a provider
             responds with a load balancer it will continue to show as
             pending.
             \begin{cmd}
    student@cp:~$ kubectl expose deployment nginx --type=LoadBalancer
    :::

    ::: out
    service/nginx exposed
    :::

    ::: cmd
    student@cp: $kubectl get svc
          \end{cmd}
             
             \begin{out}[]
    NAME         TYPE          CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
    kubernetes   ClusterIP     10.96.0.1        <none>        443/TCP        4h
    nginx        LoadBalancer  10.104.249.102   <pending>     80:32753/TCP   6s
          \end{out}

             \item

             Open a browser on your local system, not the lab exercise node, and
             use the public IP of your node and node port \verb:32753:, shown in the
             output above. If running the labs on remote nodes like
             \textbf{AWS} or \textbf{GCE} use
             the public IP you used with PuTTY or SSH to gain access. You may be
             able to find the IP address using \textbf{curl}.
             \begin{cmd}
    student@cp:~$ curl ifconfig.io
    :::

    ::: out
    54.214.214.156
    :::

    ![External Access via
    Browser](IMAGES/External_Access.png){width="7.0in"}

40. Scale the deployment to zero replicas. Then test the web page again.
    Once all pods have finished terminating accessing the web page
    should fail.

    ::: cmd
    student@cp: $kubectl scale deployment nginx --replicas=0
          \end{cmd}
             
             \begin{out}[]
    deployment.apps/nginx scaled
        \end{out}
             \begin{cmd}
    student@cp:~$ kubectl get po
    :::

    ::: out
    No resources found in default namespace.
    :::

41. Scale the deployment up to two replicas. The web page should work
    again.

    ::: cmd
    student@cp: $kubectl scale deployment nginx --replicas=2
          \end{cmd}
             
             \begin{out}[]
    deployment.apps/nginx scaled
         \end{out}
             \begin{cmd}
    student@cp:~$ kubectl get po
    :::

    ::: out
    NAME READY STATUS RESTARTS AGE nginx-1423793266-7x181 1/1 Running 0
    6s nginx-1423793266-s6vcz 1/1 Running 0 6s
    :::

42. Delete the deployment to recover system resources. Note that
    deleting a `deployment` does not delete the endpoints or services.

    ::: cmd
    student@cp: $kubectl delete deployments nginx
          \end{cmd}
             
             \begin{out}[]
    deployment.apps "nginx" deleted
          \end{out}
             \begin{cmd}
    student@cp:~$ kubectl delete svc nginx
    :::

    ::: out
    service \"nginx\" deleted
    :::
:::
:::
