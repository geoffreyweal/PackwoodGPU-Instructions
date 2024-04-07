# PackwoodGPU Instructions

This repository provides the information for using the PackwoodGPU computer

## Logging in

To log in via ssh, you want to type the following into the terminal

```bash
ssh -Y packwoodgpu_username@packwoodgpu_ip_address
```

where ``packwoodgpu_username`` is your username, and ``packwoodgpu_ip_address`` is the ip address for the PackwoodGPU computer. Get this from the Packwood group. 

You will be asked to the follow: 
```bash
The authenticity of host 'XX.XXX.XX.XXX (XX.XXX.XX.XXX)' can't be established.
YYYYYYYY key fingerprint is ZZZZZZZ:qwertyuiopasdfghjklzxcvbnm/1234567890
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

Type ``yes`` and hit enter. You will only be asked this once. Then type in your password, and you will gain access to your login on the PackwoodGPU computer. 

You can also set up your login so that you dont need to enter your password every time. This is generally recommended as it is a more secure way of logging in. To do this:

1. Make sure you are logged in to the PackwoodGPU computer. 
2. Generate a key on your computer by typing the following into the terminal and hitting enter:
    * You will be asked to enter in a passphrase. If you decide to give a passphrase, enter it here. Otherwise just hit enter twice. You do not need to give a passphrase
```bash
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_packwoodgpu
```
3. copy the public key to your login on the PackwoodGPU computer. Do this by entering into the following into the terminal, where ``packwoodgpu_username`` is your username, and ``packwoodgpu_ip_address`` is the ip address for the PackwoodGPU computer. Get this from the Packwood group. 
```bash
ssh-copy-id packwoodgpu_username@packwoodgpu_ip_address
```
4. You can not log in without a password by typing the following into the terminal
```bash
ssh -Y packwoodgpu_username@packwoodgpu_ip_address -i ~/.ssh/id_ed25519_packwoodgpu
```

## Make your life easier by entering in custom commands into the ``.bashrc`` file

To make life easier, you can also include the following in your ``.bashrc`` file which will allow you to log in without having to type the full statement into the terminal. To do this:

1. Open a new terminal, type ``cd`` into the terminal, and press enter
2. Type the full test below into the terminal and press enter:
```bash
echo '
#############################################
# PackwoodGPU Computer

packwoodgpu_username=Put your user name in here
packwoodgpu_ip_address=Put the ip address for the PackwoodGPU computer here

alias packwoodgpu="ssh -Y $packwoodgpu_username@$packwoodgpu_ip_address -i ~/.ssh/id_ed25519_packwoodgpu"

function topdgpuserver()
{
scp -r "${1}" $packwoodgpu_username@$packwoodgpu_ip_address:"${2}"
}
function frompdgpuserver()
{
scp -r $packwoodgpu_username@$packwoodgpu_ip_address:"${1}" "${2}"
}

#############################################' >> ~/.bashrc
```
3. Open the ``.bashrc`` file using either ``vim`` or ``nano``:
```bash
vim ~/.bashrc
# If this doesnt work, try the following
vi ~/.bashrc
# or
nano ~/.bashrc
```

4. Scroll to the bottom of the file and check that the following has been entered in here.
    * Note: You will need to change the input for ``packwoodgpu_username`` and ``packwoodgpu_ip_address`` to your own username and the ip address of the PackwoodGPU computer.
5. Source the ``.bashrc`` file by typing into the terminal
```bash
source ~/.bashrc
```

#### Notes for using ``vim`` or ``vi`` to change the inputs for ``packwoodgpu_username`` and ``packwoodgpu_ip_address``:

Type the following buttons into ``vim`` and press enter to perform the following commands:

* `i` (insert): Allows you to edit the file. Do this to add your username and the ip address of the PackwoodGPU computer.
* `esc`: Step doing what you are doing (for example, stop editting the file). You may need to hit this a few time in order to allow you to perform other commands in ``vim``
* `u` (undo): If you want to undo your last commands, press this. Press it multiple times if you want to undo multiple previous commands. 
* `w` (write): Save the ``.bashrc`` file.
* `q` (quit): Quit the ``.bashrc`` file.
* `wq` (write and quit): Save the ``.bashrc`` file and then quit.
* `q!` (force quit): If you want to quit, but you have made changes to the ``.bashrc`` but you dont want to save your changes, press this. 

[Click here](https://opensource.com/article/19/3/getting-started-vim) to learn more about how to use ``vim``/``vi``

## Transferring files between PackwoodGPU and your computer

You will notice that in the above that there are two functions, called ``topdgpuserver`` and ``frompdgpuserver``. These are:

```bash
function topdgpuserver()
{
scp -r "${1}" $packwoodgpu_username@$packwoodgpu_ip_address:"${2}"
}
function frompdgpuserver()
{
scp -r $packwoodgpu_username@$packwoodgpu_ip_address:"${1}" "${2}"
}
```

These functions make it easy to transfer files and folder between the PackwoodGPU computer and your own computer. 

#### Transfer a file/folder from your computer to PackwoodGPU

To transfer a file to PackwoodGPU, type the following into the terminal:

```bash
topdgpuserver file.txt path_to_write_file_to
```

You can also copy a folder to the PackwoodGPU computer with this command.

```bash
topdgpuserver folder_name path_to_write_file_to
```

#### Transfer a file/folder from PackwoodGPU to your computer

To transfer a file from PackwoodGPU to your computer, type the following into the terminal:

```bash
frompdgpuserver file.txt path_to_write_file_to_on_your_computer
```

You can also copy a folder from the PackwoodGPU computer to your computer with this command.

```bash
frompdgpuserver folder_name path_to_write_file_to_on_your_computer
```

