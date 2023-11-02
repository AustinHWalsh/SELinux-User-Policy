## Setup

### Create a custom SELinux user

```
sudo semanage user -a -R "user_r" example_u
```

### Create a new user 

```
sudo useradd -Z example_u example
sudo passwd example
```

### Setup SSH

```
mkdir /home/example/.ssh

chown -R example:example /home/example/.ssh
cp ~/.ssh/authorized_keys /home/example/.ssh
chmod 700 /home/example/.ssh
chmod 600 /home/example/.ssh/authorized_keys
```

### Allow new user to have Sudo access

```
sudo usermod -aG wheel example
```

The user will now be confined as an "example\_u" with the role "user\_r" which does not permit sudo access. This overrides the sudo access given by the above command.

The confirm this set SELinux to permissive
```
sudo setenforce 0
```
