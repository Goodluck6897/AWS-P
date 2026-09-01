data:
  - objectName: username    # Points to → objectAlias: username → value: "catalog"
    key: username           # Stores "catalog" under this key in K8s Secret
  - objectName: password    # Points to → objectAlias: password → value: "dYxxx"
    key: password           # Stores "dYxxx" under this key in K8s Secret


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
          - path: username
            objectAlias: username
          - path: password
            objectAlias: password
    usePodIdentity: "true"
  secretObjects:
    - secretName: catalog-secret
      type: Opaque
      data:
        - objectName: username
          key: username
        - objectName: password
          key: password
