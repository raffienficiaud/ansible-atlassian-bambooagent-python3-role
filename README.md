Atlassian Bamboo agent Python3 role
===================================

Installs a specific version of `python3` (from an installer or Debian packages) into a remote Atlassian Bamboo build agent and declares its capabilities to the agent.

Requirements
------------

This role needs to declare the `python3` capabilities after installation, and relies on the format of the role `raffienficiaud.atlassian-bambooagent-role` for declaring those.

Role Variables
--------------

| variable | default | meaning |
|----------|---------|---------|
|python3_version| **required**| the exact version of `python3`|
|python3_installer| **required** | the installer file (OSX, Windows)|
|bamboo_capabilities|**required**| capabilities that will be populated by the role. |

### Capabilities declared by the role

| capability | value |
|----------|---------|
|`system.builder.command.python${version.major}.${version.minor}`| location of the `python3` binary of the given major/minor version|

### Variables and tags

* Ubuntu: `apt-get install` is used on this platform, with the version major and minor coming from the `python3_version` variable. After the installation
  `python3` location is taken from `/usr/bin/`
* OSX: the provided installer from the variable `python3_installer` is used. This currently should be a `.pkg` file, `python3` location is looked for using the `which python3` shell command.

The tags in use by the role are:

* `capability`: task filling a capability
* `python3`: all tasks of the role
* `installation`: task installing new software

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------

* In the inventory file for the `osx` group, we can have configure the `python3` install like the following
  ```yaml
  bamboo_python3_version:
    major: 3
    minor: 4
    patch: 3

  bamboo_python3_installer:
    file: "/path/to/installer-pkg/python-{{bamboo_python3_version.major}}.{{bamboo_python3_version.minor}}.{{bamboo_python3_version.patch}}-macosx10.6.pkg"
  ```
* In the inventory for `linux` group, we can just have this:
  ```yaml
  bamboo_python3_version:
    major: 3
    minor: 4
    patch: 3
  ```

  although we will not be ensured that the `patch` part will be honoured properly by `apt-get`

* then the playbook can be writen like this:

  ```yaml
  - hosts: all-bamboo-agents

    vars:
      - osfile_location: "{% if ansible_distribution=='MacOSX' %}osx{% elif ansible_distribution=='Ubuntu' %}linux{% elif ansible_os_family=='Windows' %}windows{% endif %}"

    pre_tasks:

      - name: '[BAMBOO] empty capabilities declaration'
        set_fact:
          bamboo_capabilities: {}

    roles:
      # Adding python3 role to the agent
      - role: bamboo-agent-python3
        python3_version: "{{ bamboo_python3_version }}"
        python3_installer: "{{ bamboo_python3_installer }}"

    post_tasks:
      - name: Updating the agent capability file
        include_role:
          name: atlassian-bambooagent-role
          tasks_from: write_capability
        tags:
          - capability:write
  ```

License
-------

BSD

Author Information
------------------

An optional section for the role authors to include contact information, or a website (HTML is not allowed).
