
# **Create an EKS Cluster and Deploy 2048 Game**  

---

## **Task 1: Create an EKS Cluster**  
### **Cluster Details:**  
- **Cluster Name:** `<yourname>-eks-cluster-1`  
- **K8s Version:** 1.31

### **Create IAM Roles:**  
1. Create **eks-cluster-role** → Attach:  
   - ✅ AmazonEKSClusterPolicy  

2. Create **eks-node-grp-role** → Attach:  
   - ✅ AmazonEKSWorkerNodePolicy  
   - ✅ AmazonEC2ContainerRegistryReadOnly  
   - ✅ AmazonEKS_CNI_Policy  

### **Configuration:**  
- **VPC:** Default  
- **Subnets:** 2 or 3  
- **Security Group:** Open ports → **22, 80, 8080**  
- **Cluster Endpoint Access:** Public  

### **Add-ons:**  
- VPC CNI, CoreDNS, and kube-proxy → Use **default versions**  

✅ **Create Cluster** → Takes about **10–12 minutes**  
✅ Wait until the status shows **Active**  

---

## **Task 2: Add Node Groups**  
1. Open the cluster → **Compute → Add Node Group**  
2. **Node Group Name:** `<yourname>-eks-nodegrp-1`  
3. **Role:** Select the previously created **eks-node-grp-role**  

### **Settings:**  
- **AMI:** Default (Amazon Linux 2)  
- **Desired/Min/Max Nodes:** 1  
- **Enable SSH Access:** Choose the security group with **22, 80, 8080** open  

✅ **Create Node Group** → Takes about **2–3 minutes**  

---

## **Task 3: Authenticate to Cluster**  
1. Open **Cloud Shell**  
2. Check your AWS identity:  
```bash
aws sts get-caller-identity
```  
3. Create kubeconfig file:  
```bash
aws eks update-kubeconfig --region <region-code> --name <your-cluster-name>
```
Example:  
```bash
aws eks update-kubeconfig --region us-east-1 --name unus-eks-cluster-1
```
4. Check nodes:  
```bash
kubectl get nodes
```  

---

## **Task 4: Deploy 2048 Game**  
1. Create a pod config file:  
```bash
nano 2048-pod.yaml
```

**2048-pod.yaml:**  
```yaml
apiVersion: v1
kind: Pod
metadata:
   name: 2048-pod
   labels:
      app: 2048-ws
spec:
   containers:
   - name: 2048-container
     image: blackicebird/2048
     ports:
       - containerPort: 80
```

2. Apply the config:  
```bash
kubectl apply -f 2048-pod.yaml
```
3. Check the pod:  
```bash
kubectl get pods
```

---

## **Task 5: Setup Load Balancer Service**  
1. Create service config file:  
```bash
nano mygame-svc.yaml
```

**mygame-svc.yaml:**  
```yaml
apiVersion: v1
kind: Service
metadata:
   name: mygame-svc
spec:
   selector:
      app: 2048-ws
   ports:
   - protocol: TCP
     port: 80
     targetPort: 80
   type: LoadBalancer
```

2. Apply the config:  
```bash
kubectl apply -f mygame-svc.yaml
```
3. Check service details:  
```bash
kubectl describe svc mygame-svc
```
4. Get the LoadBalancer URL and test it:  
```bash
curl <LoadBalancer_Ingress>:80
```
or open the DNS in a browser → **Play 2048**  

---

## **Cleanup**  
1. Delete the pod:  
```bash
kubectl delete -f 2048-pod.yaml
```
2. Delete the service:  
```bash
kubectl delete -f mygame-svc.yaml
```
3. Delete the node group  
4. Delete the EKS cluster  

---

✅ **Done!** 😎
