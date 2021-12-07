# monero-nodes-ansible

Monero node ansible playbook setup and hardening

### Setup

Edit `env_variables` file and set current monerod version, for example `v0.17.2.3` (current version).

Edit `4-playbook-ss-.yaml` file and edit:

```yaml
  - name: Ensure valid ssh accounts
    lineinfile:
      state: present
      dest: /etc/ssh/sshd_config
      regexp: "^#AllowUsers|^AllowUsers"
      line: 'AllowUsers root'
      validate: /usr/sbin/sshd -t -f %s
```

and add you user to `AllowUsers root`, for example `AllowUsers myusername root`

Edit `hosts` file and include your own nodes, for example:

```bash
[nodes]
node1	ansible_host=10.15.22.31
node2	ansible_host=10.15.22.32
node3	ansible_host=10.15.22.33
node4	ansible_host=10.15.22.34
```

then apply your manifests, (you need to be sure that the ssh passworless access is granted to root user)

```shell
root@node0:~# ansible-playbook 1-playbook-monero-nodes.yaml
root@node0:~# ansible-playbook 2-playbook-kernel-hardening.yaml
root@node0:~# ansible-playbook 3-playbook-aide.yaml
root@node0:~# ansible-playbook 4-playbook-ssh.yaml
root@node0:~# ansible-playbook 5-playbook-firewall.yaml
```

Your'e done !!

TODO: Add Tor support

# Additional Monero Daemon Options

Below are some popular daemon options. Please ignore all brackets [] and the help text after 
(see https://www.monero.how/how-to-run-monero-node)

```shell
#. You will need to replace these with your own values.

--rpc-bind-ip [IP ADDRESS] # Binds the daemon to an IP address. You need to use your external IP if you plan to access this daemon from outside the internal network, or an internal one if you only want it to work for devices in the same network. If you are unsure about whether to use internal or external, you most likely want to use the external IP address. You can find this by using the IP address the VPS provider gave you or by searching for it with a site such as ipleak.net.

--rpc-bind-port [PORT] # Binds the daemon to a port. Keep in mind the daemon will be unsafe unless this option is also run with --restricted-rpc. The default option is 18081, though some services (such as MoneroWorld) use 18089.

--restricted-rpc # Restricts the actions that external users can perform when they are connected to the node over RPC. You will typically want to use this option.

--confirm-external-bind # A required verification if using RPC bind options.

--rpc-login [USERNAME]:[PASSWORD] # Restricts use of the node to users who know the username and password. Make sure to use a strong password.

--limit [KB/s] # Limits the total download and upload limit to a certain value in kilobytes per second. Eg: 128 would set the maximum upload and download speed to one megabit per second.

--limit_up [KB/s] # Limits the total upload speed to a certain value in kilobytes per second.

--limit_down [KB/s] # Limits the total download speed to a certain value in kilobytes per second.

--data-dir [LOCATION] # Saves the blockchain to a manual location by file path. Can be used to save the blockchain in another folder on one hard drive or even another hard drive or flash drive.

--db-sync-mode safe # Syncs the blockchain in a way that avoids corruption. This is much slower, so it’s typically best to run with the normal parameters without worrying about a very small chance of corruption.

--out-peers [NUM] # Sets the max number of outgoing peers (ones you connect with). The default is 8.

--block-sync-size [NUM] # Sets the number of batched blocks. The default is 20. If you are having issues syncing the blockchain, try reducing the number to 10.

--db-salvage # Try using this command if your database becomes corrupt.

--add-peer [IP]:[PORT] # Manually adds a peer by IP address and port.
```