


,,,,
➜  Ellingson nmap -sC -sV -vvv Ellingson.htb
Starting Nmap 7.80 ( https://nmap.org ) at 2020-05-05 07:06 EDT
Reason: 998 no-responses
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.6p1 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:e8:f1:2a:80:62:de:7e:02:40:a1:f4:30:d2:88:a6 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDNekDYOEF9YFIYWBGwCNM94Oy44bjNn9VlRp8oG8/a+yOHjo0sNd/aYksGIznnzYovF+0h6GDbM1dl/tX3eJgNy8Yil1BOe/sEYajT0vVn8BgJKcdHzAfA9AfpVwkQiJeigy2hcX7urDdoEi5L5uhjl8EqWU15m4bErudukQjmNeTGn1RgW88g8SKNOMI4/weDc2tI8G1J+ZLB/+wpcJe5gCdfnTyhucJqmeZzVy9lDHLpeXlET6Nx931KuJh6+ToXFVT5qB6yMuTnQRgn814uTABbEhxLUUWeFtOtCxLoAjQtpJrQLYaYPaVeewOHWDgDWvhPxYFIZFtiYw7wxsyf
|   256 c8:02:cf:a0:f2:d8:5d:4f:7d:c7:66:0b:4d:5d:0b:df (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBAmN0IVX/rflrwZLRjH3CC3nkAP5gXJwVUK3N7xctzWzR5IpPThLnVpjqGb9IwgROMmS8uTi0ZCQQ/9WSspATFg=
|   256 a5:a9:95:f5:4a:f4:ae:f8:b6:37:92:b8:9a:2a:b4:66 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIM8R/Ym8nOpVNvOnGkx6ndSdgJV1UWXwYaCu76M1HYNb
80/tcp open  http    syn-ack ttl 63 nginx 1.14.0 (Ubuntu)
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.14.0 (Ubuntu)
| http-title: Ellingson Mineral Corp
|_Requested resource was http://ellingson.htb/index
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

,,,,

Two ports are open:

SSH on 22
Apache 2.4.29 on 80
Browsing to openadmin.htb

------------------------
Browsing through page we see the team, possible usernames for our attack
hal, margo, eugene, duke, wallace, belford, ellingson

## Initial Foothold

we browsed to a page that didn’t exist, it triggered a python exception displaying Interactive Werkzeug Debugger

Let’s import os and then try to execute commands.

The application is running as hal user. We have access to “/home/hal/.ssh/authorized_keys”. We will add the public key and use the key to get ssh as hal user.

-------
Within Hal’s .ssh directory, we see a private key; unfortunately, it’s encrypted. Since we don’t have any creds to try, let’s add our key to the authorized_keys file.







