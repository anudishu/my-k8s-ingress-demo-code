Deployments
A Deployment provides declarative updates for Pods and ReplicaSets.

You describe a desired state in a Deployment, and the Deployment Controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.

Note: Do not manage ReplicaSets owned by a Deployment. Consider opening an issue in the main Kubernetes repository if your use case is not covered below.
Use Case
The following are typical use cases for Deployments:

Create a Deployment to rollout a ReplicaSet. The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
Declare the new state of the Pods by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
Rollback to an earlier Deployment revision if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
Scale up the Deployment to facilitate more load.
Pause the rollout of a Deployment to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
Use the status of the Deployment as an indicator that a rollout has stuck.
Clean up older ReplicaSets that you don't need anymore.
Creating a Deployment
The following is an example of a Deployment. It creates a ReplicaSet to bring up three nginx Pods:

controllers/nginx-deployment.yaml Copy controllers/nginx-deployment.yaml to clipboard
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
In this example:

A Deployment named nginx-deployment is created, indicated by the .metadata.name field.

The Deployment creates three replicated Pods, indicated by the .spec.replicas field.

The .spec.selector field defines how the Deployment finds which Pods to manage. In this case, you select a label that is defined in the Pod template (app: nginx). However, more sophisticated selection rules are possible, as long as the Pod template itself satisfies the rule.

Note: The .spec.selector.matchLabels field is a map of {key,value} pairs. A single {key,value} in the matchLabels map is equivalent to an element of matchExpressions, whose key field is "key", the operator is "In", and the values array contains only "value". All of the requirements, from both matchLabels and matchExpressions, must be satisfied in order to match.
The template field contains the following sub-fields:

The Pods are labeled app: nginxusing the .metadata.labels field.
The Pod template's specification, or .template.spec field, indicates that the Pods run one container, nginx, which runs the nginx Docker Hub image at version 1.14.2.
Create one container and name it nginx using the .spec.template.spec.containers[0].name field.
Before you begin, make sure your Kubernetes cluster is up and running. Follow the steps given below to create the above Deployment:

Create the Deployment by running the following command:

kubectl apply -f https://k8s.io/examples/controllers/nginx-deployment.yaml
Run kubectl get deployments to check if the Deployment was created.

If the Deployment is still being created, the output is similar to the following:

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   0/3     0            0           1s
When you inspect the Deployments in your cluster, the following fields are displayed:

NAME lists the names of the Deployments in the namespace.
READY displays how many replicas of the application are available to your users. It follows the pattern ready/desired.
UP-TO-DATE displays the number of replicas that have been updated to achieve the desired state.
AVAILABLE displays how many replicas of the application are available to your users.
AGE displays the amount of time that the application has been running.
Notice how the number of desired replicas is 3 according to .spec.replicas field.

To see the Deployment rollout status, run kubectl rollout status deployment/nginx-deployment.

The output is similar to:

Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
deployment "nginx-deployment" successfully rolled out
Run the kubectl get deployments again a few seconds later. The output is similar to this:

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           18s
Notice that the Deployment has created all three replicas, and all replicas are up-to-date (they contain the latest Pod template) and available.

To see the ReplicaSet (rs) created by the Deployment, run kubectl get rs. The output is similar to this:

NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-75675f5897   3         3         3       18s
ReplicaSet output shows the following fields:

NAME lists the names of the ReplicaSets in the namespace.
DESIRED displays the desired number of replicas of the application, which you define when you create the Deployment. This is the desired state.
CURRENT displays how many replicas are currently running.
READY displays how many replicas of the application are available to your users.
AGE displays the amount of time that the application has been running.
Notice that the name of the ReplicaSet is always formatted as [DEPLOYMENT-NAME]-[RANDOM-STRING]. The random string is randomly generated and uses the pod-template-hash as a seed.

To see the labels automatically generated for each Pod, run kubectl get pods --show-labels. The output is similar to:

NAME                                READY     STATUS    RESTARTS   AGE       LABELS
nginx-deployment-75675f5897-7ci7o   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-kzszj   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
nginx-deployment-75675f5897-qqcnn   1/1       Running   0          18s       app=nginx,pod-template-hash=3123191453
The created ReplicaSet ensures that there are three nginx Pods.

Note:
You must specify an appropriate selector and Pod template labels in a Deployment (in this case, app: nginx).

Do not overlap labels or selectors with other controllers (including other Deployments and StatefulSets). Kubernetes doesn't stop you from overlapping, and if multiple controllers have overlapping selectors those controllers might conflict and behave unexpectedly.

Pod-template-hash label
Caution: Do not change this label.
The pod-template-hash label is added by the Deployment controller to every ReplicaSet that a Deployment creates or adopts.

This label ensures that child ReplicaSets of a Deployment do not overlap. It is generated by hashing the PodTemplate of the ReplicaSet and using the resulting hash as the label value that is added to the ReplicaSet selector, Pod template labels, and in any existing Pods that the ReplicaSet might have.

Updating a Deployment
Note: A Deployment's rollout is triggered if and only if the Deployment's Pod template (that is, .spec.template) is changed, for example if the labels or container images of the template are updated. Other updates, such as scaling the Deployment, do not trigger a rollout.
Follow the steps given below to update your Deployment:

Let's update the nginx Pods to use the nginx:1.16.1 image instead of the nginx:1.14.2 image.

kubectl set image deployment.v1.apps/nginx-deployment nginx=nginx:1.16.1
or use the following command:

kubectl set image deployment/nginx-deployment nginx=nginx:1.16.1
The output is similar to:

deployment.apps/nginx-deployment image updated
Alternatively, you can edit the Deployment and change .spec.template.spec.containers[0].image from nginx:1.14.2 to nginx:1.16.1:

kubectl edit deployment/nginx-deployment
The output is similar to:

deployment.apps/nginx-deployment edited
To see the rollout status, run:

kubectl rollout status deployment/nginx-deployment
The output is similar to this:

Waiting for rollout to finish: 2 out of 3 new replicas have been updated...
or

deployment "nginx-deployment" successfully rolled out
Get more details on your updated Deployment:

After the rollout succeeds, you can view the Deployment by running kubectl get deployments. The output is similar to this:

NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   3/3     3            3           36s
Run kubectl get rs to see that the Deployment updated the Pods by creating a new ReplicaSet and scaling it up to 3 replicas, as well as scaling down the old ReplicaSet to 0 replicas.

kubectl get rs
The output is similar to this:

NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-1564180365   3         3         3       6s
nginx-deployment-2035384211   0         0         0       36s
Running get pods should now show only the new Pods:

kubectl get pods
The output is similar to this:

NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-1564180365-khku8   1/1       Running   0          14s
nginx-deployment-1564180365-nacti   1/1       Running   0          14s
nginx-deployment-1564180365-z9gth   1/1       Running   0          14s
Next time you want to update these Pods, you only need to update the Deployment's Pod template again.

Deployment ensures that only a certain number of Pods are down while they are being updated. By default, it ensures that at least 75% of the desired number of Pods are up (25% max unavailable).

Deployment also ensures that only a certain number of Pods are created above the desired number of Pods. By default, it ensures that at most 125% of the desired number of Pods are up (25% max surge).

For example, if you look at the above Deployment closely, you will see that it first creates a new Pod, then deletes an old Pod, and creates another new one. It does not kill old Pods until a sufficient number of new Pods have come up, and does not create new Pods until a sufficient number of old Pods have been killed. It makes sure that at least 3 Pods are available and that at max 4 Pods in total are available. In case of a Deployment with 4 replicas, the number of Pods would be between 3 and 5.

Get details of your Deployment:

kubectl describe deployments
The output is similar to this:

Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Thu, 30 Nov 2017 10:56:25 +0000
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision=2
Selector:               app=nginx
Replicas:               3 desired | 3 updated | 3 total | 3 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
   Containers:
    nginx:
      Image:        nginx:1.16.1
      Port:         80/TCP
      Environment:  <none>
      Mounts:       <none>
    Volumes:        <none>
  Conditions:
    Type           Status  Reason
    ----           ------  ------
    Available      True    MinimumReplicasAvailable
    Progressing    True    NewReplicaSetAvailable
  OldReplicaSets:  <none>
  NewReplicaSet:   nginx-deployment-1564180365 (3/3 replicas created)
  Events:
    Type    Reason             Age   From                   Message
    ----    ------             ----  ----                   -------
    Normal  ScalingReplicaSet  2m    deployment-controller  Scaled up replica set nginx-deployment-2035384211 to 3
    Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 1
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 2
    Normal  ScalingReplicaSet  22s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 2
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 1
    Normal  ScalingReplicaSet  19s   deployment-controller  Scaled up replica set nginx-deployment-1564180365 to 3
    Normal  ScalingReplicaSet  14s   deployment-controller  Scaled down replica set nginx-deployment-2035384211 to 0
Here you see that when you first created the Deployment, it created a ReplicaSet (nginx-deployment-2035384211) and scaled it up to 3 replicas directly. When you updated the Deployment, it created a new ReplicaSet (nginx-deployment-1564180365) and scaled it up to 1 and waited for it to come up. Then it scaled down the old ReplicaSet to 2 and scaled up the new ReplicaSet to 2 so that at least 3 Pods were available and at most 4 Pods were created at all times. It then continued scaling up and down the new and the old ReplicaSet, with the same rolling update strategy. Finally, you'll have 3 available replicas in the new ReplicaSet, and the old ReplicaSet is scaled down to 0.

Note: Kubernetes doesn't count terminating Pods when calculating the number of availableReplicas, which must be between replicas - maxUnavailable and replicas + maxSurge. As a result, you might notice that there are more Pods than expected during a rollout, and that the total resources consumed by the Deployment is more than replicas + maxSurge until the terminationGracePeriodSeconds of the terminating Pods expires.