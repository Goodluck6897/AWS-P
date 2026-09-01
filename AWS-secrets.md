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




3. objectName: username (in secretObjects) — Must it match the Secret key?
NO ❌ — It must match the objectAlias from jmesPath, not the AWS secret key.

So if you used objectAlias: myuser, then you'd need objectName: myuser here.

4. key: username (in secretObjects) — Must it match the Secret key?
NO ❌ — This can be any name you choose. It defines the key name inside the Kubernetes Secret.

