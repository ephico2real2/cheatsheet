apiVersion: v1
kind: StorageClass 
apiVersion: storage.k8s.io/v1beta1 
metadata:   
  name: hostpath
provisioner: docker.io/hostpath


---------------

#
# PersistentVolume
#
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-persistent-volume
  labels:
    type: local
spec:
  storageClassName: hostpath
  capacity:
    storage: 256Mi
  accessModes:
    - ReadWriteMany
  hostPath:
    path: /tmp
  persistentVolumeReclaimPolicy: Retain


  --------
#
# PersistentVolumeClaim
#
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-persistent-volume-claim
spec:
  storageClassName: hostpath
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 256Mi
  
