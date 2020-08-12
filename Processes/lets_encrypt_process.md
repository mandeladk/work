# Overview

This document outlines the technical operating procedure for support engineers to utilize when updating SSL certificates on various client devices that are not visible on the public internet for automated certificate generation. The process describes a way for a support engineer to utilize tools that will help prevent problems when generating certficates. This includes dedicating a personal machine to the process, immobilizing the support engineer, as well as enabling hand-off if necessary to other support engineers.

# Helpful Tips

### Screen Overview
Screen is a terminal multiplexor. It allows an operator to run commands within a contained session, and escape back to the main window without losing access, or breaking running processes. For more information see:
https://www.gnu.org/software/screen/manual/screen.html#Overview

### Creating a New Screen Session
syntax:
```
screen -S someUniqueName
```

example:
```
screen -S web.nkarrick.com_Cert_Renewal
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
	5855.web.nkarrick.com_Cert_Renewal	(08/10/2020 11:36:53 AM)	(Detached)
1 Socket in /run/screen/S-stepcg.
stepcg@STEPcgOffice-Dashboard:~$ 
```

### Connect to Existing Screen Session
syntax:
```
screen -x desiredScreenName
```
note: use the `-d` option to attach to a session listed as already attached  
example:
```
screen -x web.nkarrick.com_Cert_Renewal
```

### Disconnecting temporarily from a Screen Session
press `ctrl`+`a` immediately followed by `d`

### Disconnecting/Deleting a screen session permanently
There are two ways to remove a screen session.

1. Type `exit` while connected to the screen session.
2. List screen sessions and kill the session by Process ID (PID). The PID will prepend the screen session name.
    ```
    stepcg@STEPcgOffice-Dashboard:~$ screen -ls
    There is a screen on:
        11641.web.nkarrick.com_Cert_Renewal	(08/10/2020 12:56:09 PM)	(Detached)
    1 Socket in /run/screen/S-stepcg.
    stepcg@STEPcgOffice-Dashboard:~$ kill 11641
    stepcg@STEPcgOffice-Dashboard:~$ 
    ```


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

# Example Process
1. Look for existing screen session.
    ```
    stepcg@STEPcgOffice-Dashboard:~$ screen -ls
    No Sockets found in /run/screen/S-stepcg.
    ```
    Create new one.
    ```
    stepcg@STEPcgOffice-Dashboard:~$ screen -S web.nkarrick.com_Cert_Renewal
    ```
2.  Issue Certbot command
    ```
    stepcg@STEPcgOffice-Dashboard:~$ sudo certbot certonly --manual --preferred-challenges dns -d web.nkarrick.com
    Saving debug log to /var/log/letsencrypt/letsencrypt.log
    Plugins selected: Authenticator manual, Installer None
    Obtaining a new certificate
    Performing the following challenges:
    dns-01 challenge for web.nkarrick.com

    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    NOTE: The IP of this machine will be publicly logged as having requested this
    certificate. If you're running certbot in manual mode on a machine that is not
    your server, please ensure you're okay with that.

    Are you OK with your IP being logged?
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    (Y)es/(N)o: Y

    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    Please deploy a DNS TXT record under the name
    _acme-challenge.web.nkarrick.com with the following value:

    YBP1QWqks0I2triKYHkQE5XpeP2f77OvpqoUJegKBhs

    Before continuing, verify the record is deployed.
    - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    ```
3. Pressed `ctrl`+`a` followed by `d` to leave screen session.
4. Verified existence of DNS TXT record using `dig`
    - Failed response
        ```
        stepcg@STEPcgOffice-Dashboard:~$ dig _acme-challenge.web.nkarrick.com TXT @8.8.8.8

        ; <<>> DiG 9.11.3-1ubuntu1.12-Ubuntu <<>> _acme-challenge.web.nkarrick.com TXT @8.8.8.8
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NXDOMAIN, id: 3043
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 512
        ;; QUESTION SECTION:
        ;_acme-challenge.web.nkarrick.com. IN	TXT

        ;; AUTHORITY SECTION:
        nkarrick.com.		254	IN	SOA	ns-cloud-c1.googledomains.com. cloud-dns-hostmaster.google.com. 119 21600 3600 259200 300

        ;; Query time: 14 msec
        ;; SERVER: 8.8.8.8#53(8.8.8.8)
        ;; WHEN: Mon Aug 10 12:34:42 EDT 2020
        ;; MSG SIZE  rcvd: 151
        ```
    - Succesful response
        ```
        stepcg@STEPcgOffice-Dashboard:~$ dig _acme-challenge.web.nkarrick.com TXT @8.8.8.8

        ; <<>> DiG 9.11.3-1ubuntu1.12-Ubuntu <<>> _acme-challenge.web.nkarrick.com TXT @8.8.8.8
        ;; global options: +cmd
        ;; Got answer:
        ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 20051
        ;; flags: qr rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

        ;; OPT PSEUDOSECTION:
        ; EDNS: version: 0, flags:; udp: 512
        ;; QUESTION SECTION:
        ;_acme-challenge.web.nkarrick.com. IN	TXT

        ;; ANSWER SECTION:
        _acme-challenge.web.nkarrick.com. 3599 IN TXT	"YBP1QWqks0I2triKYHkQE5XpeP2f77OvpqoUJegKBhs"

        ;; Query time: 19 msec
        ;; SERVER: 8.8.8.8#53(8.8.8.8)
        ;; WHEN: Mon Aug 10 12:36:07 EDT 2020
        ;; MSG SIZE  rcvd: 117
        ```
5. List and reconnect to screen session
    ```
    stepcg@STEPcgOffice-Dashboard:~$ screen -ls
    There is a screen on:
        9098.web.nkarrick.com_Cert_Renewal	(08/10/2020 12:25:50 PM)	(Detached)
    1 Socket in /run/screen/S-stepcg.
    stepcg@STEPcgOffice-Dashboard:~$ screen -x web.nkarrick.com_Cert_Renewal 
    ```
6. Press enter to continue
    ```
    Waiting for verification...
    Cleaning up challenges

    IMPORTANT NOTES:
    - Congratulations! Your certificate and chain have been saved at:
    /etc/letsencrypt/live/web.nkarrick.com/fullchain.pem
    Your key file has been saved at:
    /etc/letsencrypt/live/web.nkarrick.com/privkey.pem
    Your cert will expire on 2020-11-08. To obtain a new or tweaked
    version of this certificate in the future, simply run certbot
    again. To non-interactively renew *all* of your certificates, run
    "certbot renew"
    - If you like Certbot, please consider supporting our work by:

    Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
    Donating to EFF:                    https://eff.org/donate-le
    ```
7. `cat` contents of `fullchain.pem` (Edited for security reasons)
    The first listed certificate is the signed host certificate. Following certificates are intermediate certificates required for validation.
    ```
    stepcg@STEPcgOffice-Dashboard:~$ sudo cat /etc/letsencrypt/live/web.nkarrick.com/fullchain.pem
    -----BEGIN CERTIFICATE-----
    MIIFWDCCBECgAwIBAgISBBwq+8WtSOeNorMdkA9MzNSiMA0GCSqGSIb3DQEBCwUA
    MEoxCzAJBgNVBAYTAlVTMRYwFAYDVQQKEw1MZXQncyBFbmNyeXB0MSMwIQYDVQQD
    ...
    g71SWdPHvqDQGin2712K3ZXGEI3gbRQZlXnTE2Q+JLrrX4mEdAgBgpuPZ3VejSEM
    wJIa9DBLHTIFIZZyHQ2GrcaU7QpgF2t5aVXqlA==
    -----END CERTIFICATE-----
    -----BEGIN CERTIFICATE-----
    MIIEkjCCA3qgAwIBAgIQCgFBQgAAAVOFc2oLheynCDANBgkqhkiG9w0BAQsFADA/
    MSQwIgYDVQQKExtEaWdpdGFsIFNpZ25hdHVyZSBUcnVzdCBDby4xFzAVBgNVBAMT
    ...
    PfZ+G6Z6h7mjem0Y+iWlkYcV4PIWL1iwBi8saCbGS5jN2p8M+X+Q7UNKEkROb3N6
    KOqkqm57TH2H3eDJAkSnh6/DNFu0Qg==
    -----END CERTIFICATE-----
    ```
8. `cat` contents of `privkey.pem` (Edited for security reasons)
    ```
    stepcg@STEPcgOffice-Dashboard:~$ sudo cat /etc/letsencrypt/live/web.nkarrick.com/privkey.pem
    -----BEGIN PRIVATE KEY-----
    MIIEvgIBADANBgkqhkiG9w0BAQEFAASCBKgwggSkAgEAAoIBAQD0/p1ZPZn1JJUR
    Ke6SmaYeEHDrWuOTVN4xlH/2ClMVl6sGkxEEz/HD7rUoLj0Dae8phfwd1b+YNSKK
    ...
    3jBOX3ANtqNo6jd33oU0tEX/Y9nJN0Iimw+UA9Me9hTl98bysorfNA936fC8vn7v
    3xAjOLsf8UleMHwT4BMDCBOI
    -----END PRIVATE KEY-----
    ```