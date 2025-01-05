# Trusted Postgres Architect on bare metal

### To use TPA on bare metal do this:

- Change IP address
- `sudo hostnamectl set-hostname <newhostname>`
- Using `sudo visudo` replace `%wheel  ALL=(ALL)       ALL` with `%wheel  ALL=(ALL)       NOPASSWD:ALL`
- `sudo systemctl restart sshd.service`
- `sudo systemctl set-default multi-user.target`
- `sudo dnf groupremove  -y 'GNOME' 'X Window System'`
- `sudo systemctl disable firewalld.service`
- `sudo dnf remove -y firewalld`
- `sudo dnf install chrony && sudo systemctl enable --now chronyd`
- Reboot
- Copy your SSH key to the targets using `ssh-copy-id -i ~/.ssh/id_rsa ton@<target>`
- In your config, make sure the correct key and ansible user are defined.
```
keyring_backend: system
vault_name: b42cba99-6453-4842-907b-4822b61230ea
ssh_key_file: ~/.ssh/id_rsa
...
instance_defaults:
  platform: bare
  vars:
    manage_ssh_hostkeys: no
    ansible_user: ton
```
- Make sure you create your EDB Repo 2.0 token using `export EDB_SUBSCRIPTION_TOKEN=<your token>` and check if the variable exists using `env`.
- Run `tpaexec provision .`
- For each host, run `ssh-keyscan -H <target> >> tpa_known_hosts` to add the targets to the TPA known hosts file.
- To test use `tpaexec ping .`
```
red | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
white | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
blue | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
pem | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
barman | SUCCESS => {
    "changed": false,
    "ping": "pong"
}

real    0m3.435s
user    0m1.773s
sys     0m0.828s
```
- Create the EDB_SUBSCRIPTION_TOKEN variable using `. ./tokens.sh`.
- If all ok, run `tpaexec deploy .`

> [!IMPORTANT]
> Once you have provisioned the environment on bare metal there is no way back. `tpaexec deprovision .` removes all TPA assets, but doesn't remove any software from the servers.

Get your enterprisedb password `tpaexec show-password . enterprisedb` (currently `Q?LoaBflDcxF%Tp#C0evNvwuu2u29x7o`)

PEM is running on `https://pem:pem`


