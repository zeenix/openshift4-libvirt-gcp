# OpenShift 4.0 libvirt on GCP

Run an [OpenShift 4.0](https://github.com/openshift/installer) cluster on a single GCP instance using nested virtualization.

## Create a cluster

First, launch a VM. This assumes images are built in the desired region.

```shell
$ gcloud compute instances create $INSTANCE \
  --zone us-east1-c --image-family openshift4-libvirt \
  --machine-type n1-standard-8 --min-cpu-platform "Intel Haswell" \
  --boot-disk-type pd-ssd --boot-disk-size 256GB \
  --metadata-from-file openshift-pull-secret=openshift-pull-secret.json
```

Next, do some post-provisioning. *Only do this once during the life of the VM.*

```shell
$ gcloud compute scp --recurse tools $INSTANCE:~/tools
$ gcloud compute ssh $INSTANCE --command '~/tools/post-provision.sh'
```

Now you can run the installer.

```shell
$ gcloud compute ssh $INSTANCE
$ ~/tools/create-cluster.sh nested
```

The `create-cluster.sh` helper is for convenience. Feel free to run the installer directly.

## Building images

Images are built with [Packer](https://www.packer.io). Override variables as necessary.

### Source image

The source image is `centos-7` with [nested virtualization enabled](https://cloud.google.com/compute/docs/instances/enable-nested-virtualization-vm-instances#restrictions).

```shell
$ packer build openshift4-libvirt-source.json
```

### Provisioned image

The provisioned image implements all the [OpenShift libvirt HOWTO](https://github.com/openshift/installer/blob/master/docs/dev/libvirt-howto.md) requirements.

```shell
$ packer build openshift4-libvirt.json
```