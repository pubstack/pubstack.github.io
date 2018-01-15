---
layout: post
title: "Automating Undercloud backups and a Mistral introduction for creating workbooks, workflows and actions"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
commentIssueId: 40
---

The goal of this developer documentation is to address the automated process
of backing up a TripleO Undercloud and to give developers a complete description
about how to integrate Mistral workbooks, workflows and actions into the Python
TripleO client.

This tutorial will be divided into several sections:

1. Introduction and prerequisites
2. Undercloud backups
3. Creating a new OpenStack CLI command in python-tripleoclient (openstack
   undercloud backup).
4. Creating Mistral workflows for the new python-tripleoclient CLI command.
5. Give support for new Mistral environment variables when installing the
   undercloud.
6. Show how to test locally the changes in python-tripleoclient and
   tripleo-common.
7. Give elevated privileges to specific Mistral actions that need to run with
   elevated privileges.
8. Debugging actions
9. Unit tests
10. Why all previous sections are related to Upgrades?

## 1. Introduction and prerequisites

Let's assume you have a TripleO development environment healthy and working
properly. All the commands and customization we are going to run will run in
the Undercloud, as usual logged in as the stack user and having sourced the
stackrc file.

Then let's proceed by cloning the repositories we are going to work with in a
temporary folder:

```
mkdir dev-docs
cd dev-docs
git clone https://github.com/openstack/python-tripleoclient
git clone https://github.com/openstack/tripleo-common
git clone https://github.com/openstack/instack-undercloud
```
* **python-tripleoclient:** Will define the OpenStack CLI commands.
* **tripleo-common:** Will have the Mistral logic.
* **instack-undercloud:** Allows to update and create mistral
environments to store configuration details needed when executing Mistral workflows.

## 2. Undercloud backups

Most of the Undercloud back procedure is available in the
[TripleO official documentation site](https://docs.openstack.org/tripleo-docs/latest/install/post_deployment/backup_restore_undercloud.html).

We will focus on the automation of backing up the resources required to restore
the Undercloud in case of a failed upgrade.

* All MariaDB databases on the undercloud node
* MariaDB configuration file on undercloud (so we can restore databases
  accurately)
* All glance image data in /var/lib/glance/images
* All swift data in /srv/node
* All data in stack users home directory

For doing this we need to be able to:

* Connect to the database server as root.
* Dump all databases to file.
* Create a filesystem backup of several folders (and be able to access folders
  with restricted access).
* Upload this backup to a swift container to be able to get it from the TripleO
  web UI.

## 3. Creating a new OpenStack CLI command in python-tripleoclient (openstack undercloud backup).

The first action needed is to be able to create a new CLI command for the
OpenStack client.  In this case, we are going to implement the **openstack
undercloud backup** command.

```
cd dev-docs
cd python-tripleoclient
```

Let's list the files inside this folder:

```
[stack@undercloud python-tripleoclient]$ ls
AUTHORS           doc                            setup.py
babel.cfg         LICENSE                        test-requirements.txt
bindep.txt        zuul.d                         tools
build             README.rst                     tox.ini
ChangeLog         releasenotes                   tripleoclient
config-generator  requirements.txt               
CONTRIBUTING.rst  setup.cfg
```

Once inside the **python-tripleoclient** folder we need to check the following
file:

**setup.cfg:** This file defines all the CLI commands for the Python TripleO
client. Specifically, we will need at the end of this file our new command
definition:

```
undercloud_backup = tripleoclient.v1.undercloud_backup:BackupUndercloud
```

This means that we have a new command defined as **undercloud backup** that
will instantiate the **BackupUndercloud** class defined in the file
**tripleoclient/v1/undercloud_backup.py**

For further details related to this class definition please go to the
[gerrit review](https://review.openstack.org/#/c/466213).

Now, having our class defined we can call other methods to invoke Mistral in
this way:

```
clients = self.app.client_manager

files_to_backup = ','.join(list(set(parsed_args.add_files_to_backup)))

workflow_input = {
    "sources_path": files_to_backup
}
output = undercloud_backup.prepare(clients, workflow_input)
```
So forth, we will call the **undercloud_backup.prepare** method defined
in the file **tripleoclient/workflows/undercloud_backup.py** wich will
call the Mistral workflow:

```
def prepare(clients, workflow_input):
    workflow_client = clients.workflow_engine
    tripleoclients = clients.tripleoclient
    with tripleoclients.messaging_websocket() as ws:
        execution = base.start_workflow(
            workflow_client,
            'tripleo.undercloud_backup.v1.prepare_environment',
            workflow_input=workflow_input
        )
        for payload in base.wait_for_messages(workflow_client, ws, execution):
            if 'message' in payload:
                return payload['message']
```
In this case, we will create a loop within the tripleoclient and wait until we receive
a message from the Mistral workflow **tripleo.undercloud_backup.v1.prepare_environment**
that indicates if the invoked workflow ended correctly.

## 4. Creating Mistral workflows for the new python-tripleoclient CLI command.

The next step is to define the
**tripleo.undercloud_backup.v1.prepare_environment** Mistral workflow, all the
Mistral workbooks, workflows and actions will be defined in the
**tripleo-common** repository.

Let's go inside **tripleo-common**

```
cd dev-docs
cd tripleo-common
```

And see it's conent:

```
[stack@undercloud tripleo-common]$ ls
AUTHORS           doc                README.rst        test-requirements.txt
babel.cfg         HACKING.rst        releasenotes      tools
build             healthcheck        requirements.txt  tox.ini
ChangeLog         heat_docker_agent  scripts           tripleo_common
container-images  image-yaml         setup.cfg         undercloud_heat_plugins
contrib           LICENSE            setup.py          workbooks
CONTRIBUTING.rst  playbooks          sudoers           zuul.d
```

Again we need to check the following file:

setup.cfg: This file defines all the Mistral actions we can call.
Specifically, we will need at the end of this file our new actions:


```
tripleo.undercloud.get_free_space = tripleo_common.actions.undercloud:GetFreeSpace
tripleo.undercloud.create_backup_dir = tripleo_common.actions.undercloud:CreateBackupDir
tripleo.undercloud.create_database_backup = tripleo_common.actions.undercloud:CreateDatabaseBackup
tripleo.undercloud.create_file_system_backup = tripleo_common.actions.undercloud:CreateFileSystemBackup
tripleo.undercloud.upload_backup_to_swift = tripleo_common.actions.undercloud:UploadUndercloudBackupToSwift
```

### 4.1. Action definition

Let's take the first action to describe it's definition,
**tripleo.undercloud.get_free_space = tripleo_common.actions.undercloud:GetFreeSpace**

We have defined the action named as **tripleo.undercloud.get_free_space** which
will instantiate the class **GetFreeSpace** defined in the file
**tripleo_common/actions/undercloud.py** file.

If we open **tripleo_common/actions/undercloud.py** we can see the class definition as:

```
class GetFreeSpace(base.Action):
    """Get the Undercloud free space for the backup.

       The default path to check will be /tmp and the default minimum size will
       be 10240 MB (10GB).
    """

    def __init__(self, min_space=10240):
        self.min_space = min_space

    def run(self, context):
        temp_path = tempfile.gettempdir()
        min_space = self.min_space
        while not os.path.isdir(temp_path):
            head, tail = os.path.split(temp_path)
            temp_path = head
        available_space = (
            (os.statvfs(temp_path).f_frsize * os.statvfs(temp_path).f_bavail) /
            (1024 * 1024))
        if (available_space < min_space):
            msg = "There is no enough space, avail. - %s MB" \
                  % str(available_space)
            return actions.Result(error={'msg': msg})
        else:
            msg = "There is enough space, avail. - %s MB" \
                  % str(available_space)
            return actions.Result(data={'msg': msg})
```

In this specific case this class will check if there is enough space to perform
the backup.  Later we will be able to inkove action as

```
mistral run-action tripleo.undercloud.get_free_space
```

or use it workbooks.


### 4.2. Workflow definition.

Once we have defined all our new actions, we need to orchestrate them in order
to have a fully working Mistral workflow.

All **tripleo-comon** workbooks are defined in the workbooks folder.

In the next example we have a workbook definition
with all actions inside it, in this case we put in the example
the first workflow with all the tasks involved.


```
---
version: '2.0'
name: tripleo.undercloud_backup.v1
description: TripleO Undercloud backup workflows

workflows:

  prepare_environment:
    description: >
      This workflow will prepare the Undercloud to run the database backup
    tags:
      - tripleo-common-managed
    input:
      - queue_name: tripleo
    tasks:
      # Action to know if there is enough available space
      # to run the Undercloud backup
      get_free_space:
        action: tripleo.undercloud.get_free_space
        publish:
            status: <% task().result %>
            free_space: <% task().result %>
        on-success: send_message
        on-error: send_message
        publish-on-error:
          status: FAILED
          message: <% task().result %>


      # Sending a message that the folder to create the backup was
      # created succesfully
      send_message:
        action: zaqar.queue_post
        retry: count=5 delay=1
        input:
          queue_name: <% $.queue_name %>
          messages:
            body:
              type: tripleo.undercloud_backup.v1.launch
              payload:
                status: <% $.status %>
                execution: <% execution() %>
                message: <% $.get('message', '') %>
        on-success:
          - fail: <% $.get('status') = "FAILED" %>
```

The workflow its self explanatory, the only not so clear part might be the last
one as the workflow uses an action to send a message stating that the workflow
ended correctly.  Passing as the message the output of the previous task, in
this case the result of the **create_backup_dir**.

## 5. Give support for new Mistral environment variables when installing the undercloud.

Sometimes is needed to use additional values inside a Mistral task. For example,
if we need to create a dump of a database we might need another that the
Mistral user credentials for authentication purposes.

Initially when the Undercloud is installed it's
created a Mistral environment called
**tripleo.undercloud-config**.
This environment variable will have all required configuration details that we
can get from Mistral. This is defined in the **instack-undercloud** repository.

Let's get into the repository and check the content of the file
**instack_undercloud/undercloud.py**.

This file defines a set of methods to interact with the Undercloud,
specifically the method called **_create_mistral_config_environment** allows to
configure additional environment variables when installing the Undercloud.

For additional testing, you can use the
[Python snippet](https://gist.github.com/ccamacho/354f798102710d165c1f6167eb533caf#file-mistral_client_snippet-py)
to call Mistral client from the Undercloud node
available in gist.github.com.

## 6. Show how to test locally the changes in python-tripleoclient and tripleo-common.

If it's needed a local test of a change in python-tripleoclient or
tripleo-common, the following procedures allow to test it locally.

For a change in **python-tripleoclient**, assuming you already have downloaded
the change you want to test, execute:

```
cd python-tripleoclient
sudo rm -Rf /usr/lib/python2.7/site-packages/tripleoclient*
sudo rm -Rf /usr/lib/python2.7/site-packages/python_tripleoclient*
sudo python setup.py clean --all install
```

For a change in **tripleo-common**, assuming you already have downloaded the
change you want to test, execute:

```
cd tripleo-common
sudo rm -Rf /usr/lib/python2.7/site-packages/tripleo_common*
sudo python setup.py clean --all install
sudo cp /usr/share/tripleo-common/sudoers /etc/sudoers.d/tripleo-common
# this loads the actions via entrypoints
sudo mistral-db-manage --config-file /etc/mistral/mistral.conf populate
# make sure the new actions got loaded
mistral action-list | grep tripleo
for workbook in workbooks/*.yaml; do
    mistral workbook-create $workbook
done

for workbook in workbooks/*.yaml; do
    mistral workbook-update $workbook
done
sudo systemctl restart openstack-mistral-executor
sudo systemctl restart openstack-mistral-engine
```

If we want to execute a Mistral action or a Mistral workflow you can execute:

Testing Mistral actions independently:

```
mistral run-action tripleo.undercloud.get_free_space

```

Testing a Mistral workflow independently:

```
mistral execution-create tripleo.undercloud_backup.v1.prepare_environment
```

## 7. Give elevated privileges to specific Mistral actions that need to run with elevated privileges.

Sometimes its is not possible to execute some restricted actions from the
Mistral user, for example, when creating the Undercloud backup we won't be able
to access the **/home/stack/** folder to create a tarball of it.  For this
cases it's possible to execute elevates actions from the Mistral user:

This is the content of the **sudoers** in the root of the **tripleo-common**
repository at the time of the creatino of this guide.

```
Defaults!/usr/bin/run-validation !requiretty
Defaults:validations !requiretty
Defaults:mistral !requiretty
mistral ALL = (validations) NOPASSWD:SETENV: /usr/bin/run-validation
mistral ALL = NOPASSWD: /usr/bin/chown -h validations\: /tmp/validations_identity_[A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_], \
        /usr/bin/chown validations\: /tmp/validations_identity_[A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_], \
        !/usr/bin/chown /tmp/validations_identity_* *, !/usr/bin/chown /tmp/validations_identity_*..*
mistral ALL = NOPASSWD: /usr/bin/rm -f /tmp/validations_identity_[A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_][A-Za-z0-9_], \
        !/usr/bin/rm /tmp/validations_identity_* *, !/usr/bin/rm /tmp/validations_identity_*..*
mistral ALL = NOPASSWD: /bin/nova-manage cell_v2 discover_hosts *
mistral ALL = NOPASSWD: /usr/bin/tar --ignore-failed-read -C / -cf /tmp/undercloud-backup-*.tar *
mistral ALL = NOPASSWD: /usr/bin/chown mistral. /tmp/undercloud-backup-*/filesystem-*.tar
validations ALL = NOPASSWD: ALL
```

Here you can grant permissions for specific tasks in when executing Mistral
workflows from **tripleo-common**

## 7. Debugging actions.

Let's assume the action is written, added to setup.cfg but not appeared.
Firstly, check if action was added by ``sudo mistral-db-manage populate``. Run

```
mistral action-list -f value -c Name | grep -e '^tripleo.undercloud'
```

If you don't see your actions check output of ``sudo mistral-db-manage populate`` as

```
sudo mistral-db-manage populate 2>&1| grep ERROR | less
```
The following output may indicate issues in code. Simply fix code.
```
2018-01-01:00:59.730 7218 ERROR stevedore.extension [-] Could not load 'tripleo.undercloud.get_free_space': unexpected indent (undercloud.py, line 40):   File "/usr/lib/python2.7/site-packages/tripleo_common/actions/undercloud.py", line 40
```
Execute single action, execute workflow from workbook to make sure it works as
designed.

## 8. Unit tests
Writing Unit test is essential instrument of Software Developer. Unit tests are
much faster that running Workflow itself. So, let's write unit tests for written
action. Let's add **tripleo_common/tests/actions/test_undercloud.py** file
with the following content in **tripleo-comon** repositiry.

```
import mock

from tripleo_common.actions import undercloud
from tripleo_common.tests import base


class GetFreeSpaceTest(base.TestCase):
    def setUp(self):
        super(GetFreeSpaceTest, self).setUp()
        self.temp_dir = "/tmp"

    @mock.patch('tempfile.gettempdir')
    @mock.patch("os.path.isdir")
    @mock.patch("os.statvfs")
    def test_run_false(self, mock_statvfs, mock_isdir, mock_gettempdir):
        mock_gettempdir.return_value = self.temp_dir
        mock_isdir.return_value = True
        mock_statvfs.return_value = mock.MagicMock(
            spec_set=['f_frsize', 'f_bavail'],
            f_frsize=4096, f_bavail=1024)
        action = undercloud.GetFreeSpace()
        action_result = action.run(context={})
        mock_gettempdir.assert_called()
        mock_isdir.assert_called()
        mock_statvfs.assert_called()
        self.assertEqual("There is no enough space, avail. - 4 MB",
                         action_result.error['msg'])

    @mock.patch('tempfile.gettempdir')
    @mock.patch("os.path.isdir")
    @mock.patch("os.statvfs")
    def test_run_true(self, mock_statvfs, mock_isdir, mock_gettempdir):
        mock_gettempdir.return_value = self.temp_dir
        mock_isdir.return_value = True
        mock_statvfs.return_value = mock.MagicMock(
            spec_set=['f_frsize', 'f_bavail'],
            f_frsize=4096, f_bavail=10240000)
        action = undercloud.GetFreeSpace()
        action_result = action.run(context={})
        mock_gettempdir.assert_called()
        mock_isdir.assert_called()
        mock_statvfs.assert_called()
        self.assertEqual("There is enough space, avail. - 40000 MB",
                         action_result.data['msg'])
```
Run
```
tox -epy27
```
to see any unit tests errors.

## 8. Why all previous sections are related to Upgrades?

* Undercloud backups are an important step before runing an Upgrade.
* Writing developer docs will help people to create and develope new features.

## 9. References

* http://www.dougalmatthews.com/2016/Sep/21/debugging-mistral-in-tripleo/
* http://blog.johnlikesopenstack.com/2017/06/accessing-mistral-environment-in-cli.html
* http://hardysteven.blogspot.com.es/2017/03/developing-mistral-workflows-for-tripleo.html

