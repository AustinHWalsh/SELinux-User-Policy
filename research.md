# SELinux-Policy
Project Goal: develop an SELinux or set of SELinux policies that manage the permissions of a daemon

An assignment for COMP6841 23T3

### SELinux users
In this section, the distinction between SELinux user and account are important. When talking about an account, it means the user logged into the system. For example,
by default on Fedora, *fedora* is the account. When executing a command with `sudo`, *root* is the account.
 
A SELinux user is an immutable field of the label used to audit and control access of the current account of the system. 
There exists multiple SELinux users:

* unconfined\_u: SELinux user for unrestricted users. They have very few restrictions in applied to their user and is the default if an account is not mapped to a user. 
* root: The SELinux user for the root account
* sysadm\_u: SELinux user with direct system admin role assigned
* staff\_u: SELinux user for accounts that need to run both non-admin and admin commands. This is done through the staff\_r and sysadm\_r roles.
* user\_u: SELinux user for non-admin accounts
* system\_u: Special SELinux user for system services. It is not possible to directly use this user.

Accounts are mapped to a single SELinux user.
The SELinux users can be found using `semanage user -l`

### SELinux roles
SELinux roles dictact which SELinux types are accessible if part of that role. Simply put a SELinux role allows a SELinux user to access a SELinux type.
SELinux users can be assigned to multiple roles, however, they can only assume a single role at a time and must swap between them to access different types if required.

This diagram depicts how this interacts, accounts are given an SELinux user. SELinux users are assigned SELinux roles. SELinux roles allow access to SELinux types/domains
![SELinux relationship diagram](https://wiki.gentoo.org/images/a/a0/SELinux_users.png)

The standard set of SELinux roles are:
* object\_r: Default role for all processes/resources, cannot be assumed by a user
* user\_r: The regular user role, only allows user applications and other non-privileged types
* staff\_r: Similar to user\_r, except allows the user to receive more system info and mainly given to users who need to swap roles
* sysadm\_r: System admin role, allowing access to most types
* system\_r: System role, not meant to be assumed by a user

### SELinux type
The SELinux type is the most important field in a label. When SELinux policies determine if a process can access a resource, it is checking if the process's type 
has access to the resource's type. The other fields in the label are ignored, they are only there to allow processes to define their type.

Types are defined by default, and inherited by newly-created files

For example, in my home directory, it contains the following files and their labels:
```
drwx------. 1 fedora fedora unconfined_u:object_r:user_home_dir_t:s0   152 Sep 24 11:21 .
drwxr-xr-x. 1 root   root   system_u:object_r:home_root_t:s0            12 Sep 19 04:14 ..
-rw-------. 1 fedora fedora unconfined_u:object_r:user_home_t:s0       692 Sep 24 08:53 .bash_history
-rw-r--r--. 1 fedora fedora unconfined_u:object_r:user_home_t:s0        18 Feb  6  2023 .bash_logout
-rw-r--r--. 1 fedora fedora unconfined_u:object_r:user_home_t:s0       141 Feb  6  2023 .bash_profile
-rw-r--r--. 1 fedora fedora unconfined_u:object_r:user_home_t:s0       524 Sep 24 09:10 .bashrc
drwxr-xr-x. 1 fedora fedora unconfined_u:object_r:cache_home_t:s0       20 Sep 24 08:46 .cache
-rw-r--r--. 1 fedora fedora unconfined_u:object_r:user_home_t:s0        60 Sep 24 09:08 .gitconfig
drwxr-xr-x. 1 fedora fedora unconfined_u:object_r:user_home_t:s0        28 Sep 24 09:04 src
drwx------. 1 fedora fedora system_u:object_r:ssh_home_t:s0            130 Sep 24 10:42 .ssh
-rw-------. 1 fedora fedora unconfined_u:object_r:user_home_t:s0     10834 Sep 24 11:20 .viminfo
``` 

The `user_home_t` type is assigned to all labels for resources created in the home directory, unless otherwise. This also extends to files in subdirectories,
so any files created in the `src` directory will have the `user_home_t` assigned, but this can be changed.
The `ssh_home_t` is a type specific for the files hosted in the `~/.ssh/` directory. 

## Confining Accounts
Good practice before the development of a policy is to confine accounts with users outlined in the previous sections. To check the current account's SELinux label, the command `id -Z` is used. For example, when logged in as `fedora`, my label is:
```bash
unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
```

Despite having the SELinux users available, my account is still unconfined. This is because there is no mapping between the account and a user. There are a couple options to solve this, either create a new account with a mapping or confine regular accounts.

### Create a new Account
Creating a new account and confining it to a user can be done with the command
```bash
sudo useradd -Z <selinx_u> <example_account>
```

This will create an account called `<example_account>` which automatically maps to the `<selinx_u>`. For example, I create an account called `staff` which maps to the `staff_u` user. Now when I login using `staff`, my label becomes:
```bash
staff_u:staff_r:staff_t:s0
```

