# my-k8s-ingress-demo-code 

To complete the Demo of Ingress and ingress rules, Attched are the yaml files that can be used to deploy pods, service, Imgress controller(Nginx) and Ingress Rules.

Once this is created:

you can access your applucation on LB, but it will redirect to only root or / application- home application

So in order to access and test all the path based routing are working , you can alternatively use node port

http:// IP of worker node:node port/path

You can get the nodeport using  command : kubectl get svc -n=ingress-nginx

Ingress Design from K8s IO:

![image](https://user-images.githubusercontent.com/72337263/181738604-9f28b4f6-5f8a-4e12-83ce-8c62eb374a8d.png)
