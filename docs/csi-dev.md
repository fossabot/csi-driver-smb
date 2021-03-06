# Azure file CSI driver development guide

## How to build this project
 - Clone repo
```
$ mkdir -p $GOPATH/src/sigs.k8s.io/
$ git clone https://github.com/csi-driver/csi-driver-smb $GOPATH/src/github.com/csi-driver/csi-driver-smb
```

 - Build CSI driver
```
$ cd $GOPATH/src/github.com/csi-driver/csi-driver-smb
$ make
```

 - Run verification test before submitting code
```
$ make verify
```

## How to test CSI driver in local environment

Install `csc` tool according to https://github.com/rexray/gocsi/tree/master/csc
```
$ mkdir -p $GOPATH/src/github.com
$ cd $GOPATH/src/github.com
$ git clone https://github.com/rexray/gocsi.git
$ cd rexray/gocsi/csc
$ make build
```

#### Start CSI driver locally
```
$ cd $GOPATH/src/github.com/csi-driver/csi-driver-smb
$ ./_output/smbplugin --endpoint tcp://127.0.0.1:10000 --nodeid CSINode -v=5 &
```

#### 1. Get plugin info
```
$ csc identity plugin-info --endpoint tcp://127.0.0.1:10000
"smb.csi.k8s.io"    "v0.1.0"
```

#### 2. Mount an smb volume to a user specified directory
```
$ mkdir ~/testmount
$ csc node publish --endpoint tcp://127.0.0.1:10000 --cap 1,block --target-path ~/testmount CSIVolumeID
#f5713de20cde511e8ba4900#pvc-file-dynamic-8ff5d05a-f47c-11e8-9c3a-000d3a00df41
```

#### 3. Unmount smb volume
```
$ csc node unpublish --endpoint tcp://127.0.0.1:10000 --target-path ~/testmount CSIVolumeID
CSIVolumeID
```

#### 4. Validate volume capabilities
```
$ csc controller validate-volume-capabilities --endpoint tcp://127.0.0.1:10000 --cap 1,block CSIVolumeID
CSIVolumeID  true
```

#### 5. Get NodeID
```
$ csc node get-info --endpoint tcp://127.0.0.1:10000
CSINode
```

## How to test CSI driver in a Kubernetes cluster

 - Build continer image and push image to dockerhub
```
# run `docker login` first
export REGISTRY=<dockerhub-alias>
make container
make push-latest
```

 - Replace `andyzhangx/smb-csi:latest` in [`csi-smb-controller.yaml`](https://github.com/csi-driver/csi-driver-smb/blob/master/deploy/csi-smb-controller.yaml) and [`csi-smb-node.yaml`](https://github.com/csi-driver/csi-driver-smb/blob/master/deploy/csi-smb-node.yaml) with above dockerhub image urls and then follow [install CSI driver master version](https://github.com/csi-driver/csi-driver-smb/blob/master/docs/install-csi-driver-master.md)
 ```
wget -O csi-smb-controller.yaml https://raw.githubusercontent.com/csi-driver/csi-driver-smb/master/deploy/csi-smb-controller.yaml
# edit csi-smb-controller.yaml
kubectl apply -f csi-smb-controller.yaml

wget -O csi-smb-node.yaml https://raw.githubusercontent.com/csi-driver/csi-driver-smb/master/deploy/csi-smb-node.yaml
# edit csi-smb-node.yaml
kubectl apply -f csi-smb-node.yaml
 ```
