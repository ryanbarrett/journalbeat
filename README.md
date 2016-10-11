[![Build Status](https://travis-ci.org/mheese/journalbeat.svg?branch=master)](https://travis-ci.org/mheese/journalbeat)

# Journalbeat

Journalbeat is the [Beat](https://www.elastic.co/products/beats) used for log
shipping from systemd/journald based Linux systems. It follows the system journal
very much like `journalctl -f` and sends the data to Logstash/Elasticsearch (or
whatever you configured for your beat).

Journalbeat is targeting pure systemd distributions like CoreOS, Atomic Host, or
others. There are no intentions to add support for older systems that do not use
journald.

## Use Cases and Goals

Besides from the obvious use case (log shipping) the goal of this project is also
to provide a common source for more advanced topics like:
- FIM (File Integrity Monitoring)
- SIEM
- Audit Logs / Monitoring

This is all possible because of the tight integration of the Linux audit events
into journald. That said _journalbeat_ can only provide the data source for
these more advanced use cases. We need to develop additional pieces for
monitoring and alerting - as well as hopefully a standardized Kibana dashboard
to cover these features.

## Documentation

None so far. As of this writing, this is the first commit. There are things to
come. You can find a `journalbeat.yml` config file in the `etc` folder which
should be self-explanatory for the time being.

## Install

You need to install `systemd` development packages beforehand. In a
RHEL or Fedora environment, you need to install the `systemd-devel` package, `libsystemd-dev` in debian-based systems, et al.

`go get github.com/mheese/journalbeat`

**NOTE:** This is not the preferred way from Elastic on how to do it. Needs to
be revised (of course).

### Ubuntu 16.04

**Install required packages to build**

`sudo apt install libsystemd-dev`

`sudo apt install golang-go`

`nano .bashrc`
```
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```
**open a new shell.**

`go get github.com/mheese/journalbeat`

**Once built, move files to production locations (including other computers)**

`sudo cp ~/go/bin/journalbeat /bin/`

`sudo mkdir -p /etc/journalbeat/ && sudo cp ~/go/src/github.com/mheese/journalbeat/etc/journalbeat.yml /etc/journalbeat/`

**Adjust journalbeat.yml as needed**

`sudo nano /etc/journalbeat/journalbeat.yml`

**Create systemd unit file**

`sudo nano /lib/systemd/system/journalbeat.service`

```
[Unit]
Description=JournalBeat service

[Service]
ExecStart=/bin/journalbeat -c /etc/journalbeat/journalbeat.yml
StandardOutput=null

[Install]
WantedBy=multi-user.target
Alias=journalbeat.service
```

**Test service**

`sudo systemctl start journalbeat`

`sudo systemctl status journalbeat`

**If everything looks good**

`sudo systemctl enable journalbeat`





## Caveats

A few current caveats with journalbeat

### go-systemd

Journalbeat currently uses a forked version of [go-systemd](https://github.com/coreos/go-systemd). All changes should be merged back upstream to the repo of CoreOS, and I will work on pull requests soon.

### cgo

The underlying system library [go-systemd](https://github.com/coreos/go-systemd) makes heavy usage of cgo and the final binary will be linked against all client libraries that are needed in order to interact with sd-journal. That means that
the resulting binary is not really Linux distribution independent (which is kind of expected in a way).
