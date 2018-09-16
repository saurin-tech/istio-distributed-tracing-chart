# istio-distributed-tracing-chart

## Brief Intro

Micro-services are awesome, well someone might argue with that.  But we are not gathered here today to have a discussion on that.  We are gathered here today to have a discussion about once you decided to let micro-services in to your gang of applications and then you start having problems; like which one is not keeping it's end of the deal and what not.  How do you track down the person that's not keeping their end of the deal?  Before you get all worked up about someone not keeping their end of the deal let me introduce a solution called [Istio](https://istio.io).  

Istio let's you connect, secure, control, and observe services.  This is coming straight from their website, all I've done in this project is create a helm chart and used some spring-boot application to generate data to show you how easy it is to do distributed tracing in Kubernetes with the help of Istio. To learn more about distributed tracing [visit](https://medium.com/nikeengineering/hit-the-ground-running-with-distributed-tracing-core-concepts-ff5ad47c7058).

## My setup

- Host OS: Ubuntu 18.04
- Docker: 18.06.0-ce
- Minikube: 0.28.2
- Kubectl:

    - Client version: 1.11.2
    - Server version: 1.10.0
- Helm:

    - Client version: 2.9.1
    - Server version: 2.9.1
- Istio: 1.0.2

## Before you get started

You will need to have Docker, Kubectl, Minikube and Helm installed.  This project builds on knowledge obtained through out my other tutorials, so to learn more about it you can visit my [github](https://github.com/saurin-tech) where I've created tutorials on [Docker](https://github.com/saurin-tech/hello-docker), [Kubernetes](https://github.com/saurin-tech/hello-docker-deployment), and [Helm](https://github.com/saurin-tech/hello-docker-chart).

You also will need three containers to generate some dummy data. I have written up some spring-boot applications that you can use to generate this dummy data.  They are called [tracer-box-one](https://github.com/saurin-tech/tracer-box-one), [tracer-box-two](https://github.com/saurin-tech/tracer-box-two), and [tracer-box-three](https://github.com/saurin-tech/tracer-box-three).

## Installing Istio

- Things to keep in mind before installing.

    - If you have other deployments of Helm running delete them before installing Istio using helm.  It causes problems or at least it did for me.
    - If your Minikube only has 2gb of memory allocated to it bump it up to 4gb.

To install [Istio](https://istio.io/docs/setup/kubernetes/helm-install/ ) follow directions on this page.  I used option 2 to install Istio on my machine. The last command to install Istio using helm needs to be tweaked a bit to accommodate for distributed tracing.  Instead of `helm install install/kubernetes/helm/istio --name istio --namespace istio-system` use `helm install install/kubernetes/helm/istio --name istio --namespace istio-system --set tracing.enabled=true`

- Errors I encountered during install process:

    - `Error: apiVersion "networking.istio.io/v1alpha3" in istio/charts/pilot/templates/gateway.yaml is not available`

        - To fix it I ran `kubectl apply -f install/kubernetes/helm/istio/templates/crds.yaml`
        - I found the above mentioned fix at [here](https://github.com/istio/istio/issues/7530).

FYI - Here are the instructions for [Istio's Distributed Tracing](https://istio.io/docs/tasks/telemetry/distributed-tracing/).

## How to run

At this point I'm assuming that you've installed everything correctly and you have read my other instructions that I've pointed out earlier, so you know how to build images, containers and put those containers in the Minikube etc.

- Start Minikube `minikube start`
- Set up access to Jaeger dashboard `$ kubectl port-forward -n istio-system $(kubectl get pod -n istio-system -l app=jaeger -o jsonpath='{.items[0].metadata.name}') 16686:16686 &`
- To access Jaeger dashboard [visit](http://localhost:16686).
- Once the dashboard is up and running run `helm install ./istio-distributed-tracing-chart` 
    - Helm install will start three deployments and three services.
- Find IP of your minikube `minikube ip`
- To test if the services are running or not go to `<minikubeip:30002/hello>`
- After that go to the Jaeger dashboard refresh the page and click on the Services drop down to check the activity.  If the activity show up proceed to the next step.
- Now for the fun part to generate API calls:
    - Go to url `<minikubeip:30002/start>`
- To stop the API activity after about 30 seconds you should have plenty of data.
    - To stop API activity `<minikubeip:30002/stop>`
- To view the distributed tracing data generated go to Jaeger Dashboard.
    - Select Service drop down and select `tracer-box-one` and click Find Traces.

Click on different traces to learn what path they took.