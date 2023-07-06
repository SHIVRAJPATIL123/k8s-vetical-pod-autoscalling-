# k8s-vetical-pod-autoscalling-
1)use this link for minikube setup check the helm command ad - inside it https://itnext.io/kubernetes-vertical-pods-scaling-with-vertical-pod-autoscaler-e2e5a3b8e1a9  
2)use this for eks only install vpa using the helmchart in above one  https://docs.aws.amazon.com/eks/latest/userguide/vertical-pod-autoscaler.html
........ dangling pvc persistentVolumeClaimRetentionPolicy
in statefulset.yaml file above 1.27 version use 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgresql-db
spec:
        # persistentVolumeClaimRetentionPolicy:        
        #whenDeleted: Retain
        #whenScaled: Delete    
  replicas: 1
  selector:
    matchLabels:
      app: postgresql-db
  template:
    metadata:
      labels:
        app: postgresql-db
    spec:
      containers:
        - name: postgresql-db
          image: postgres:latest
          volumeMounts:
            - name: postgresql-db-disk
              mountPath: /data
          env:
            - name: POSTGRES_PASSWORD
              value: testpassword
            - name: PGDATA
              value: /data/pgdata
          resources:
            requests:
              cpu: "100m"
              memory: "50Mi"    
      volumes:
        - name: postgresql-db-disk
          emptyDir: {}
  volumeClaimTemplates:
    - metadata:
        name: postgresql-db-disk
      spec:
        accessModes:
          - ReadWriteOnce
        resources:
          requests:
            storage: 1Gi
2)now for versions less than 1.27
create a script 
vi delete-dangling-pvcs.sh
....
#!/bin/bash

# Get a list of all PVCs in the cluster
pvc_list=$(kubectl get pvc --all-namespaces -o jsonpath='{range .items[?(@.status.phase=="Bound")]}{.metadata.namespace}/{.metadata.name}{"\n"}{end}')

# Iterate over the PVCs
for pvc in $pvc_list; do
    namespace=$(echo "$pvc" | cut -d'/' -f1)
    pvc_name=$(echo "$pvc" | cut -d'/' -f2)

    # Check if the PVC has any associated PVs
    pv_list=$(kubectl describe pvc "$pvc_name" -n "$namespace" | grep -i "Used By:")
    echo $pvc_name
    # If no associated PV found, delete the PVC
    #if [ -z "$pv_list" && "$pv_list" == *"none"* ]; then
    if [[ "$pv_list" == *"<none>"* ]]; then
        echo "Deleting PVC: $pvc"
        echo $pv_list
        kubectl delete pvc "$pvc_name"
    fi
done

.....

chmod +x delete-dangling-pvcs.sh
./delete-dangling-pvcs.sh
