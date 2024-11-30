# CKAD Preparation Questions: Storage

## **Question 1: Create an Ephemeral Storage Setup**
1. Create a Pod with an `emptyDir` volume named `data-storage`.
2. Mount the volume inside the container at `/app/data`.
3. Use the `nginx` image for the container.
4. Verify that any data written to `/app/data` inside the container is stored in the `emptyDir` volume and does not persist after the Pod is deleted.

---

## **Question 2: Full Flow - PVC, PV, StorageClass, and Pod**
1. Create a `StorageClass` named `local-storage` with the following specifications:
   - Reclaim policy: `Retain`
   - Volume binding mode: `WaitForFirstConsumer`
2. Create a `PersistentVolume` named `local-pv` with:
   - A capacity of 1Gi.
   - Access mode `ReadWriteOnce`.
   - HostPath volume pointing to `/mnt/local-storage`.
   - StorageClass set to `local-storage`.
3. Create a `PersistentVolumeClaim` named `local-pvc` with:
   - A request for 1Gi of storage.
   - StorageClass set to `local-storage`.
4. Create a Pod named `local-storage-pod` that:
   - Uses the `nginx` image.
   - Mounts the PVC to `/usr/share/nginx/html`.
   - Writes a file named `index.html` in the mounted directory.
5. Verify:
   - The Pod successfully mounts the PVC.
   - The file is retained even after the Pod is deleted.

---

## **Question 3: Dynamic Storage with ConfigMap**
1. Create a ConfigMap named `app-config` with a file key `config.yaml` and arbitrary YAML content.
2. Create a Pod named `configmap-pod` that:
   - Uses the `nginx` image.
   - Mounts the ConfigMap as a file at `/etc/config/config.yaml`.
   - Mounts an `emptyDir` volume at `/data`.
3. Verify:
   - The file `config.yaml` is available inside the container at `/etc/config/config.yaml`.
   - Any data written to `/data` is ephemeral and is cleared after the Pod is deleted.

---

## **Question 4: Static Binding of PVC to PV**
1. Create a `PersistentVolume` named `static-pv` with:
   - A capacity of 500Mi.
   - Access mode `ReadWriteOnce`.
   - HostPath volume pointing to `/mnt/static-storage`.
2. Create a `PersistentVolumeClaim` named `static-pvc` with:
   - A request for 500Mi of storage.
   - Access mode `ReadWriteOnce`.
3. Manually bind the `static-pvc` to `static-pv`.
4. Create a Pod named `static-storage-pod` that:
   - Uses the `nginx` image.
   - Mounts the PVC to `/usr/share/nginx/html`.
   - Verifies the storage is accessible by creating a file inside the mounted path.

---

## **Question 5: Multi-Container Pod with Shared Volume**
1. Create a Pod named `shared-storage-pod` with:
   - Two containers using the `busybox` image.
   - Both containers sharing a single `emptyDir` volume mounted at `/shared-data`.
   - The first container writes a file named `shared.txt` into `/shared-data`.
   - The second container reads and prints the content of `shared.txt`.
2. Verify the Pod runs successfully and both containers can share data via the volume.

---

## **Question 6: SubPath with PersistentVolume**
1. Create a `PersistentVolume` named `subpath-pv` with:
   - A capacity of 2Gi.
   - Access mode `ReadWriteMany`.
   - HostPath volume pointing to `/mnt/subpath-storage`.
2. Create a `PersistentVolumeClaim` named `subpath-pvc` requesting 2Gi of storage.
3. Create a Pod named `subpath-pod` that:
   - Uses the `busybox` image.
   - Mounts the PVC at `/mnt/storage` using the `subPath` feature to map only the subdirectory `data`.
4. Verify the Pod runs and only the `data` directory is mounted inside the container.

---

## **Question 7: Restricting Namespace Storage Access**
1. Create a `StorageClass` named `restricted-storage` with:
   - Volume binding mode `Immediate`.
   - Reclaim policy `Delete`.
2. Create a `PersistentVolume` named `restricted-pv` with:
   - A capacity of 1Gi.
   - HostPath volume at `/mnt/restricted`.
   - StorageClass set to `restricted-storage`.
3. Create a `PersistentVolumeClaim` named `restricted-pvc` in the namespace `restricted-namespace` that:
   - Requests 1Gi of storage.
   - Uses the `restricted-storage` StorageClass.
4. Deploy a Pod in the `restricted-namespace` using the PVC and verify storage access.

---

## **Question 8: Mount Multiple Volumes in a Single Pod**
1. Create a Pod named `multi-volume-pod` that:
   - Uses the `nginx` image.
   - Mounts an `emptyDir` volume at `/temp-data`.
   - Mounts a `hostPath` volume from `/mnt/host-data` to `/host-data`.
2. Verify the Pod can write to both mounted directories and the data behaves as expected (ephemeral for `emptyDir`, persistent for `hostPath`).

