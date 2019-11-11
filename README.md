# diy-vpn

A collection of scripts and config files to set up/run your own [OpenVPN](https://openvpn.net/) server and provision clients.

## Install
Clone the repo, `cd` into it, and `sudo bash install.sh`.

This will download and install OpenVPN and [easy-rsa](https://github.com/OpenVPN/easy-rsa). Then it will build the public-key infrastructure (PKI), create a certificate authority (CA), generate a key and sign a certificate for the VPN server. You'll be prompted for CA and server passwords during this process.

## Usage

### Configure
If clients will be using `client.conf`, you'll need to replace `<my-server>` (L42) with the hostname or address of your VPN server. You can modify other configuration options in `client.conf` and `server.conf` if you feel so inclined.

### Add a client
To generate credentials for a client, `cd` into the project directory on the host and run the following command:

```
$ sudo bash add-client.sh <username>
```

This creates a directory with `client.conf`, the CA certificate, a private key and certificate for the client. Then it zips and encrypts the directory with a password you provide.

The client will need to get the encrypted zip file `clients/<username>.zip`. If you have SSH access to the VPN host, `sftp` should work.

Once the file's on the client device, run the following commands:

```
$ unzip -d ~ <username>.zip
$ rm <username>.zip
```

Now you can find the client files at `~/openvpn`.

### Start
```
$ sudo systemctl start diy-vpn
```

This command enables IPv4 forwarding, sets an iptables rule to allow NAT routing, and then starts the VPN server.

### Stop
```
$ sudo systemctl stop diy-vpn
```

### View status
```
$ sudo systemctl status diy-vpn
```

### Remove a client
`cd` into the project directory on the host and run the following command:

```
$ sudo bash remove-client.sh <username>
```

This removes the .zip file from the clients directory and revokes the client certificate so it can no longer authenticate with the VPN server.

### Connect the client
Make sure you've added the client first!

#### On macOS
Download and install [Tunnelblick](https://tunnelblick.net/downloads.html) if you haven't already.

Then drag-and-drop the `client.conf`/`client.ovpn` file into "Configurations" on the left side of the app window. Tunnelblick should ask you a few questions and install the client configuration.

#### On Ubuntu
First, you'll want to install the Network Manager for OpenVPN.

```
$ sudo apt install network-manager-openvpn-gnome
```

1. Open "Settings" and click on "Network"
1. Add (+) a VPN and select the "OpenVPN" option
1. Name the VPN whatever you want
1. Under "General", enter the address/hostname of the VPN server for "Gateway"
1. Under "Authentication", choose the "CA certificate" `~/openvpn/ca.crt`
1. Choose the "User certificate" `~/openvpn/client.crt`
1. Choose the "User private key" `~/openvpn/private/client.key`
1. You can save your "User key password" or click the (?) on the right and select another option (e.g. "Ask for this password every time")
1. Click "Advanced" and then "TLS Authentication"
1. Under "Additional TLS authentication or encryption", select "TLS-Auth" for "Mode"
1. Choose `~/openvpn/private/ta.key` for "Key file"
1. Select 1 for "Key Direction" and click OK!

You should be able to connect to your VPN now!

## Contributing

Please do!

If you find a bug, think of an enhancement, or just have a question, feel free to open an issue.
