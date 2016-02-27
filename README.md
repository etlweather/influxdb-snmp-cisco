influxdb-snmp-cisco
==========
[![Build Status](https://travis-ci.org/etlweather/influxdb-snmp-cisco.svg?branch=master)](https://travis-ci.org/etlweather/influxdb-snmp-cisco)

A tool to collect Cisco switch ports in/out Octets counters. Data pushed to 
InfluxDB is tagged with the *host*, *interface* name and *vlandId*.

**Note:** This is a very early stage, the code is basically a quick hack on top
of the original [influxsnmp](https://github.com/paulstuart/influxsnmp)
project. Improvements and code clean ups are welcome!

# Command line parameters

   - `config`: Path to the config file.
   - `freq`: Delay between polls (in seconds).
   - `http`: Http port to listen on for monitoring this program. Set to `0` to 
             avoid starting the monitoring web service.
   - `logs`: Path to logs directory.
   - `names`: Print interface names and exit.
   - `oids`: Path to the OIDs file.
   - `repeat`: Number of times to repeat - set to `0` to poll forever.
   - `testing`: Print data w/o saving.
   - `verbose`: Verbose mode.

# Configuration
## Defining hosts to check
A host is defined with an `[snmp]` entry. The header contains a pointer to the 
MIBs configuration to apply to the host. In the example below, "switch" is the
MIBs configuration.

```
[snmp "switch"]
host   = 192.168.1.1
community = public
port   = 161 
timeout = 20
retries = 5
repeat = 0
freq   = 30
portfile =  sample_ports.txt
```
A `portfile` can be set which defines which ports to record. If omitted, all 
ports are recorded.

## MIBs configuration
What to check for a host is configured as a "MIB" configuration. It is comprised
of a list of columns to read as shown in this example:

```
[mibs "*"]
name = ifXEntry
column = ifHCInOctets
column = ifHCOutOctets

[mibs "switch"]
name = ifXEntry
column = ifHCInOctets
column = ifHCOutOctets
```

The OID names must be defined in the OIDs file. In this example, the `"switch"` 
in the header is the name of the configuration. If a host is defined with
a MIBs configuration which isn't specifically defined, the MIBs configuration
with `"*"` will be used.

## InfluxDB output
The output to InfluxDB is configured with the `influx` configuration. Again,
a name can be provided in the header, or a `"*"` for default, so that different
database can be used for different host types.

```
[influx "*"]
host = influxdb.example.com
port = 8086
db   = perfmon
user = perfmon
password = perfmon
```


# OIDs 
The program requires MIB OIDs to be "pre-digested", e.g.,

```
snmptranslate -M $MIBDIR -Tz -On -m IF-MIB | sed -e 's/"//g' > oids.txt
```

The results from the above are included in the project as a start.

# Credits
This project is originally based on [influxsnmp](https://github.com/paulstuart/influxsnmp)
by [paulstuart](https://github.com/paulstuart).

