# Playbook Integrity Operator
Playbook integrity operator is a Kubernetes operator for managing integrity of playbooks and roles executed in Ansible Controller.

A custom resource which is installed & managed by this operator is `PlaybookIntegrity`. You can define your playbook integrity configuration like eanble/disable flag and verification key under the `spec`. Once `PlaybookIntegrity` resource is create/updated on a cluster, then the operator will sync the settings to Ansible Controller, and will trigger `ProjectUpdate` if necessarry. After that, the latest integrity check result will be propagated into `status` in the PlaybookIntegrity resource, so you can get integrity check result without directly accessing Ansible Controller.

## Installation

You can install this operator by just one make command.

```
$ make deploy
```

If you would like to use your own operator image instead of the default one, or if you install this in the different namespace from the default `awx-playbook-integrity-operator`, you can configure those like this.

```
$ make deploy IMG=<your-registry>/<image-name> NAMESPACE=<your-target-namespace>
```

After installing the operator, by creating a authentication secret to access your Ansible Controller in a namespace `awx-playbook-integrity-operator` (by default), all installation steps are done.

## Usage

To configure playbook integrity settings, you can set the following fields in PlaybookIntegrity resource.

```yaml
apiVersion: tower.ansible.com/v1alpha1
kind: PlaybookIntegrity
metadata:
  name: demo-playbook-integrity
spec:
  tower_auth_secret: towerauthsecret               # a secret name to access ansible controller
  project_name: Demo Playbook Integrity Project    # an Ansible Project name you are configuring 
  enabled: true                                    # a flag to enable/disable playbook integrity check
  public_key: <base64-encoded-public-key>          # a base64 encoded public key for playbook verification
  signature_type: <gpg/x509/sigstore>              # a signature type (gpg/x509/sigstore)
```

When you create the PlaybookIntegrity, a Kubernetes Job for configuring the project will be executed immediately.

Then, a CronJob for checking the configuration and get the integrity check result will be run periodically.

The updated PlaybookIntegrity status will be something like this.

```yaml
apiVersion: tower.ansible.com/v1alpha1
kind: PlaybookIntegrity
metadata:
  name: demo-playbook-integrity
spec: 
  ...
status:
  playbookIntegrityResult:
    elapsed: "6.421"
    enabled: "true"
    finished: "2021-12-01T11:12:16.441750Z"
    latest_result:
    - error: ""
      playbook: hosts.yml
      verified: true
    - error: ""
      playbook: site.yml
      verified: true
    started: "2021-12-01T11:12:10.020986Z"
    status: successful
```

If there was an verification error while playbook integrity check, it should be reported in the `error` fields, and the result should be `verified: false`.
