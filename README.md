
# Integreatly Installation

## Overview

The purpose of this repository is to provide a set of Ansible playbooks that can be used to install a range of Red Hat middleware products on Openshift.

These products include:

* Single Sign On
* EnMasse
* Fuse iPaaS

## Prerequisites

* Ansible v2.6
* Openshift Container Platform v3.9
* Openshift CLI (OC) v3.9
* SSH Access to Openshift master(s)
* Cluster administrator permissions

## Installation Steps

### Existing Openshift v3.9 clusters

The following section demonstrates how to install each of the products listed above on an existing Openshift cluster.

#### 1. Clone installation GIT repository locally

```shell
git clone https://github.com/integr8ly/installation.git
```

#### 2. Update inventory hosts file

Prior to running the playbooks the master hostname and associated SSH username need to be added to the inventory file. The following example sets the SSH username to ```evals``` and the master hostname to ```master.evals.example.com```:

```yaml
~/installation/evals/inventories/hosts

[local:vars]
ansible_connection=local

[local]
127.0.0.1

[OSEv3:children]
master

[OSEv3:vars]
ansible_user=evals
openshift_master_config=/etc/origin/master/master-config.yaml

[master]
master.evals.example.com
```

#### 3. Run Single Sign On install playbook

```shell
cd evals/
ansible-playbook -i inventories/hosts playbooks/rhsso.yml
```

Upon completion, a new identity provider named ```rh_sso``` should be presented on the Openshift master console login screen.

Default login credentials are evals@example.com / Password1

To configure custom account credentials, simply override the rhsso role environment variables by specifying user parameters as part of the install command:

```shell
ansible-playbook -i inventories/hosts playbooks/rhsso.yml -e rhsso_evals_username=<username> -e rhsso_evals_password=<password>
```

#### 4. Run EnMasse install playbook

```shell
cd evals/
ansible-playbook -i inventories/hosts playbooks/enmasse.yml
```

Once the playbook has completed a service named `EnMasse (standard)` will be available
in the Service Catalog. This can be provisioned into your namespace to use EnMasse.

#### 5. Run Fuse iPaaS install playbook

```shell
cd evals/
ansible-playbook -i inventories/hosts playbooks/ipaas.yml
```

#### 6. Run Launcher install playbook

Before running the playbook, create a new OAuth Application on GitHub. This can
be done at https://github.com/settings/developers. Note the `Client ID` and
`Client Secret` fields of the OAuth Application, these are required by the
Launcher playbook.

The Launcher playbook also requires information about the existing SSO that was
provisioned previously. It needs to know the route of the SSO. This can be
retrieved using:

```shell
oc get route secure-sso -o jsonpath='{.spec.host}' -n rhsso
```

It also needs to know the realm to interact with. By default this would be
`openshift`. Finally it needs the credentials of a user to login as, by default
this would be the `admin` user created by the SSO playbook.

Specify the following variables in the inventory files or as `--extra-vars` when
running the playbook.

* `launcher_openshift_sso_route` - The route to the previously created SSO, without protocol.
* `launcher_openshift_sso_realm` - The realm to create resources in the SSO, this would be `openshift` by default.
* `launcher_openshift_sso_username` - Username to authenticate as, this would be the admin user by default.
* `launcher_openshift_sso_password` - Password of the user.
* `launcher_github_client_id` - The `Client ID` of the created GitHub OAuth Application.
* `launcher_github_client_secret` - The `Client Secret` of the created GitHub OAuth Application.

If using self signed certs set `launcher_sso_validate_certs` to `no/false`.
Without this, an error will be thrown similar to this:

```
fatal: [127.0.0.1]: FAILED! => {"msg": "The conditional check 'launcher_sso_auth_response.status == 200' failed. The error was: error while evaluating conditional (launcher_sso_auth_response.status == 200): 'dict object' has no attribute 'status'"}
```

Next, run the playbook.

```shell
cd evals
ansible-playbook -i inventories/hosts playbooks/launcher.yml
```

Once the playbook has completed it will print a debug message saying to update
the `Authorization callback URL` of the GitHub OAuth Application. Once this is
done the launcher setup has finished.
