###Log nf_conntrack results and ship logs to ELK stack
####Tested and working on CentOS 6 and 7 boxes

On CentOS 6 create file in directory of your choice and insert the code
(make sure your IP resolves or your host name is in  your hosts file):

```bash
#!/bin/bash
conns=$(cat /proc/sys/net/netfilter/nf_conntrack_count)
echo "$(date) $(hostname) $(hostname -i) nf_conntrack $conns" >> /var/log/conntrack
```

#####In CentOS 7 the script is slightly different

```bash
#!/bin/bash
conns=$(cat /proc/sys/net/netfilter/nf_conntrack_count)
echo "$(date) $(hostname) $(hostname -I)nf_conntrack $conns" >> /var/log/conntrack
```


#####in your logstash forwarder file, put the below snippet in:
```json
{
      "paths": [
        "/var/log/conntrack"
       ],
      "fields": { "type": "syslog" }
}
```


#####On your ELK stack box, put the below snippet in your syslog.conf file:


```json
grok {
        match => { "message" => "%{DATESTAMP_OTHER} %{HOSTNAME:host} %{IPV4:ip} %{SYSLOGPROG} %{BASE10NUM:cons:int}" }
        add_field => [ "received_at", "%{@timestamp}" ]
        add_tag => ["netfilter-connections"]
        }
        mutate {
                convert => { "cons" => "integer"}
}


