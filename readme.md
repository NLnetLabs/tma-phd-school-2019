# TMA 2019 RPKI Lab

Welcome to the TMA 2019 RPKI Lab. In this lab, you can log in to
student machines set up to run Bird, and learn about RPKI filtering.

## Objectives

* Enable RPKI filtering on your sessions
* Authorise your own announcements
* Stop hijacking space, or hijack a bit more, choose your path :)

Bonus:
* Traffic engineering, divide your traffic
* Peer with other students

For those interested in RPKI certificate and object structures,
you can use an [online tool](https://lapo.it/asn1js/#) to show
the structure of objects. 

## Lab structure

### AS65000: master

Name: rpki-workshop-master.do.nlnetlabs.nl

This is a transit AS that all student VMs connect to. This AS does
not do any RPKI filtering, and it does not announce any IP space of
its own. It accepts all prefixes sent by the student VMs and exports
all prefixes in the 192.168.0.0/16 space.

### AS65001: filter

Name: rpki-workshop-filter.do.nlnetlabs.nl

This is also a transit AS that all student VMs connect to. It is
set up the same way as AS65000, except that this AS does implement
RPKI filtering and rejects announcements with a status RPKI invalid.

This machine also hosts the RPKI CA and publishes it at
rsync://rpki-workshop-filter.do.nlnetlabs.nl/rpki/

### AS65101 - AS65018: student1-17

Name: rpki-workshop-student-$NR.do.nlnetlabs.nl

These are the student machines. Each is set up to connect to both
'master' and 'filter'. These ASes do not do any RPKI filtering by
default.

Student machines are allocated the following prefixes:

 NR | Prefix
 ---|---
  1 | 192.168.0.0/22
  2 | 192.168.4.0/22
  3 | 192.168.8.0/22
  4 | 192.168.12.0/22
  5 | 192.168.16.0/22
  6 | 192.168.20.0/22
  7 | 192.168.24.0/22
  8 | 192.168.28.0/22
  9 | 192.168.32.0/22
  10 | 192.168.36.0/22
  11 | 192.168.40.0/22
  12 | 192.168.44.0/22
  13 | 192.168.48.0/22
  14 | 192.168.52.0/22
  15 | 192.168.56.0/22
  16 | 192.168.60.0/22
  17 | 192.168.64.0/22

By default all machines announce these prefixes in full. In
addition to all hijack a /24 from another student.

The machines run an rsync daemon publishing the
`/home/rpki/repository` directory under
rsync://rpki-workshop-student-$NR.do.nlnetlabs.nl/rpki/.

Each has been given a certificate to for its own RPKI CA authorized to
publish objects for their prefix and ASN. The key pair for this CA is in
`/home/rpki/student-{1..18].{key,pub}`. The certificate is at
rsync://rpki-workshop-filter.do.nlnetlabs.nl/rpki/student-{1..18].cer. It
expects the manifest for the CA to be
rsync://rpki-workshop-student-$NR.do.nlnetlabs.nl/rpki/ca.mft.

## Instructions

### Login

Login details are handed out on paper at the start of the lab.

### Bird configuration

Student machines run Bird 2.0.4. Documentation for Bird can
be found [here](https://bird.network.cz/?get_doc&f=bird.html&v=20)

The bird config files are here:
* `/usr/local/etc/bird.conf`
* `/usr/local/etc/bird.conf.d/*.conf`

The 'rpki' user can stop/start and reconfigure bird. Some useful
commands:

  Command                 | Effect
  ------------------------|----------------------
  bird                    | Start the bird daemon
  birdc down              | Stop the bird daemon
  birdc configure         | Reload configuration
  birdc restart all       | Restart all systems (rpki config, sessions)
  birdc show route        | Show route information
  birdc show route all    | Same, more verbose

Bird is configured to import any prefixes tied to the "lo" interface.

### List / add / del prefixes

We made a small script that allows the 'rpki' user to list/add/del
prefixes on the "lo" interface:
```
  rpki@rpki-workshop-student-1:~$ prefix.sh 
  /usr/local/bin/do-prefix.sh add <prefix>
  /usr/local/bin/do-prefix.sh del <prefix>
  /usr/local/bin/do-prefix.sh list

  Where prefix is inside 192.168.0.0/16.
```

### Create an RPKI CA

You can create your RPKI objects using the `mkrpki` command line tool.
It has a man page, so you can find instructions by saying `man mkrpki`.

For your CA, you will need to create a CRL, one or more ROAs, and a
manifest. Place all objects in `/home/rpki/repository/` so they get
published by rsync.

To verify whether things work, you can run Routinator locally. It has been
set up with the correct TAL, so you can just do

```
$ routinator -vv vrps
```

To look at your objects, you can use the
[ASN.1 JavaScript Decoder](https://lapo.it/asn1js/) after copying to your
own machine.

