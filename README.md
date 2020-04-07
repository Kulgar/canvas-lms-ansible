# Canvas LMS Ansible

These Ansible scripts will install all the Canvas LMS dependencies on a remote server with common security measures. Careful, they won't actually **deploy** Canvas LMS, but they will prepare everything so that Canvas LMS can be deployed using **Capistrano**.

These scripts are normally following this official guide recommendations: [Canvas LMS Production Start](https://github.com/instructure/canvas-lms/wiki/Production-Start#production-start)

With some extra stuff (nginx instead of apache, rvm to handle several rubies...)

These scripts main goal is to offer a solid basis to install a server ready to host a Canvas LMS app. You can fork them, download them, modify playbooks, add your own rules and do whatever with them.

All PRs to improve these scripts are welcomed.

## Servers roles / inventory

Create an `inventory` file from `inventory.example` and add your server IPs there.

You can also specify some variables there: [Ansible inventory parameters](https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html#connecting-to-hosts-behavioral-inventory-parameters)

We usually set ansible_user and eventually ansible_ssh_private_key_file. Or we use `~/.ssh/config` or `ssh-agent`. Don't forget `ansible_port` if you decided to change the default ssh 22 port.

## Prepare variables

In `playbooks/group_vars` you will find different sub folders, one per server group.
Create a `secrets.yml` file from every `secrets.yml.example` files you will find there.

Be sure to read comments in these files and set your variables accordingly.

All playbooks also have comments and some commented lines.
Be sure to read these ones, as it is always best to understand what is actually going on.

## Prepare server:

### If you don't have a sudoer user

Be sure to set `create_admin_account` in `all/secrets.yml` to `true` and set the `server_admin_account` to the sudoer account name you want.

Then run the first server playbook as root:

**!! This is the only playbook you should run as root and only once!!**

```bash
ansible-playbook ./playbooks/1_server.yml -k -u root -i ./inventory
```
Then you need to run everything else with the `server_admin_account` you set.

### If you do have a sudoer user already

Set `create_admin_account` in `all/secrets.yml` to `false` and set the `server_admin_account` to the sudoer account name you want to use.

Then run ansible scripts.

## Run ansible scripts

Then you can run the 'all.yml'

```bash
ansible-playbook ./playbooks/all.yml -i ./inventory
```

Or you can run them individually in that order:

1 - server
2 - security
3 - db
4 - redis
5 - ruby_env
6 - prepare_deploy
7 - web_server

Now use capistrano for first deploy, within the canvas project:
__use staging instead of production for preprod__

```bash
cap production deploy first_deploy=true
```

Populate db using ansible populate_db playbook:

```bash
ansible-playbook ./playbooks/populate_db.yml -i ./inventory
```

Rerun capistrano for final deploy:
__use staging instead of production for preprod__

```bash
cap production deploy
```

And finally, a last playbook that will install delayed jobs and QTI migration tools:

```bash
ansible-playbook ./playbooks/after_deploy.yml -i ./inventory
```

