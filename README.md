To deploy the two containers that scale independently, we will create two Kubernetes Deployments, one for the users container and one for the shifts container. Each deployment will have its own replica set, which will manage the scaling of the containers. We will also create two Kubernetes Services, one for each deployment, to expose the containers as APIs.
Here is an example YAML file that creates the deployments and services:
`````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: users-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: users
  template:
    metadata:
      labels:
        app: users
    spec:
      containers:
      - name: users-container
        image: <docker-image>
        ports:
        - containerPort: 8080
`````

`````
apiVersion: v1
kind: Service
metadata:
  name: users-service
spec:
  selector:
    app: users
  ports:
  - name: http
    port: 80
    targetPort: 8080
`````


`````
apiVersion: apps/v1
kind: Deployment
metadata:
  name: shifts-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: shifts
  template:
    metadata:
      labels:
        app: shifts
    spec:
      containers:
      - name: shifts-container
        image: <docker-image>
        ports:
        - containerPort: 8080
`````

`````
apiVersion: v1
kind: Service
metadata:
  name: shifts-service
spec:
  selector:
    app: shifts
  ports:
  - name: http
    port: 80
    targetPort: 8080
`````


To auto scale the services when CPU reaches 70%, we can use the Kubernetes HorizontalPodAutoscaler (HPA). We will create an HPA for each deployment that targets the deployment's replica set, and set the target CPU utilization to 70%.
Here is an example YAML file that creates the HPAs:
`````
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: users-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: users-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 70
`````

`````
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: shifts-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: shifts-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      targetAverageUtilization: 70
`````
To handle rolling deployments and rollbacks, we will use Kubernetes Deployments. When we update the deployment's container image or configuration, Kubernetes will create a new replica set with the updated configuration and gradually replace the old replica set's pods with new ones. If there are any issues with the new pods, we can roll back the deployment to the previous configuration by using the kubectl rollout undo command.
To restrict the development team's access to certain commands, we will create a Kubernetes Role that grants the team permission to create, update, and list deployments and services, but not delete or modify them.We will also create a Kubernetes RoleBinding that assigns the Role to the development team's Kubernetes ServiceAccount.
Here is an example YAML file that creates the Role and RoleBinding:
`````
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: deployment-operator
rules:
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["create", "update", "list"]
- apiGroups: [""]
  resources: ["services"]
  verbs: ["create", "update", "list"]
`````

`````
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: development-team
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: deployment-operator
subjects:
- kind: ServiceAccount
  name: development-team
`````

To apply the configs to multiple environments, such as staging and production, we can create separate Kubernetes namespaces for each environment, and deploy the same YAML files to each namespace. This allows us to isolate the resources and configurations for each environment and prevents any interference between them.
Here is an example YAML file for creating an HPA that scales based on CPU utilization:


To auto-scale the deployment based on network latency instead of CPU, we can use the Kubernetes HorizontalPodAutoscaler with the target average network latency as the metric. We can use a Kubernetes Custom Metric Adapter to expose the network latency as a custom metric, and then configure the HPA to use the custom metric as the target metric. This requires setting up the custom metric adapter separately and configuring it to scrape the necessary network latency data.
Here is an example YAML file for creating an HPA that scales based on network latency:
`````
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: users-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: users-deployment
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Object
    object:
      metricName: network-latency
      target:
        type: AverageValue
        averageValue: 100ms
`````
In this example, we use a custom metric named "network-latency" to scale the users deployment. The target metric is set to the average value of the custom metric, and the target value is set to 100ms. This HPA will automatically scale the deployment based on the network latency of the application.

Overall, the solution to this exercise involves creating Kubernetes Deployments and Services for two containers that scale independently, using HPAs to auto-scale the services based on CPU utilization, using Deployments to handle rolling deployments and rollbacks, and using RBAC to restrict access to certain Kubernetes commands. We can also apply the same configs to multiple environments by using separate Kubernetes namespaces for each environment, and auto-scale the deployment based on network latency by using a custom metric adapter and configuring the HPA to use the custom metric.
