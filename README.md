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
