# Build Jupyterhub by Anaconda Environment
Because the official documentation use built-in python to build jupyterhub, not use an anaconda environment. So this tutorial explains how to build jupyterhub by anaconda environment, and forgive my poor English.
## OS
ubuntu-20.04.2-live-server-amd64
# 1. Install Node.js (LTS)
```bash
test_admin@testserver:~$ curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo -E bash -
```
```bash
test_admin@testserver:~$ sudo apt-get install -y nodejs
```
# 2. Install configurable-http-proxy
```bash
test_admin@testserver:~$ sudo npm install -g configurable-http-proxy
```
# 3. Install latest Anaconda
```bash
test_admin@testserver:~$ wget https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh
```
```bash
test_admin@testserver:~$ bash Anaconda3-2020.11-Linux-x86_64.sh

Do you accept the license terms? [yes|no]
[no] >>> yes

Do you wish the installer to initialize Anaconda3
by running conda init? [yes|no]
[no] >>> yes
```
When done Anaconda install
```bash
test_admin@testserver:~$ source .bashrc
```
It will let your bash like this
```bash
(base) test_admin@testserver:~$ 
```
# 4. Create new environment
You can change the name and version whatever you want
```bash
(base) test_admin@testserver:~$ conda create --name jupyterhub37 python=3.7

Proceed ([y]/n)? y
```
# 5. Set the new environment as default
Edit .bashrc you can see the conda section in the bottom
```bash
(base) test_admin@testserver:~$ vi .bashrc
...
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/test_admin/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/test_admin/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/test_admin/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/test_admin/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```
Add `source activate jupyterhub37` in the end of conda section
```bash
# >>> conda initialize >>>
# !! Contents within this block are managed by 'conda init' !!
__conda_setup="$('/home/test_admin/anaconda3/bin/conda' 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "/home/test_admin/anaconda3/etc/profile.d/conda.sh" ]; then
        . "/home/test_admin/anaconda3/etc/profile.d/conda.sh"
    else
        export PATH="/home/test_admin/anaconda3/bin:$PATH"
    fi
fi
unset __conda_setup
source activate jupyterhub37
# <<< conda initialize <<<
```
Save .bashrc
```bash
(base) test_admin@testserver:~$ source .bashrc
```
It will like this
```bash
(jupyterhub37) test_admin@testserver:~$ 
```
If you have better solution to set environment as default please tell me
# 6. Install requirement package
This will install
- jupyterhub
- notebook
- jupyterlab
- sudospawner
```bash
(jupyterhub37) test_admin@testserver:~$ conda install -y -c conda-forge jupyterhub && conda install -y -c conda-forge notebook && conda install -y -c conda-forge jupyterlab && conda install -y -c conda-forge sudospawner
```
# 7. Generate and set jupyter config
```bash
(jupyterhub37) test_admin@testserver:~$ mkdir /home/test_admin/jupyterhub && cd /home/test_admin/jupyterhub
```
```bash
(jupyterhub37) test_admin@testserver:~/jupyterhub$ jupyterhub --generate-config
```
```bash
(jupyterhub37) test_admin@testserver:~/jupyterhub$ vi jupyterhub_config.py
```
Add below config to the config file top<br>

c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'<br>
c.SudoSpawner.sudospawner_path = '/home/test_admin/anaconda3/envs/jupyterhub37/bin/sudospawner'<br>
c.Spawner.default_url = '/lab'<br>
c.PAMAuthenticator.open_sessions=False<br>
c.Authenticator.admin_users = {'test_admin'}<br>

and will like this
```bash
c.JupyterHub.spawner_class = 'sudospawner.SudoSpawner'
c.SudoSpawner.sudospawner_path = '/home/test_admin/anaconda3/envs/jupyterhub37/bin/sudospawner'
c.Spawner.default_url = '/lab'
c.PAMAuthenticator.open_sessions=False
c.Authenticator.admin_users = {'test_admin'}
# Configuration file for jupyterhub.
  
#------------------------------------------------------------------------------
# Application(SingletonConfigurable) configuration
#------------------------------------------------------------------------------
## This is an application.
...
```
# 7. Test jupyterhub
Remember that, you need to start jupyterhub in the path where your config file is
```bash
(jupyterhub37) test_admin@testserver:~/jupyterhub$ sudo /home/test_admin/anaconda3/envs/jupyterhub37/bin/jupyterhub
```
Open your browser and enter http://yourip:8000, if everything goes well, use Ctrl+C to close jupyterhub
# 8. Safer setting
Documentation mentions that it isn’t especially safe, because you have a process running on the public web as root. The safer setting refers to jupyterhub documentation, if you want to know more detail, You can read 
<https://jupyterhub.readthedocs.io/en/stable/reference/config-sudo.html>
```bash
(jupyterhub37) test_admin@testserver:~$ sudo useradd jupyterhub_admin
```
## 8.2 Edit /etc/sudoers
```bash
(jupyterhub37) test_admin@testserver:~$ sudo visudo
```
add this line to below

Cmnd_Alias JUPYTER_CMD = /home/test_admin/anaconda3/envs/jupyterhub37/bin/sudospawner<br>
jupyterhub_admin ALL=(%jupyterhub_admin) NOPASSWD:JUPYTER_CMD

It will like this
```bash
...
# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d

Cmnd_Alias JUPYTER_CMD = /home/test_admin/anaconda3/envs/jupyterhub37/bin/sudospawner
jupyterhub_admin ALL=(%jupyterhub_admin) NOPASSWD:JUPYTER_CMD
```
## 8.2 Add user to jupyterhub_admin group to use jupyterhub
```bash
(jupyterhub37) test_admin@testserver:~$ sudo usermod -a -G jupyterhub_admin test_admin
```
### 8.2.1 Create new user to use jupyterhub
```bash
(jupyterhub37) test_admin@testserver:~$ sudo adduser test_user_1
Adding user `test_user_1' ...
Adding new group `test_user_1' (1003) ...
Adding new user `test_user_1' (1003) with group `test_user_1' ...
Creating home directory `/home/test_user_1' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for test_user_1
Enter the new value, or press ENTER for the default
        Full Name []: 
        Room Number []: 
        Work Phone []: 
        Home Phone []: 
        Other []: 
Is the information correct? [Y/n] y
```
```bash
(jupyterhub37) test_admin@testserver:~$ sudo usermod -a -G jupyterhub_admin test_user_1
```
## 8.2 Test sudo setup
```bash
(jupyterhub37) test_admin@testserver:~$ sudo -u jupyterhub_admin sudo -n -u test_admin /home/test_admin/anaconda3/envs/jupyterhub37/bin/sudospawner --help
Usage: /home/test_admin/anaconda3/envs/jupyterhub37/bin/sudospawner [OPTIONS]

Options:

  --help                           show this help information
...
```
## 8.3 Enable PAM for non-root
```bash
(jupyterhub37) test_admin@testserver:~$ sudo usermod -a -G shadow jupyterhub_admin
```
## 8.4 Test that PAM works
```bash
(jupyterhub37) test_admin@testserver:~$ sudo -u jupyterhub_admin /home/test_admin/anaconda3/envs/jupyterhub37/bin/python -c "import pamela, getpass; print(pamela.authenticate('test_admin', getpass.getpass()))"
Password: [enter your unix password]
```
## 8.5 Change entire jupyterhub folder owner
```bash
(jupyterhub37) test_admin@testserver:~$ sudo chown -R jupyterhub_admin /home/test_admin/jupyterhub/
```
## 8.6 Start jupyterhub
```bash
(jupyterhub37) test_admin@testserver:~$ cd /home/test_admin/jupyterhub/
```
```bash
(jupyterhub37) test_admin@testserver:~/jupyterhub$ sudo -u jupyterhub_admin /home/test_admin/anaconda3/envs/jupyterhub37/bin/jupyterhub
```
