# wireguard-vpn
This Ansible playbook configure your server as wireguard VPN concentrator, also it could create on Amazon Web Services (AWS) instance which will become your wireguard VPN concentrator.

# Usage
If you want to launch an AWS EC2 instance create **custom-variables.yaml** file and inside it set your **aws_access_key**, **aws_secret_key** and **aws_key_name** variables:

```shell
$ touch custom-variables.yaml
$ vi custom-variables.yaml
$ cat custom-variables.yaml
---
aws_provision: true
aws_access_key: <your aws access key>
aws_secret_key: <your aws secret key>
aws_key_name: <your aws key name>
aws_region: eu-south-1
aws_image_id: ami-0ef2a393a2f3af862 # Amazon Linux 2023 AMI 64-bit (x86)
```

instead if don't use AWS, set **aws_provision** to false and create your own Ansible **hosts** file:
```shell
$ touch custom-variables.yaml
$ vi custom-variables.yaml
$ cat custom-variables.yaml
---
aws_provision: false

$ touch hosts-wireguard
$ vi hosts-wireguard
$ cat hosts-wireguard
[wireguard]
swg01 ansible_ssh_user=<User with sudo permission> ansible_ssh_host=<IP or hostname> ansible_ssh_common_args='-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null'
```
## Run the Ansible Playbook
If you're running wireguard VPN on AWS run the Ansible Playbook only with your custom-variables.yaml file:

```shell
$ ansible-playbook --extra-vars "@custom-variables.yaml" main.yaml
```

instead if you want to run the Ansible playbook on host not automatically provisioned on AWS run with custom-variables.yaml file and hosts-wireguard inventory file:
```shell
$ ansible-playbook -i hosts-wireguard --extra-vars "@custom-variables.yaml" main.yaml
```

## Link your wg0.conf fetched file to system wgX.conf
The playbook automatically download under **fetchedfiles** your peer wg0.conf:
```shell
$ tree fetchedfiles/
fetchedfiles/
├── peerprivatekey
├── peerpublickey
├── privatekey
├── publickey
├── README
└── wg0.conf
```

I'd suggest you to create a symlink beetween this wg0.conf and system wgX.conf file like /etc/wireguard/wg0.conf

```shell
$ sudo ln -s $(/bin/pwd)/fetchedfiles/wg0.conf /etc/wireguard/wg0.conf
```

## Start wireguard VPN
Run wg-quick command:
```shell
$ sudo wg-quick up wg0
```
