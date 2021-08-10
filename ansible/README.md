
# Developer machine setup

## Vault password

Should be added as `.vault_pass` in project toplevel.

## SSH config to reach devices

The firmware units establish a reverse SSH tunnel to the gateway machine.
This lets one connect even if the firmware devices are not directly reachable on the internet,
ie behind NAT on 4G or WiFi/Ethernet. 

One can connect via the gateway as 'jumphost' by adding a SSH config like:
```
Host door1.dlock.trygvis.io
 	User USERNAME
	ProxyCommand=ssh dlock.trygvis.io nc localhost 2001
```
Where 2001 is 2000+$devicenumber

Then test it using 
```
ssh -t door2.dlock.trygvis.io bash
```

# Common tasks

## Deploy firmware to devices

    ansible-playbook firmware.yml -l dlock-0

## Deploy gateway update

    ansible-playbook gateway.yml

## Initialize a new firmware device

    ansible-playbook bootstrap.yml --extra-vars "hosts=dlock-99.local user=pi" --ask-pass

## Certbot

    certbot register --agree-tos -m $EMAIL
    certbot certonly -d $DOMAIN --webroot --webroot-path /var/www/html

## Checking mosquitto passwords

    python
    from passlib.apache import HtpasswdFile
    ht = HtpasswdFile("/etc/mosquitto/conf.d/dlock.passwords")
    ht.users()
    ht.check_password("dlock-gateway", "foo")

## Generating secure passwords

Use [pwgen](https://github.com/tytso/pwgen) or another appropriate tool to generate passwords for the system. Remember to make the passwords long enough. pwgen is available as a package for Debian. pwgen example:

    pwgen -nyc -1 16

this generates a password that is 16 characters long.

## Generate hashes

The hash string for `vault_http_api_credentials_temp` (the part after ';') is generated using the apikey.py script in the gateway directory with the password as input.

The hash strings for `vault_mqtt_passwd` (the part after ':') is generated using `printf <password> | python3 -c 'import crypt; print(crypt.crypt(input(), crypt.METHOD_SHA512))'` with the relevant password as input.