# WHAT ARE TASKS?

Tasks are the most basic unit of work in your deployment configuration. Like
most everything in Ansible, tasks are specified using YAML. Each task is backed
by an Ansible module which provides the relevant functionality.

By default, Ansible runs tasks in a synchronous/blocking fashion. While it is
possible to execute tasks asynchronously, that use-case will not be covered in
this workshop.

The example below shows a task listing that displays the hostname of the
machine the task is being run on using the [debug module]. Then, it installs
nginx using the [yum module].

```
tasks:
  - debug:
      var: ansible_distribution
  - yum:
      name: epel-release
      state: present
  - yum:
      name: nginx
      state: present
```

Module documentation is also available on the web at:
http://docs.ansible.com.

*You can also use the command `ansible-doc`*

### NAMING TASKS

While the tasks above would run successfully, it would behoove you to take the
time to name your tasks so they are more easily understood, both for others
and yourself in the future. This can be done like so:

```
tasks:
  - name: show the distribution
    debug:
      var: ansible_distribution
  - name: ensure extra packages are available
    yum:
      name: epel-release
      state: present
  - name: ensure nginx is installed
    yum:
      name: nginx
      state: present
```

### IDEMPOTENCY

Most Ansible tasks are idempotent by default. This means that you can run the
same task repeatedly and the target host will remain in the same state. This
may not seem compelling when considering something as simple as installing a
piece of software, but it quickly becomes critical as your tasks get more
complex.

For example, imagine running a task to add a line to a configuration file. Even
if you run that task twenty times, you only want the line added once (like most
things, Ansible has a module for that, it's called `lineinfile`).

### USING SUDO

If you run a task that requires administrative privileges, you'll need to
"elevate" it with the `become` directive, e.g:

```
tasks:
  - name: ensure extra packages are available
    yum:
      name: epel-release
      state: present
    become: yes

  - name: ensure nginx is installed
    yum:
      name: nginx
      state: present
    become: yes
```

You can also elevate an entire playbook with `become: yes` at the top level.
A `become_method` can also be specified, which tells Ansible how the permission
elevation should happen. This defaults to `sudo`, which is probably what you'll
want to use most of the time.

## EXERCISE

A playbook with named tasks has been provided. Try to fill in the missing
configuration. Once you have the playbook working, run it multiple times,
paying close attention the output to see that (when possible) Ansible will
run idempotently.

```
ansible-playbook -i 10.10.10.10, playbook.yml
```

Try SSHing into your target machine to run `sudo yum remove nginx`. Log off and
run your playbook again. Note how Ansible restores the machine to the correct
state.

Try removing an installation task from the playbook and note that running the
playbook does not uninstall the package! It is important to understand that
Ansible leaves no configuration on the machines it controls. Instead, it
dynamically introspects the state of the machine on a per-task basis each time
it is run.

If you need to remove something, you will almost always be altering the `state`
directive of the task.

## LEARNING OBJECTIVES

- Where is the best place to look for documentation about an Ansible module?
- How do you install a yum package using Ansible?
- What is idempotency?
- How does Ansible indicate if something changed?
- What happens on the next run of a playbook if you delete something instead of
  changing its `state`?

[debug module]: http://docs.ansible.com/ansible/debug_module.html
[yum module]: http://docs.ansible.com/ansible/yum_module.html
