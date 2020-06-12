# Viscosity OTP VPN Connection Helper

This command-line utility helps remove friction from the steps of
connecting to an OpenVPN server that requires an OTP second factor
authentication, using the Viscosity VPN client for MacOS.

The utility takes in a VPN connection's "short name" provided on the
command-line and looks for its full specification into a YAML
configuration file. It relies on
[viscosity.py](https://github.com/andreax79/viscosity.py) to interact
with the Viscosity VPN client, `pyotp` for generating the TOTP secret, a
bit of `osascript` and `pyperclip` to tie everything together.

## Installation

Install the Python dependencies:

```
$ pip install -r requirements.txt
```

If you plan on using an encrypted configuration file, you'll also need
GnuPG and a generated key pair. The use of the GnuPG Agent and a pin
entry tool like `pinentry` is recommended for easier operation.

## Configuration

By default, `viscosity-otp-connect` will look for a configuration file
at `~/.config/viscosity-otp-connect/specs.yaml`. This should be a YAML
file, optionally GPG-encrypted, containing the definitions of the
available Viscosity VPN connections.

For each configured connection in your Viscosity VPN client that you
wish to use with this helper, add a section like this to your
configuration file:

```yaml
my-vpn:
  connection: max@vpn.server.domain
  name: My sweet OTP-enabled VPN
  secret: TOTP-SECRET
```

The `connection` should match the name of the configured VPN in
Viscosity. The `secret` should be the base-32 TOTP secret used for the
OTP second factor authentication on this connection; it is commonly
found as the `secret` parameter of the `otpauth://` URI encoded in the
QR-code.

### Working with an encrypted config

Make sure your configuration file is GPG-encrypted:

```
$ cd ~/.config/viscosity-otp-connect/
$ gpg --encrypt --recipient <yourself> specs.yaml && rm -f specs.yaml
```

Note that `viscosity-otp-connect` will automatically try appending the
`.gpg` extension to the configuration file name if it is not found at
first, allowing it to correctly find
`~/.config/viscosity-otp-connect/specs.yaml.gpg` out of the box when
working with an encrypted config file.

## Usage

The most common usage is to simply connect to a named VPN:

```
$ viscosity-otp-connect my-vpn
```

If you need to specify another location for the configuration file, use
`-f`. If the file isn't encrypted, additionally pass `-p`:

```
$ viscosity-otp-connect -f other.yaml -p my-vpn
```
