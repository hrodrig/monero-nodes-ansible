# monero-nodes-ansible

Monero node ansible playbook setup and hardening with Tor support

### Changelog

see changelog [here](CHANGELOG.md)

### Setup

Edit `env_variables` file and set current monerod version, for example `v0.17.3.0` (current version).

Edit `hosts` file and include your nodes, for example:

```shell
[nodes]
node1	ansible_host=10.15.22.31
node2	ansible_host=10.15.22.32
node3	ansible_host=10.15.22.33
node4	ansible_host=10.15.22.34
```

then apply your manifests, (you need to be sure that the ssh passworless access is granted to root user)

```shell
# base file
root@localhost:~# ansible-playbook 1-playbook-monero-nodes.yaml

# some kernel hardening
root@localhost:~# ansible-playbook 2-playbook-kernel-hardening.yaml

# configure AIDE
root@localhost:~# ansible-playbook 3-playbook-aide.yaml

# SSH hardening
root@localhost:~# ansible-playbook 4-playbook-ssh.yaml

# Firewall configuration
root@localhost:~# ansible-playbook 5-playbook-firewall.yaml

# you can use all playboks at same time
root@localhost:~# ansible-playbook 1-playbook-monero-nodes.yaml 2-playbook-kernel-hardening.yaml 3-playbook-aide.yaml 4-playbook-ssh.yaml 5-playbook-firewall.yaml
```

  **Tip:** you can speed bootstrap download by comment `db-sync-mode=safe` in `/etc/monero/monerod.conf` file and restart monerod service. After initial bootstrap download, it's recommend to set active the db-sync-mode option.


Check for monero logs:

```shell
tail -f /var/log/monero/monerod.log
```

You can check connections with `iptraf-ng`.

Some **bad monero guys ip addresses** are included in the file `/etc/monero/monero-ban-list.txt`. Please check this file.

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

From the monerod help:

```sh
# monerod --help
Options:
  --help                                Produce help message
  --version                             Output version information
  --os-version                          OS for which this executable was
                                        compiled
  --config-file arg (=/root/.bitmonero/bitmonero.conf, /root/.bitmonero/testnet/bitmonero.conf if 'testnet', /root/.bitmonero/stagenet/bitmonero.conf if 'stagenet')
                                        Specify configuration file
  --detach                              Run as daemon
  --pidfile arg                         File path to write the daemon's PID to
                                        (optional, requires --detach)
  --non-interactive                     Run non-interactive

Settings:
  --log-file arg (=/root/.bitmonero/bitmonero.log, /root/.bitmonero/testnet/bitmonero.log if 'testnet', /root/.bitmonero/stagenet/bitmonero.log if 'stagenet')
                                        Specify log file
  --log-level arg
  --max-log-file-size arg (=104850000)  Specify maximum log file size [B]
  --max-log-files arg (=50)             Specify maximum number of rotated log
                                        files to be saved (no limit by setting
                                        to 0)
  --max-concurrency arg (=0)            Max number of threads to use for a
                                        parallel job
  --public-node                         Allow other users to use the node as a
                                        remote (restricted RPC mode, view-only
                                        commands) and advertise it over P2P
  --zmq-rpc-bind-ip arg (=127.0.0.1)    IP for ZMQ RPC server to listen on
  --zmq-rpc-bind-port arg (=18082, 28082 if 'testnet', 38082 if 'stagenet')
                                        Port for ZMQ RPC server to listen on
  --zmq-pub arg                         Address for ZMQ pub - tcp://ip:port or
                                        ipc://path
  --no-zmq                              Disable ZMQ RPC server
  --data-dir arg (=/root/.bitmonero, /root/.bitmonero/testnet if 'testnet', /root/.bitmonero/stagenet if 'stagenet')
                                        Specify data directory
  --test-drop-download                  For net tests: in download, discard ALL
                                        blocks instead checking/saving them
                                        (very fast)
  --test-drop-download-height arg (=0)  Like test-drop-download but discards
                                        only after around certain height
  --testnet                             Run on testnet. The wallet must be
                                        launched with --testnet flag.
  --stagenet                            Run on stagenet. The wallet must be
                                        launched with --stagenet flag.
  --regtest                             Run in a regression testing mode.
  --keep-fakechain                      Don't delete any existing database when
                                        in fakechain mode.
  --fixed-difficulty arg (=0)           Fixed difficulty used for testing.
  --enforce-dns-checkpointing           checkpoints from DNS server will be
                                        enforced
  --prep-blocks-threads arg (=4)        Max number of threads to use when
                                        preparing block hashes in groups.
  --fast-block-sync arg (=1)            Sync up most of the way by using
                                        embedded, known block hashes.
  --show-time-stats arg (=0)            Show time-stats when processing
                                        blocks/txs and disk synchronization.
  --block-sync-size arg (=0)            How many blocks to sync at once during
                                        chain synchronization (0 = adaptive).
  --check-updates arg (=notify)         Check for new versions of monero:
                                        [disabled|notify|download|update]
  --fluffy-blocks                       Relay blocks as fluffy blocks
                                        (obsolete, now default)
  --no-fluffy-blocks                    Relay blocks as normal blocks
  --test-dbg-lock-sleep arg (=0)        Sleep time in ms, defaults to 0 (off),
                                        used to debug before/after locking
                                        mutex. Values 100 to 1000 are good for
                                        tests.
  --offline                             Do not listen for peers, nor connect to
                                        any
  --disable-dns-checkpoints             Do not retrieve checkpoints from DNS
  --block-download-max-size arg (=0)    Set maximum size of block download
                                        queue in bytes (0 for default)
  --sync-pruned-blocks                  Allow syncing from nodes with only
                                        pruned blocks
  --max-txpool-weight arg (=648000000)  Set maximum txpool weight in bytes.
  --block-notify arg                    Run a program for each new block, '%s'
                                        will be replaced by the block hash
  --prune-blockchain                    Prune blockchain
  --reorg-notify arg                    Run a program for each reorg, '%s' will
                                        be replaced by the split height, '%h'
                                        will be replaced by the new blockchain
                                        height, '%n' will be replaced by the
                                        number of new blocks in the new chain,
                                        and '%d' will be replaced by the number
                                        of blocks discarded from the old chain
  --block-rate-notify arg               Run a program when the block rate
                                        undergoes large fluctuations. This
                                        might be a sign of large amounts of
                                        hash rate going on and off the Monero
                                        network, and thus be of potential
                                        interest in predicting attacks. %t will
                                        be replaced by the number of minutes
                                        for the observation window, %b by the
                                        number of blocks observed within that
                                        window, and %e by the number of blocks
                                        that was expected in that window. It is
                                        suggested that this notification is
                                        used to automatically increase the
                                        number of confirmations required before
                                        a payment is acted upon.
  --keep-alt-blocks                     Keep alternative blocks on restart
  --extra-messages-file arg             Specify file for extra messages to
                                        include into coinbase transactions
  --start-mining arg                    Specify wallet address to mining for
  --mining-threads arg                  Specify mining threads count
  --bg-mining-enable                    enable background mining
  --bg-mining-ignore-battery            if true, assumes plugged in when unable
                                        to query system power status
  --bg-mining-min-idle-interval arg     Specify min lookback interval in
                                        seconds for determining idle state
  --bg-mining-idle-threshold arg        Specify minimum avg idle percentage
                                        over lookback interval
  --bg-mining-miner-target arg          Specify maximum percentage cpu use by
                                        miner(s)
  --db-sync-mode arg (=fast:async:250000000bytes)
                                        Specify sync option, using format
                                        [safe|fast|fastest]:[sync|async]:[<nblo
                                        cks_per_sync>[blocks]|<nbytes_per_sync>
                                        [bytes]].
  --db-salvage                          Try to salvage a blockchain database if
                                        it seems corrupted
  --p2p-bind-ip arg (=0.0.0.0)          Interface for p2p network protocol
                                        (IPv4)
  --p2p-bind-ipv6-address arg (=::)     Interface for p2p network protocol
                                        (IPv6)
  --p2p-bind-port arg (=18080, 28080 if 'testnet', 38080 if 'stagenet')
                                        Port for p2p network protocol (IPv4)
  --p2p-bind-port-ipv6 arg (=18080, 28080 if 'testnet', 38080 if 'stagenet')
                                        Port for p2p network protocol (IPv6)
  --p2p-use-ipv6                        Enable IPv6 for p2p
  --p2p-ignore-ipv4                     Ignore unsuccessful IPv4 bind for p2p
  --p2p-external-port arg (=0)          External port for p2p network protocol
                                        (if port forwarding used with NAT)
  --allow-local-ip                      Allow local ip add to peer list, mostly
                                        in debug purposes
  --add-peer arg                        Manually add peer to local peerlist
  --add-priority-node arg               Specify list of peers to connect to and
                                        attempt to keep the connection open
  --add-exclusive-node arg              Specify list of peers to connect to
                                        only. If this option is given the
                                        options add-priority-node and seed-node
                                        are ignored
  --seed-node arg                       Connect to a node to retrieve peer
                                        addresses, and disconnect
  --tx-proxy arg                        Send local txes through proxy:
                                        <network-type>,<socks-ip:port>[,max_con
                                        nections][,disable_noise] i.e.
                                        "tor,127.0.0.1:9050,100,disable_noise"
  --anonymous-inbound arg               <hidden-service-address>,<[bind-ip:]por
                                        t>[,max_connections] i.e.
                                        "x.onion,127.0.0.1:18083,100"
  --ban-list arg                        Specify ban list file, one IP address
                                        per line
  --hide-my-port                        Do not announce yourself as peerlist
                                        candidate
  --no-sync                             Don't synchronize the blockchain with
                                        other peers
  --enable-dns-blocklist                Apply realtime blocklist from DNS
  --no-igd                              Disable UPnP port mapping
  --igd arg (=delayed)                  UPnP port mapping (disabled, enabled,
                                        delayed)
  --out-peers arg (=-1)                 set max number of out peers
  --in-peers arg (=-1)                  set max number of in peers
  --tos-flag arg (=-1)                  set TOS flag
  --limit-rate-up arg (=2048)           set limit-rate-up [kB/s]
  --limit-rate-down arg (=8192)         set limit-rate-down [kB/s]
  --limit-rate arg (=-1)                set limit-rate [kB/s]
  --pad-transactions                    Pad relayed transactions to help defend
                                        against traffic volume analysis
  --rpc-bind-port arg (=18081, 28081 if 'testnet', 38081 if 'stagenet')
                                        Port for RPC server
  --rpc-restricted-bind-port arg        Port for restricted RPC server
  --restricted-rpc                      Restrict RPC to view only commands and
                                        do not return privacy sensitive data in
                                        RPC calls
  --bootstrap-daemon-address arg        URL of a 'bootstrap' remote daemon that
                                        the connected wallets can use while
                                        this daemon is still not fully synced.
                                        Use 'auto' to enable automatic public
                                        nodes discovering and bootstrap daemon
                                        switching
  --bootstrap-daemon-login arg          Specify username:password for the
                                        bootstrap daemon login
  --rpc-bind-ip arg (=127.0.0.1)        Specify IP to bind RPC server
  --rpc-bind-ipv6-address arg (=::1)    Specify IPv6 address to bind RPC server
  --rpc-restricted-bind-ip arg (=127.0.0.1)
                                        Specify IP to bind restricted RPC
                                        server
  --rpc-restricted-bind-ipv6-address arg (=::1)
                                        Specify IPv6 address to bind restricted
                                        RPC server
  --rpc-use-ipv6                        Allow IPv6 for RPC
  --rpc-ignore-ipv4                     Ignore unsuccessful IPv4 bind for RPC
  --rpc-login arg                       Specify username[:password] required
                                        for RPC server
  --confirm-external-bind               Confirm rpc-bind-ip value is NOT a
                                        loopback (local) IP
  --rpc-access-control-origins arg      Specify a comma separated list of
                                        origins to allow cross origin resource
                                        sharing
  --rpc-ssl arg (=autodetect)           Enable SSL on RPC connections:
                                        enabled|disabled|autodetect
  --rpc-ssl-private-key arg             Path to a PEM format private key
  --rpc-ssl-certificate arg             Path to a PEM format certificate
  --rpc-ssl-ca-certificates arg         Path to file containing concatenated
                                        PEM format certificate(s) to replace
                                        system CA(s).
  --rpc-ssl-allowed-fingerprints arg    List of certificate fingerprints to
                                        allow
  --rpc-ssl-allow-chained               Allow user (via --rpc-ssl-certificates)
                                        chain certificates
  --disable-rpc-ban                     Do not ban hosts on RPC errors
  --rpc-ssl-allow-any-cert              Allow any peer certificate
  --rpc-payment-address arg             Restrict RPC to clients sending
                                        micropayment to this address
  --rpc-payment-difficulty arg (=1000)  Restrict RPC to clients sending
                                        micropayment at this difficulty
  --rpc-payment-credits arg (=100)      Restrict RPC to clients sending
                                        micropayment, yields that many credits
                                        per payment
  --rpc-payment-allow-free-loopback     Allow free access from the loopback
                                        address (ie, the local host)
```

# Fail2Ban

Your server will always be under multiple attacks, especially SSH. It is recommended to install and configure Fail2Ban to mitigate these types of attacks. In our default configuration, for every three unsuccessful access attempts via SSH, occurring within the one minute time interval, then that IP address will be blocked for 24 hours. All communication originating from the banned IP will be simply dropped.

Use strong passwords, at least 24 characters or user `pwgen` to generate it.

```shell
% pwgen -By 24 1
ti7eufi.yeitahxech9goiCh
```

By editing the `env_variables` file you can set your custom values to define Fail2Ban behavior, including where to receive emails when an attacker is banned.

```bash
# Setup Fail2Ban (is important to avoid SSH attacks)
root@localhost:~# ansible-playbook 6-playbook-fail2ban.yaml
```

You can manually view banned ip's with:

```shell
root@node1:~# fail2ban-client status sshd
```

# Additional sudo user

It's recommended to have an additional privileged user in your server.

Please, edit the `env_variables` file and setup your local private and pub keys.

You can customize these settings in the `env_variables` file.

```bash
# Custom sudo user (recommended)
root@localhost:~# ansible-playbook 7-playbook-users.yaml
```

# Donation Address

Your support is important to keep this project. If you want to support, please send colaborations to the following Monero Address: 

**42SrpsLT5SuJZFCA9UKVp8cYMsYpmi66MczLkNobaJ9wQVKuiRwDYQd6jrwCPAHSjkVfthw8Jrv1JQdMWiqYdy1WRfphh7J**

Right now I'm setting four public monero nodes to support Monero Project. When first donation was received, I'll publish here the onion addresses for these servers (2021-12-07).
