---
title: 'Kubernetes and Data Engineering'
date: 2022-12-11
permalink: /posts/2022/12/k8s-for-a-data-eng/
tags:
  - learning
  - kubernetes
---

# Kubernetes and Data Engineering

A note about "Data Engineering". Data Engineering can mean many different things depending upon one's exposure to different aspects of the extremely broad field. Instead of providing my definition of data engineering, I would encourage everyone to take a look at the [data engineer roadmap](https://github.com/datastacktv/data-engineer-roadmap) which I believe has many different tools that I use on a regular basis (and others I would like to work with).

The below text is a mixture of how I work with kubernetes and some tools I've found are useful.


## [Kubernetes](https://kubernetes.io/)

Using the cool new tool, [chatgpt](chat.openai.com/chat), kubernetes is "an open-source container orchestration system for automating the deployment, scaling, and management of containerized applications. 

Containers are a way to package software in a way that is portable and easy to deploy. They allow you to package up an application, its dependencies, and its configuration into a single container image that can be run on any machine that has a container runtime installed. This makes it easy to deploy applications consistently, regardless of the underlying infrastructure.

Kubernetes is a tool that helps you manage a fleet of containerized applications. It provides a number of features that make it easier to deploy, scale, and manage containerized applications at scale."

Essentially, the end result of how I like to think about kubernetes is that I as a data engineer get access to any amount of compute resources I want, in a portable environment.

### How I use it

Getting started is non-trivial, however the kubernetes documentation is excellent, and a primer with a simple python example is located [here](https://kubernetes.io/blog/2019/07/23/get-started-with-kubernetes-using-python/). 

Ok so we have the basics, but kubernetes can be overwhelming especially at first. Usually, I myself am a command line fan for interacting with new tools, however in this instance I highly recommend downloading and installing the GUI for kubernetes: [Lens](https://k8slens.dev/).

#### A note on authentication

Kubernetes authentication can be weird, especially using `yaml` for credentials files. All you need to know is, `~/.kube/` should have your `kubeconfig.yaml` file(s), and you must have your `$KUBECONFIG` path variable pointing to the path of your file (`~/.kube/kubeconfig.yaml`).

Lens should automatically display the cluster if it is set up right.

### Continuing on

Typically if I get a notification about an error from a pod, and I want to quickly diagnose an issue, I will view the pod logs directly from lens. If I get a cryptic error and need to debug, I will typically run the script locally, replicate the issue, and fix it. 

Usually if I am unable to reproduce the issue locally, typically its something with permissions or networking (*it's always DNS*). This however helps me identify that the issue has something/nothing to do with kubernetes itself, and I can continue with my debugging.

On top of that, say I get a `KeyError` in kubernetes. I know then that my kubernetes environment is missing an environment variable (say a secret or configuration argument). By clicking on the pod in lens, I can view all the environment variables that are attached to that pod. If there are a lot of variables and I just want to grep them quickly, I will click the pod shell button which executes the `kubectl exec` command into the pod I clicked on for me (which is really nifty), and then do an `env | grep FOO` or similar to see if the variable is set. 

If it is not set I'll check the deployment or job yaml, and see if the secret key reference is defined in the yaml. I have been burned in the past where I set the secret key reference but forgot to set the secret in the k8s secrets, and in those instances the pod will not even spin up and error immediately after being triggered.

This I can take care of easily by base64 encoding my variable and saving in the k8s secrets. It is as simple as clicking the edit button of the secret key reference and pasting the key and value in with a colon separator. The name need not be base64 encoded, however the value needs to be.

Suppose the secret_key with the value "secret_value". I have provided two small bash functions to add to your toolkit. 

```bash
## base64 string encode decode easy (with encode ignoring new line)
bsencode() { echo -n $@ | base64 }
bsdecode() { echo $@ | base64 --decode }
```

Thus I will run `bsencode secret_value` in my terminal, and get back the base 64 encoded `c2VjcmV0X3ZhbHVl` string. 

As we can see by the following, we can decode the value like so:

```bash
$ bsdecode c2VjcmV0X3ZhbHVl
secret_value
```

### Ok, but what else?

In my opinion, having image tags defined as an environment variable in every pod you run is an essential **best practice**. Suppose a requirement is updated, but your image tag is set to "latest". If a script requires a new dependency in that updated requirements.txt file, but the container registry does not upload the image properly for any reason, it is much easier seeing a "Image could not be pulled" error as opposed to viewing the subsequent `ImportError` or `NameError` that occurs at runtime. 

It also ensures you have a proper CI pipeline using image tagging in the first place. Being able to pull a specific image down locally, even just knowing which image it was and pushing a new image with a new id and running each side by side to verify changes worked, etc. I have found it very helpful.


### Yaml?

I have a love/hate relationship with yaml. On the one hand, I like it being a superset of json, and I can put comments within the file and so much more. On the other hand, I am terrible at synthesizing yaml myself. 

If I need to create a deployment yaml, I'll copy paste one I've found online, or copy paste a devops wizardry and change what I need within the file, and likewise for a job yaml.

I rarely use `kubectl apply -f *.yaml` myself because I'm not a great yaml writer, but on occasion for simple creations I will. If you're more comfortable using an api interface, you can with python using the official [kubernetes python client library](https://github.com/kubernetes-client/python)! 

This I find makes sure I'm deploying what I want in the namespace I want and using the right yaml file and more. You can use [kube-ps1](https://github.com/jonmosco/kube-ps1) in your terminal as well to know what cluster and namespace you are in in your terminal.

### Monitoring

The beauty of deploying an application on kubernetes is its running somewhere else with access to a lot of resources. With that comes the need for **good** monitoring. Having used all three, in combination or separately, you can't go wrong with these:

- [Prometheus](https://prometheus.io/)
- [Grafana](https://grafana.com/)
- [Datadog](https://www.datadoghq.com/)


Also, a "Warning" in kubernetes is an error. Confusing, I know.


## Where to go from here

I would start with taking an old project and trying to run it in kubernetes. Start simple at first, maybe something which just sleeps as a "deployment". Then add some requirements and copy the data movement bits of code into that python deploy script. Connect it to a container registry, add some secrets, and run it.

Having developed and deployed production grade code for most of my working career on k8s, it is **the** tool that in my opinion, all engineers writing application code should familiarize themselves with. There is no better way to handle compute, especially when you're dealing with a lot of it.
