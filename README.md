# Testing Ansible Roles

This framework provides the necessary files and configurations to easily setup your environment for testing ansible-roles.
It uses test-kitchen, vagrant and serverspec to test your roles on multiple operating systems.
It also supports automated travis-test out of the box.

## Prerequesites

Install the following software:

- Git
- Ruby
- VirtualBox
- Vagrant

## Setup

Create a directory for your role you want to test (called `ansible_role` in the following example).
**The name directory and the name of the role have to be the same!**
Git-clone the testing-framework into your newly created directory and change into it:
```
# basic setup
mkdir ansible_role
git clone https://github.com/rndmh3ro/ansible-test-framework ansible_role/
cd ansible_role/
```

Create an empty role with `ansible-galaxy`.
Run inside your role-directory, replace `ansible_role` with the name you gave the directory.

```
# create empty ansible role
ansible-galaxy init -p ../ --force ansible_role
```

Install test-kitchen, serverspec, the provisioner, driver and all its dependencies, with the help of [bundler]:
```
# Install software and dependencies
gem install bundler
bundle install
```

Customize your testing-setup. 
Replace the default name `ansible-test-framework` with the name of your role (in this example `ansible_role`) in two places:
- `default.yml` -> replacement should be in the `roles_path`.
- `.kitchen.yml` -> replacement should be the first item after `roles`.

You can also use this `sed`-command to replace the occurences.
Just replace `ansible_role` in the command with the name of your role.

```
# replace ansible-test-framework with your role-name in:
sed -i 's/ansible-test-framework/ansible_role/g' default.yml .kitchen.yml
```

## Write the role and tests

Write your ansible role and tests now.
There's already a file called *test_spec.rb* where you can write your tests in and the spec_helper is configured for serverspec.

For more help writing tests and using serverspec, read the docs:
- http://kitchen.ci/docs/getting-started/writing-server-test
- https://github.com/neillturner/kitchen-ansible#test-kitchen-serverspec

## Testing

You can test single machines, a set of machines or all at once. See the following examples or take a look at the test-kitchen [docs].
```

    # fast test on one machine
    bundle exec kitchen test default-ubuntu-1204

    # test on all machines in parallel
    bundle exec kitchen test -c

    # test all ubuntu machines
    bundle exec kitchen test ubuntu
```

## Tips

While developing roles it can be cumbersome to always destroy and recreate the machines if a test or the role fails at some point.
To circumvent this, you can create and then converge the machine. If the role fails during converging, you can simply run the converge again:

    # for development
    bundle exec kitchen create default-ubuntu-1204
    bundle exec kitchen converge default-ubuntu-1204

    # ... run fails, change role, converge again

    bundle exec kitchen converge default-ubuntu-1204


If your role takes a long time to run and you want to debug a specific task, you can run the converge with the help of an environment variable like this:

    ANSIBLE_EXTRA_FLAGS='--start-at-task="ansible_role | name of last working instruction"' bundle exec kitchen converge

Replace *ansible_role | name of last working instruction* with the name of the task you want to start at, so you can skip others.

Similary if you want to skip certain tasks, you can use the environment variable like this:

    ANSIBLE_EXTRA_FLAGS='--skip-tags=beginning' bundle exec kitchen converge

[test-kitchen]: https://github.com/test-kitchen/test-kitchen
[vagrant]: https://www.vagrantup.com/
[VirtualBox]: https://www.virtualbox.org/
[rake]: https://github.com/ruby/rake
[serverspec]: http://serverspec.org/
[kitchen-ansible]: https://github.com/neillturner/kitchen-ansible
[kitchen-vagrant]: https://github.com/test-kitchen/kitchen-vagrant
[kitchen-sync]: https://github.com/coderanger/kitchen-sync
[kitchen-transport-rsync]: https://github.com/unibet/kitchen-transport-rsync
[thor-foodcritic]: https://github.com/reset/thor-foodcritic
[hardening.io]: http://hardening.io/
[git]: https://www.git-scm.com/
[bundler]: http://bundler.io/
