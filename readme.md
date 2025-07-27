> minikube addons enable metrics-server

> kubectl top nodes

> kubectl apply -f nginx-deployment.yaml

> kubectl apply -f nginx-hpa.yaml

> kubectl apply -f service.yaml

> kubectl get hpa -w

Generate Load to Test Autoscaling
Open a new terminal and run this command to generate CPU load:

> kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://nginx; done"

In your original terminal, watch the HPA and pods scale up:
Watch HPA
> kubectl get hpa -w

Watch pods
> kubectl get pods -w




