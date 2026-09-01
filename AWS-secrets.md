data:
  - objectName: username    # Points to → objectAlias: username → value: "catalog"
    key: username           # Stores "catalog" under this key in K8s Secret
  - objectName: password    # Points to → objectAlias: password → value: "dYxxx"
    key: password           # Stores "dYxxx" under this key in K8s Secret

```
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: catalog-spc
  namespace: catalog
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "$SECRET_NAME"
        objectType: "secretsmanager"
        jmesPath:
          - path: username #this should be same name as the Secret key ?
            objectAlias: username #this should be same name as the Secret key ?
          - path: password
            objectAlias: password
    usePodIdentity: "true"
  secretObjects:
    - secretName: catalog-secret
      type: Opaque
      data:
        - objectName: username #this should be same name as the Secret key ?
          key: username #this should be same name as the Secret key ?
        - objectName: password
          key: password
```





## SecretProviderClass Field Mapping

| Field | Must Match | Can Be Custom | Description |
|-------|------------|---------------|-------------|
| `path` | ✅ Must match **AWS Secret JSON key** | ❌ | Extracts the value from the JSON stored in AWS Secrets Manager |
| `objectAlias` | ❌ | ✅ Any name you want | Internal alias used to reference the extracted value |
| `objectName` (in secretObjects) | ✅ Must match **`objectAlias`** | ❌ | Links to the `objectAlias` defined in `jmesPath` |
| `key` (in secretObjects) | ❌ | ✅ Any name you want | Defines the key name inside the Kubernetes Secret |



```
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/created-by: eks-workshop
    app.kubernetes.io/type: app
  name: catalog
  namespace: catalog
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/component: service
      app.kubernetes.io/instance: catalog
      app.kubernetes.io/name: catalog
  template:
    metadata:
      annotations:
        prometheus.io/path: /metrics
        prometheus.io/port: "8080"
        prometheus.io/scrape: "true"
      labels:
        app.kubernetes.io/component: service
        app.kubernetes.io/created-by: eks-workshop
        app.kubernetes.io/instance: catalog
        app.kubernetes.io/name: catalog
    spec:
      containers:
        - env:
            - name: RETAIL_CATALOG_PERSISTENCE_USER
              valueFrom:
                secretKeyRef:
                  key: username **# ← YES! This must match the "key" in secretObjects**
                  name: catalog-secret **# ← Must match the "secretName" in secretObjects**
            - name: RETAIL_CATALOG_PERSISTENCE_PASSWORD
              valueFrom:
                secretKeyRef:
                  key: password
                  name: catalog-secret
          envFrom:
            - configMapRef:
                name: catalog
          image: public.ecr.aws/aws-containers/retail-store-sample-catalog:1.2.1
          imagePullPolicy: IfNotPresent
          livenessProbe:
            httpGet:
              path: /health
              port: 8080
            initialDelaySeconds: 30
            periodSeconds: 3
          name: catalog
          ports:
            - containerPort: 8080
              name: http
              protocol: TCP
          readinessProbe:
            httpGet:
              path: /health
              port: 8080
            periodSeconds: 5
            successThreshold: 3
          resources:
            limits:
              memory: 512Mi
            requests:
              cpu: 250m
              memory: 512Mi
          securityContext:
            capabilities:
              drop:
                - ALL
            readOnlyRootFilesystem: true
            runAsNonRoot: true
            runAsUser: 1000
          volumeMounts:
            - mountPath: /etc/catalog-secret
              name: catalog-secret
              readOnly: true
            - mountPath: /tmp
              name: tmp-volume
      securityContext:
        fsGroup: 1000
      serviceAccountName: catalog
      volumes:
        - csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: catalog-spc
          name: catalog-secret
        - emptyDir:
            medium: Memory
          name: tmp-volume
```
