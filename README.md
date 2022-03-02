# geoipblock

Block network traffic for IP addresses of specific countries. This manual
describes how to use xtables-addons to drop incoming and outgoing packages for
all or certain ports. See also
https://inai.de/projects/xtables-addons/geoip.php for more documentation.

## Ubuntu 21.10 with xtables-addons 3.18

Install packages with

    sudo apt-get install -y xtables-addons-common libtext-csv-xs-perl libnet-cidr-lite-perl

Create the file `/etc/cron.daily/xt_geoip` containing

    #!/bin/sh -e
    /usr/libexec/xtables-addons/xt_geoip_dl
    /usr/libexec/xtables-addons/xt_geoip_build -s

and give that file execution rights with

    sudo chmod a+x /etc/cron.daily/xt_geoip

## Ubuntu 20.04 LTS with xtables-addons 3.9

Install packages with

    sudo apt-get install -y xtables-addons-common libtext-csv-xs-perl libnet-cidr-lite-perl
    sudo chmod a+x /usr/lib/xtables-addons/xt_geoip_build
    sudo mkdir /usr/share/xt_geoip/

Create the file `/etc/cron.daily/xt_geoip` containing

    #!/bin/sh -e
    /usr/lib/xtables-addons/xt_geoip_dl
    /usr/lib/xtables-addons/xt_geoip_build -D /usr/share/xt_geoip/

and give that file execution rights with

    sudo chmod a+x /etc/cron.daily/xt_geoip

## Ubuntu 18.04 LTS with xtables-addons 3.0

Here xtables-addons uses only the maxmind geo IP database. However, that
database is now available under another URL than xtables-addons needs it to be.
Additionally, this version of xtables-addons is rather old to what is available.

This manual has not have a workaround for the database issue, but contributing
a workaround is welcome.

## Testing

Test the installation with

    sudo /etc/cron.daily/xt_geoip
    sudo modprobe xt_geoip
    lsmod | grep ^xt_geoip
    sudo iptables -m geoip -h

WARNING: The following commands can lock you and all others out of your machine!

Look up the country codes of the countries to block at https://db-ip.com/faq.php
and note there are also some additional codes available. Use the codes instead
of `XX,YY` below.

Block incoming packages by adding these rules

    #UNTESTEDiptables -I INPUT -m geoip --src-cc XX,YY -j DROP
    #UNTESTEDip6tables -I INPUT -m geoip --src-cc XX,YY -j DROP

Also block outgoing packages by adding these rules

    #UNTESTEDiptables -A OUTPUT -m geoip --dst-cc XX,YY -j DROP
    #UNTESTEDip6tables -A OUTPUT -m geoip --dst-cc XX,YY -j DROP

All rules can be listed with

    sudo iptables -L --line-numbers
    sudo ip6tables -L --line-numbers

Rules can be deleted with

    sudo iptables -D INPUT 1
    sudo iptables -D OUTPUT 1
    sudo ip6tables -D INPUT 1
    sudo ip6tables -D OUTPUT 1

where the number is the line number of the rule to delete.    

## Configuration

Make the iptables command persistent by first saving the current configuration
with

    iptables-save > rules
    ip6tables-save > rules6

This can result in an empty file or something that looks like

    # Generated by iptables-save ...
    *filter
    :INPUT ACCEPT [0:0]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [0:0]
    ...
    COMMIT
    # Completed on ...

Change that into

    *filter
    :INPUT ACCEPT [0:0]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [0:0]
    -I INPUT -m geoip --src-cc XX,YY -j DROP
    ...
    -A OUTPUT -m geoip --dst-cc XX,YY -j DROP
    COMMIT

(TODO add line :GEOIP - [0:0] directly after :OUTPUT ?)

(TODO example ip6tables)

Store and activate (TODO?) the new configuration with

    iptables-restore < rules
    ip6tables-restore < rules6

## Troubleshooting

Effect of the test or persistent configuration can be monitored with

    sudo iptables -L -v
    tail -f /var/log/kern.log 

## TODO

Omit (allow) specific tcp ports:
- https://superuser.com/questions/152829/block-all-ports-except-ssh-http-in-ipchains-and-iptables/152830#152830
- https://superuser.com/questions/319589/how-do-i-block-all-ports-except-22-and-80-in-fedoras-iptables/319675#319675
- https://superuser.com/questions/1310059/how-can-i-block-all-ports-except-22-80-443-to-all-incoming-traffic-except-localh/1310107#1310107
- https://superuser.com/questions/769814/how-to-block-all-ports-except-80-443-with-iptables/770191#770191
