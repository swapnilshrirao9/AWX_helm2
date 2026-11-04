
# AWX Helm chart


# REQUIREMENT
1) Need to setup Secret with ssh (id_rsa and id_rsa.pub)
2) ADO we required to setup Libary and Variable for ssh
3) Secret name will be awx-ssh-private-key
4) Need to add run the https://dev.azure.com/Azure-Creation/Create_Backend/_build?definitionId=25



This chart creates an `AWX` custom resource. An AWX Operator that serves
`awx.ansible.com/v1beta1` must already be installed in the cluster before
syncing this chart.

The chart also installs the SSH ingress NetworkPolicy from the original
configuration. Disable it with `networkPolicy.enabled: false` if that policy
is not wanted in this cluster.

## SSH key Secret

The chart mounts the Secret `awx-ssh-private-key` and copies its keys into
`/root/.ssh` in the AWX task container with private-key permissions. Create
the Secret in the `awx` namespace before syncing the Application. From the
workspace root in PowerShell:

```powershell
kubectl create namespace awx
kubectl create secret generic awx-ssh-private-key `
  --namespace awx `
  --from-file=id_rsa=.\k3s-awx-setup\AWX_helm\ssh_key\id_rsa `
  --from-file=id_rsa.pub=.\k3s-awx-setup\AWX_helm\ssh_key\id_rsa.pub
```

If the namespace or Secret already exists, create/update it using your
cluster's secret-management process. Do not put the private key in
`values.yaml`, commit it to Git, or expose it in the Argo CD Application.

## Argo CD

`application.yaml` is an Argo CD `Application` manifest. Replace
`https://github.com/YOUR_ORG/YOUR_REPOSITORY.git` with the Git repository
containing this folder, push the chart, then apply `application.yaml` or paste
it into the Argo CD web portal. The source path is
`k3s-awx-setup/AWX_helm2`.

The Application creates the `awx` namespace if needed, but the SSH Secret
must be created separately. The AWX Operator is also a prerequisite and is
not installed by this chart.
