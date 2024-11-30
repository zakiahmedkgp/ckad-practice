# Storage (StorageClass, PVC, PV)

basic persistent voume:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
    - nfsvers=4.1
  nfs:
    path: /tmp
    server: 172.17.0.2
```

all attributes:
```yaml
apiVersion: v1  # The API version for the PersistentVolume resource
kind: PersistentVolume  # Specifies that this resource is a PersistentVolume
metadata:
  name: my-pv  # Name of the PersistentVolume object
  labels:  # Labels attached to the PV for identification and categorization
    app: my-app
    environment: production
spec:
  capacity:  # Specifies the storage capacity of the PersistentVolume
    storage: 5Gi  # The amount of storage the volume will provide (5Gi in this case)
  volumeMode: Filesystem  # Defines the volume mode, can be "Filesystem" or "Block"
  accessModes:  # Defines how the PersistentVolume can be accessed
    - ReadWriteOnce  # The volume can be mounted as read-write by a single node
    # Other possible values: ReadOnlyMany, ReadWriteMany
  persistentVolumeReclaimPolicy: Retain  # What happens to the PV after it's released from a claim
    # Other possible values: Delete, Recycle
  storageClassName: standard  # The name of the StorageClass that defines the provisioner and other parameters
  volumeAttributes:  # Custom attributes related to the volume (used by specific provisioners)
    # For example, you could add cloud provider-specific attributes here
    type: gp2  # Volume type (e.g., for AWS EBS)
  gcePersistentDisk:  # Example of volume source using Google Cloud Persistent Disk
    pdName: my-disk  # The name of the persistent disk
    fsType: ext4  # The filesystem type to mount (e.g., ext4, xfs)
  awsElasticBlockStore:  # Example of volume source using AWS EBS
    volumeID: vol-xxxxxxxx  # The ID of the EBS volume
    fsType: ext4  # The filesystem type to mount
  cinder:  # Example of volume source using OpenStack Cinder
    volumeID: volume-xxxxxxxx  # The ID of the Cinder volume
    fsType: ext4  # Filesystem type to mount
  hostPath:  # Example of volume source using a hostPath (local node path)
    path: /mnt/data  # Path on the node to use for the volume
    type: Directory  # The type of the hostPath (e.g., Directory, File, Socket)
  nfs:  # Example of volume source using NFS
    path: /exported/path  # The path to the shared NFS directory
    server: 192.168.1.100  # The NFS server IP or DNS name
  flexVolume:  # Example of volume source using a FlexVolume driver
    driver: example.com/driver  # The FlexVolume driver
    options:  # Any specific options required by the driver
      option1: value1
      option2: value2
  iscsi:  # Example of volume source using iSCSI
    targetPortal: 192.168.1.100:3260  # The iSCSI target portal (IP and port)
    iqn: iqn.1993-08.org.debian:01:9d30a7b45971  # iSCSI Qualified Name (IQN)
    lun: 0  # Logical Unit Number
    fsType: ext4  # The filesystem type to mount
    readOnly: false  # Whether the volume should be mounted read-only
  claimRef:  # Reference to a PersistentVolumeClaim (PVC) that binds to this PV
    name: my-pvc  # The name of the PVC
    namespace: default  # The namespace of the PVC
    uid: 12345678-90ab-cdef-ghij-klmnopqrstuv  # The unique identifier (UID) of the PVC
    resourceVersion: "1"  # The version of the PVC reference
  nodeAffinity:  # Affinity rules that restrict where the PersistentVolume can be used
    required:  # Specifies required node conditions
      nodeSelectorTerms:
        - matchExpressions:
            - key: kubernetes.io/hostname
              operator: In
              values:
                - node1  # The PV should only be bound to a node with this hostname
  volumeMode: Filesystem  # The mode of the volume, either 'Filesystem' or 'Block'
  mountOptions:  # Additional mount options
    - noatime  # Mount option to prevent updating access time of files
  blockCleanerCommand:  # Command used for cleaning the block storage
    - "/scripts/clean.sh"  # A script that runs during volume cleanup
  fsType: ext4  # Filesystem type for the volume

```


4 attributes are important which should be remembered

```yaml
spec:
	capacity:
		storage: 5Gi
	accessModes:
	- ReadWriteOnce
	
```

Practice

```yaml
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: standard
  capacity:
    storage: 10Gi
  persistentVolumeReclaimPolicy: Retain
  volumeMode: Filesystem
  claimRef:
    name: foo
    namespace: foo-ns
```


# Ephemeral volumes

short lived volumes. examples: emptyDir, configMap, secret are some ofthe local ephemral storage

practice:

```yaml

```

# StorageClasses

basic storage class

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: low-latency
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
provisioner: csi-driver.example-vendor.example
reclaimPolicy: Retain # default value is Delete
allowVolumeExpansion: true
mountOptions:
  - discard # this might enable UNMAP / TRIM at the block storage layer
volumeBindingMode: WaitForFirstConsumer
parameters:
  guaranteedReadWriteLatency: "true" # provider-specific

```

Important attributes for storage classes 

```yaml
provisioner: ""
reclaimPolicy: Retain|Reclaim|Delete 
allowVolumeExpansion: true|false
volumeBindingMode: Immediate|WaitForFirstConsumer
```

# PersistentVolumeClaim

basic pvc 

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
    matchExpressions:
      - {key: environment, operator: In, values: [dev]}
```

all attributes

```yaml
apiVersion: v1  # The API version for the PersistentVolumeClaim resource
kind: PersistentVolumeClaim  # Specifies that this resource is a PersistentVolumeClaim
metadata:
  name: my-pvc  # The name of the PersistentVolumeClaim object
  namespace: default  # The namespace in which the PVC is created (default if not specified)
  labels:  # Optional: Labels to categorize and organize the PVC
    app: my-app
    environment: production
  annotations:  # Optional: Metadata for additional information or configuration hints
    volume.beta.kubernetes.io/storage-provisioner: kubernetes.io/aws-ebs

spec:  # The specification for the PVC
  accessModes:  # The access mode for the PersistentVolume
    - ReadWriteOnce  # Example: Allows read-write by a single node
    # Other possible values: ReadOnlyMany, ReadWriteMany
  resources:  # Specifies the storage requirements for the claim
    requests:
      storage: 10Gi  # The amount of storage requested (e.g., 10Gi)
  storageClassName: standard  # Specifies the StorageClass to use for dynamic provisioning
  volumeName: my-pv  # Optional: Binds this PVC to a specific PersistentVolume by name
  selector:  # Optional: Specifies conditions that a PV must satisfy to be bound to this PVC
    matchLabels:  # Select PVs with specific labels
      environment: production
    matchExpressions:  # More advanced selection logic
      - key: storage-tier
        operator: In  # Operators can be In, NotIn, Exists, DoesNotExist
        values:
          - high-performance
  dataSource:  # Optional: Used for volume cloning or snapshots (requires support from the provisioner)
    name: my-snapshot  # The name of the source snapshot or volume
    kind: VolumeSnapshot  # The type of the data source (VolumeSnapshot or PersistentVolumeClaim)
    apiGroup: snapshot.storage.k8s.io  # API group for the data source (if applicable)

```

Important attributes 

```yaml
spec: 
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 8Gi
  storageClassName: standard
  volumeName: my-pv
  
```

# VolumeMounts inside a Pod

all attributes

```yaml
volumeMounts:
  - name: my-volume  # (Required) Name of the volume from the 'volumes' section
    mountPath: /data  # (Required) Path inside the container where the volume is mounted
    subPath: my-subdirectory  # (Optional) Subdirectory of the volume to mount
    readOnly: true  # (Optional) Mounts the volume as read-only if set to true
    mountPropagation: Bidirectional  # (Optional) Controls mount propagation; options are 'None', 'HostToContainer', 'Bidirectional'
    subPathExpr: $(POD_NAME)  # (Optional) Subdirectory path with variable expansion
    mountOptions:  # (Optional) List of mount options, such as 'noatime'
      - noatime
    volumeMountGroup: 1000  # (Optional) GID applied to the volume; requires fsGroupChangePolicy to be 'OnRootMismatch' or 'Always'

```

# Volumes for a pod

all attributes 

```yaml
volumes:
  - name: my-volume  # (Required) Unique name for the volume within the Pod
    emptyDir:  # Ephemeral storage that is initially empty
      medium: ""  # (Optional) Storage medium; "" (default) or "Memory"
      sizeLimit: 1Gi  # (Optional) Total size limit for the volume
    hostPath:  # Mounts a file or directory from the host node's filesystem
      path: /data  # (Required) Path on the host
      type: Directory  # (Optional) Type of the host path
    persistentVolumeClaim:  # References a PersistentVolumeClaim in the same namespace
      claimName: my-pvc  # (Required) Name of the PVC
      readOnly: false  # (Optional) Mounts the volume as read-only if true
    configMap:  # Populates the volume with data from a ConfigMap
      name: my-config  # (Required) Name of the ConfigMap
      items:  # (Optional) Specific key-to-path mappings
        - key: config.yaml
          path: config.yaml
      defaultMode: 420  # (Optional) Default file permissions
      optional: false  # (Optional) Specifies if the ConfigMap must be present
    secret:  # Populates the volume with data from a Secret
      secretName: my-secret  # (Required) Name of the Secret
      items:  # (Optional) Specific key-to-path mappings
        - key: password
          path: password.txt
      defaultMode: 420  # (Optional) Default file permissions
      optional: false  # (Optional) Specifies if the Secret must be present
```


