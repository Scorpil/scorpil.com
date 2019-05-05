+++
title = "Kubernetes Toolchain Overview"
description = "Because plain kubernetes is (almost) never enough"
tags = [
    "kubernetes",
    "helm",
    "operators",
    "skaffold",
]
date = 2019-05-05T10:13:50Z
author = "Scorpil"
+++

Kubernetes appears to be a de-facto standard platform for running distributed containerized applications in 2019. With its increasing adoption rate, we get more and more tools designed to extend it, automate typical workflows or generally improve it in one way or another. I compiled a brief overview of the most popular tools people use with kubernetes in May 2019 to help myself navigate the ever-changing technological landscape of containerization era.

#### [Helm](https://helm.sh/)
* Backing: incubating at [CNCF](https://www.cncf.io/) and used by top tech companies in the world.
* GitHub: [11k+ stars](https://github.com/helm/helm)

Helm is the most widely recognized external tool in the kubernetes world. Ideologically, it’s the “missing package manager” of the platform. Helm gives a developer an ability to write reusable and configurable packages (charts) which can be distributed through a repository, and then installed on any kubernetes cluster with a single command. Helm allows you to see which charts are installed in a cluster, which ones were removed, what kubernetes resources each chart uses, check the status of installed charts and so on.

###### Use it if you want:
* to install an open source app into your cluster (good chance the chart is already present in the default repo);
* to use a powerful templating engine with kubernetes config files;
* to rely on 3rd party charts in your app (you can include them as dependencies to your chart);

###### Look for something else if you want:
* to customize application lifecycle management - as much as Helm is concerned, its job is finished after a package was deployed. If you need an ability to "keep an eye" on your app - your best bet is to develop an operator with [Operator SDK](https://github.com/operator-framework/operator-sdk), then use Helm to deploy that operator into a kubernetes cluster;
* to deploy your simple app with one command - just run `kubectl apply -f` on a directory of config files. Small or monolithic apps often have no reason to use Helm.

#### [Operators](https://coreos.com/operators/)
* Backing: developed by [CoreOS](https://coreos.com/), supported by [RedHat](https://www.redhat.com/).
* GitHub: [1k+ stars](https://github.com/operator-framework/operator-sdk)

Operators are essentially kubernetes extensions. With Operator SDK you can teach kubernetes how to “understand” new types of resources and how to keep the cluster status in sync with resource configurations.

Conceptually, Operator enables a developer to put application-specific operational knowledge into code and distribute it together with an app itself. When using operators, end users do not configure kubernetes primitives (deployments, services, replica sets, etc.) directly. They only need to concern themselves with application-level configuration (e.g. replication factor, retention policy). Operator is responsible for taking actions to bring the application to the desired state. Furthermore, it ensures that the application’s real state is equal to its desired state over time, by constantly monitoring application and reacting to the changes accordingly.

#### [Kustomize](https://kustomize.io/)
* Backing: embedded in `kubectl` as `kubectl apply -k` (kubernetes v1.14). It's here to stay.

Kustomize provides a way to combine configurational YAMLs in relatively simple ways, without resorting to templating. It fits right at home in the kubernetes ecosystem because it extends the declarative configuration style instead of replacing it with templating, like so many other tools.

A typical use case for this tool is the customization of application parameters for deploying the app in development vs. production environments. Most configuration parameters will probably (hopefully) be the same in both, but we want to be able to adapt things like volumes, secrets and environment variables. Kustomize is a good choice to achieve that.

###### Use it if you want:
* a simple, reliable tool for tweaking configuration files

###### Look for something else if you want:
* anything more than configuration management. Kustomize does one thing, but it does it well.

#### [Skaffold](https://skaffold.dev/)
* Backing: built and maintained by Google (as was kubernetes itself).
* GitHub: [6k+ stars](https://github.com/GoogleContainerTools/skaffold)

Skaffold is a surprisingly mature project, designed to automate code-build-push-deploy development cycle on a kubernetes cluster. It's tailored to satisfy the requirements of both development environment and CI/CD pipelines. Skaffold needs just one configuration file to work and is reasonably flexible. It supports multiple options for building containers (e.g. [Bazel](https://bazel.build/ ) and [Kaniko](https://github.com/GoogleContainerTools/kaniko)), as well as different deployment methods (kubectl, kustomize and Helm).

###### Use it if you want:
* to keep your development, testing, staging and production environments essentially the same;
* hot-reload-kinda-thing with kubernetes (be it [minikube](https://github.com/kubernetes/minikube), [kind](https://github.com/kubernetes-sigs/kind) or even actual remote cluster);

###### Look something else if you want:
* basic automation. Skaffold adds operational overhead on top of already quite complex kubernetes. Many things can be automated with a simple bash script, Makefile, gitlab's Auto DevOps, Draft, etc.

#### [Draft](https://draft.sh)
* Backing: developed by some people from Helm team, was acquired by Microsoft around two years ago.
* GitHub: [3k+ stars](https://github.com/Azure/draft)

Draft is a very opinionated tool which, like Skaffold, automates code-build-push-deploy development cycle. The difference is that while Skaffold allows you to configure pretty much everything to your liking, Draft decides which way is "the right way" for you. If you can live with that, Draft is unbelievably simple and easy to use. Unfortunately, this tool does not seem to be in active development right now.

###### Use it if you want:
* to use exclusively languages supported: .NET, Go, Node, PHP, Java (Maven, Gradle), Python & Ruby;
* to simplify operations for a simple app with no need for complex configuration (quick pet-project, maybe?);

###### Look for something else if you want:
* reliable, production-grade tool for long-term use;
* flexible configuration options - check Skaffold instead;