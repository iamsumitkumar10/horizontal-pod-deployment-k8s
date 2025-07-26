minikube addons enable metrics-server

kubectl top nodes

# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 1
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
        resources:
          requests:
            cpu: "100m"
            memory: "50Mi"
          limits:
            cpu: "200m"
            memory: "100Mi"


kubectl apply -f nginx-deployment.yaml

kubectl expose deployment nginx --port=80

# nginx-hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 30


kubectl apply -f nginx-hpa.yaml

kubectl get hpa -w



Step 5: Generate Load to Test Autoscaling
Open a new terminal and run this command to generate CPU load:

bash
kubectl run -i --tty load-generator --rm --image=busybox --restart=Never -- /bin/sh -c "while true; do wget -q -O- http://nginx; done"



In your original terminal, watch the HPA and pods scale up:

bash
# Watch HPA
kubectl get hpa -w

# Watch pods
kubectl get pods -w

Step 6: Clean Up

When done testing:

kubectl delete deployment nginx
kubectl delete service nginx
kubectl delete hpa nginx-hpa


