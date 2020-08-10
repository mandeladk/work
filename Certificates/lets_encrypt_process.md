# Helpful Tips
### Creating a New Screen Session
syntax:
```
screen -S someUniqueName
```

example:
```
screen -S rxg.nkarrick.com_Cert_Renewal
```

### List Existing Screen Sessions
syntax:
```
screen -ls
```
example:
```
stepcg@STEPcgOffice-Dashboard:~$ screen -ls
There is a screen on:
	5855.rxg.nkarrick.com_Cert_Renewal	(08/10/2020 11:36:53 AM)	(Detached)
1 Socket in /run/screen/S-stepcg.
stepcg@STEPcgOffice-Dashboard:~$ 
```

### Connect to Existing Screen Session
syntax:
```
screen -x desiredScreenName
```
example:
```
screen -S rxg.nkarrick.com_Cert_Renewal
```

### Disconnecting from a Screen Session
press `ctrl`+`a` immediately followed by `d`

# Process
1. Connect to an existing session or create a screen session for this instance
    - Use human readable names
2. Execute the following command (substituting the FQDN)
    ```
    certbot certonly --manual --preferred-challenges dns -d fully.qualified.domain.name
    ```
    note: you may be prompted for additional information such as
    - Country Code (two letter)
    - State (full name)
    - City/Locale (full name)
    - Organization
    - Department
    - Email
    - etc...
3. Create/update a DNS TXT record with the value presented on screen. _DO NOT_ press Enter to continue until verifying
    - The TXT record should be named `_acme-challenge.fully.qualified.domain.name` (substitue FQDN) and contain the string presented on screen.
4. Verify the DNS TXT record contains the expected string by:
    ```
    dig _acme-challenge.fully.qualified.domain.name TXT @8.8.8.8
    ```
    or
    ```
    nslookup
    server 8.8.8.8
    set type=txt
    _acme-challenge.fully.qualified.domain.name
    ```
5. Only continue once you have verified the TXT record existence. Resume the Screen session and press enter to continue.
6. Certbot will then notify you of the locations of 
    - Private Key File
        - ` /etc/letsencrypt/live/fully.qualified.domain.name/privkey.pem`
    - Signed Certificate and Certificate Chain File
        - `/etc/letsencrypt/live/fully.qualified.domain.name/fullchain.pem`
7. The first entry in the fullchain.pem file will be the signed certificate. Subsequent entries are considered _Intermediate_ certificates to be used accordingly.