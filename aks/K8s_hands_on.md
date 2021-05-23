My k8s hands-on
===============
with AKS

5/22
Reference : https://github.com/tadeugr/aks-references/

// define values
```
AKS_NAME="hyo-cluster"
AKS_RG="rg-hyo-cluster"
SUBSCRIPTION="ae36a85f-76be-4bb1-bc9e-e92d8a0af831"
REGION="koreacentral"
```
// resource group
```
az group create \
  --location $REGION \
  --name $AKS_RG \
  --subscription $SUBSCRIPTION
```
// service principal
```
az ad sp create-for-rbac \
  --skip-assignment \
  -n "sp-aks"
Changing "sp-aks" to a valid URI of "http://sp-aks", which is the required format used for service principal names
{
  "appId": "cb07b3ab-4943-4450-b31a-82ffa63b3790",
  "displayName": "sp-aks",
  "name": "http://sp-aks",
  "password": "7e5e0be0-9fec-4c0c-8a3e-529899b9fcea",
  "tenant": "ab5cf8c1-0907-4db6-a510-9feba384f8a3"
}
```
//
```
az ad sp list -o json | jq ".[].displayName"
"aciapi"
"O365 Demeter"
"AAD Request Verification Service - PROD"
"Jarvis Transaction Service"
"Microsoft Modern Contact Master"
"Billing RP"
"Azure ESTS Service"
"Windows Azure Service Management API"
"Microsoft App Access Panel"
"Signup"
"Azure Resource Graph"
"Office 365 Configure"
"Microsoft Graph Change Tracking"
"Microsoft.SMIT"
"Policy Administration Service"
"Windows Azure Active Directory"
"AzureSupportCenter"
"Microsoft Graph"
"sp-aks"
"IAM Supportability"
"MCAPI Authorization Prod"
```
//
```
az aks create \
  --location $REGION \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME \
  --ssh-key-value $HOME/.ssh/id_rsa.pub \
  --service-principal "cb07b3ab-4943-4450-b31a-82ffa63b3790" \
  --client-secret "7e5e0be0-9fec-4c0c-8a3e-529899b9fcea" \
  --network-plugin azure \
  --load-balancer-sku standard \
  --outbound-type loadBalancer \
  --node-vm-size Standard_B2s \
  --node-count 1 \
  --tags 'ENV=DEV' 'SRV=EXAMPLE2'

// backups --
//  --network-plugin kubenet \
//  --load-balancer-sku basic \
// ----
```
// delete
```
az aks delete \
  --subscription $SUBSCRIPTION \
  --resource-group $AKS_RG \
  --name $AKS_NAME
FILE="./$AKS_NAME.kubeconfig"
```
// get credentials
```
az aks get-credentials \
  -n $AKS_NAME \
  -g $AKS_RG \
  --subscription $SUBSCRIPTION \
  --admin \
  --file $FILE
export KUBECONFIG=$FILE
```
==============
Create VM : for testing

```
VM_NAME="hyo-test-vm"
VM_RG="rg-hyo-test-vm"
//SUBSCRIPTION="REPLACE-WITH-YOUR-VM-SUBSCRIPTION"
//REGION="REPLACE-WITH-YOUR-VM-REGION"
```
// group
```
az group create \
  --location $REGION \
  --name $VM_RG \
  --subscription $SUBSCRIPTION
{
  "id": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/rg-hyo-test-vm",
  "location": "koreacentral",
  "managedBy": null,
  "name": "rg-hyo-test-vm",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null,
  "type": "Microsoft.Resources/resourceGroups"
}
```
----
// create vm just for test
```
az vm create \
 --location $REGION \
 --subscription $SUBSCRIPTION \
 --resource-group $VM_RG \
 --name $VM_NAME \
 --ssh-key-values $HOME/.ssh/id_rsa.pub \
 --admin-username devops \
 --image UbuntuLTS \
 --nsg-rule SSH \
 --public-ip-address-allocation static \
 --public-ip-sku Basic \
 --size Standard_A2_v2
{- Finished ..
 "fqdns": "",
 "id": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/rg-hyo-test-vm/providers/Microsoft.Compute/virtualMachines/hyo-test-vm",
 "location": "koreacentral",
 "macAddress": "00-22-48-05-37-77",
 "powerState": "VM running",
 "privateIpAddress": "10.0.0.4",
 "publicIpAddress": "20.194.5.74",
 "resourceGroup": "rg-hyo-test-vm",
 "zones": ""
}
```
// connect to VM

```
ssh devops@20.194.5.74
```
// apply following
```
kind: Service
apiVersion: v1
metadata:
  name: my-custom-egress
spec:
  ports:
  - protocol: TCP
    port: 60000
  type: LoadBalancer
```
//
```
기본적으로 kubectl은 $HOME/.kube 디렉터리에서 config라는 이름의 파일을 찾는다. KUBECONFIG 환경 변수를 설정하거나 --kubeconfig 플래그를 지정해서 다른 kubeconfig 파일을 사용할 수 있다.
kubectl get pods --kubeconfig my_study_note/hyo-cluster.kubeconfig
export KUBECONFIG=$FILE    // set as ENV
```
[ HANDS-ON ] External nginx-ingress / cert-manager (letsencrypt) / external-dns
-------------------------------------
```
az ad sp create-for-rbac -n sp-external-dns
{
  "appId": "cd4d61e4-ac40-4f24-b6c4-a0ca1b62b1af",
  "displayName": "sp-external-dns",
  "name": "http://sp-external-dns",
  "password": "fjQHBwsKcWVzRnEcJD-wLWvR6~v_btowKp",
  "tenant": "ab5cf8c1-0907-4db6-a510-9feba384f8a3"
}
```
//---
```
ns1-04.azure-dns.com.
ns2-04.azure-dns.net.
ns3-04.azure-dns.org.
ns4-04.azure-dns.info.
// az.precipi.com
```
//
```
az role assignment create \
  --role "Reader" \
  --assignee "cd4d61e4-ac40-4f24-b6c4-a0ca1b62b1af" \
  --scope "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/az.precipi.com"
  {
    "canDelegate": null,
    "condition": null,
    "conditionVersion": null,
    "description": null,
    "id": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/az.precipi.com/providers/Microsoft.Authorization/roleAssignments/3d125d49-7fb9-4064-91ef-e07cb0837285",
    "name": "3d125d49-7fb9-4064-91ef-e07cb0837285",
    "principalId": "91733dd6-a706-4418-a747-4238545b8adf",
    "principalType": "ServicePrincipal",
    "resourceGroup": "az.precipi.com",
    "roleDefinitionId": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7",
    "scope": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/az.precipi.com",
    "type": "Microsoft.Authorization/roleAssignments"
  }
```  
//
```
az role assignment create \
  --role "Contributor" \
  --assignee "cd4d61e4-ac40-4f24-b6c4-a0ca1b62b1af" \
  --scope "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/az.precipi.com/providers/Microsoft.Network/dnszones/az.precipi.com"
  {
  "canDelegate": null,
  "condition": null,
  "conditionVersion": null,
  "description": null,
  "id": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/az.precipi.com/providers/Microsoft.Network/dnszones/az.precipi.com/providers/Microsoft.Authorization/roleAssignments/15541b66-03f9-42b1-b8d7-7eab77b4d11c",
  "name": "15541b66-03f9-42b1-b8d7-7eab77b4d11c",
  "principalId": "91733dd6-a706-4418-a747-4238545b8adf",
  "principalType": "ServicePrincipal",
  "resourceGroup": "az.precipi.com",
  "roleDefinitionId": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
  "scope": "/subscriptions/ae36a85f-76be-4bb1-bc9e-e92d8a0af831/resourceGroups/az.precipi.com/providers/Microsoft.Network/dnszones/az.precipi.com",
  "type": "Microsoft.Authorization/roleAssignments"
}
az account show --query "tenantId"
"ab5cf8c1-0907-4db6-a510-9feba384f8a3"
az account show --query "id"
"ae36a85f-76be-4bb1-bc9e-e92d8a0af831"
```
//
```
vim azure.json

{
  "tenantId": "ab5cf8c1-0907-4db6-a510-9feba384f8a3",
  "subscriptionId": "ae36a85f-76be-4bb1-bc9e-e92d8a0af831",
  "resourceGroup": "az.precipi.com",
  "aadClientId": "cd4d61e4-ac40-4f24-b6c4-a0ca1b62b1af",
  "aadClientSecret": "fjQHBwsKcWVzRnEcJD-wLWvR6~v_btowKp"
}
kubectl create ns my-app
kubectl create secret generic azure-config-file --from-file=./azure.json -n my-app
```
//
```
kubectl create namespace ingress-external

// xxx - helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm repo add stable	https://charts.helm.sh/stable
helm repo update
// optionally install tiller
kubectl -n kube-system create serviceaccount tiller

kubectl create clusterrolebinding tiller \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:tiller

helm init --service-account tiller
// end tiller

helm upgrade -i ingress-external stable/nginx-ingress \
    --namespace ingress-external \
    --set controller.publishService.enabled=true \
    --set controller.publishService.pathOverride="ingress-external/ingress-external-nginx-ingress-controller" \
    --set controller.replicaCount=2 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux
```
//
```
vim external-dns.yaml
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: external-dns
  namespace: my-app
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRole
metadata:
  name: external-dns
rules:
- apiGroups: [""]
  resources: ["services","endpoints","pods"]
  verbs: ["get","watch","list"]
- apiGroups: ["extensions"]
  resources: ["ingresses"]
  verbs: ["get","watch","list"]
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list"]
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: external-dns-viewer
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: external-dns
subjects:
- kind: ServiceAccount
  name: external-dns
  namespace: my-app
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: my-app
spec:
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: external-dns
  template:
    metadata:
      labels:
        app: external-dns
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: us.gcr.io/k8s-artifacts-prod/external-dns/external-dns:v0.7.2
        args:
        - --source=service
        - --source=ingress
        - --domain-filter=az.precipi.com
        - --provider=azure
        - --azure-resource-group=az.precipi.com
        volumeMounts:
        - name: azure-config-file
          mountPath: /etc/kubernetes
          readOnly: true
      volumes:
      - name: azure-config-file
        secret:
          secretName: azure-config-file

kubectl -n my-app apply -f external-dns.yaml
serviceaccount/external-dns created
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
clusterrole.rbac.authorization.k8s.io/external-dns created
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/external-dns-viewer created
deployment.apps/external-dns created          
```
// cert
```
kubectl create namespace cert-manager

helm repo add jetstack https://charts.jetstack.io

helm repo update

helm upgrade -i cert-manager jetstack/cert-manager \
  --namespace cert-manager \
  --version v0.15.1 \
  --set installCRDs=true
```
//
```
vim letsencrypt.yaml
//
apiVersion: cert-manager.io/v1alpha2
kind: ClusterIssuer
metadata:
  name: letsencrypt
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: tadeugraz@outlook.com
    privateKeySecretRef:
      name: letsencrypt
    solvers:
    - http01:
        ingress:
          class: nginx

kubectl apply -f letsencrypt.yaml

kubectl describe clusterissuers letsencrypt
```
//
```
vim app-my-app.yaml


---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
      - image: r.j3ss.co/party-clippy
        name: my-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: my-app
spec:
  ports:
  - port: 80
    protocol: TCP
    targetPort: 8080
  selector:
    app: my-app
  type: ClusterIP

---
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  name: my-app
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: "letsencrypt"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  tls:
  - hosts:
    - app.my-domain.com
    secretName: tls-secret
  rules:
  - host: app.my-domain.com
    http:
      paths:
      - backend:
          serviceName: my-app
          servicePort: 80
        path: /(/|$)(.*)

kubectl -n my-app apply -f app-my-app.yaml
kubectl get certificate -n my-app
NAME         READY   SECRET       AGE
tls-secret   True    tls-secret   8m42ss
```
