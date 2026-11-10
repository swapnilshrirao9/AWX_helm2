
# AWX Helm chart


# REQUIREMENT
1) Need to set up a secret with SSH (id_rsa and id_rsa.pub)
2) In ADO, we are required to set up a library and Variable for SSH
3) Secret name will be awx-ssh-private-key
4) Need to run https://dev.azure.com/Azure-Creation/Create_Backend/_build?definitionId=25
5) ArgoCD for deploying the Helm Chart of AWX

# Issue face
1) Secrets in the previous release are not working properly, so delete them; otherwise, change the namespace
This chart creates an `AWX` custom resource. An AWX Operator that serves
`awx.ansible.com/v1beta1` must be installed in the cluster before syncing.
The AWX image version is pinned to `18.0.0` in `values.yaml`.
The bundled operator chart is configured for two controller replicas and
requires pod anti-affinity by node hostname, so each replica needs a separate
schedulable node. With fewer than two eligible nodes, one replica will remain
Pending.

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
must be created separately. The AWX Operator is also a prerequisite.
