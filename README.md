# HA-Cookbook
High availability resources, examples and proposals


# LDAP HaProxy

````
# if self signed certificates, create a pem with crt and key
cat certs/ldap.ha-cert.pem certs/ldap.ha-key.pem > /etc/ssl/certs/ldap.ha/ldap.ha-cert-key.pem

# validate configuration
haproxy -c -V -f /etc/haproxy/haproxy.cfg

# run haproxy
service haproxy restart

# restart rsyslog
service rsyslog restart

````

LDAP Configuration
````
frontend ldap-636
        bind 192.168.27.27:636 ssl crt /etc/ssl/certs/ldap.ha/ldap.ha-cert-key.pem  no-sslv3 no-tlsv10 ciphers ECDH+AESGCM:DH+AESGCM:ECDH+AES256:DH+AES256:ECDH+AES128:DH+AES:ECDH+3DES:DH+3DES:RSA+AESGCM:RSA+AES:RSA+3DES:!aNULL:!MD5:!DSS
        mode tcp
        option socket-stats
        option tcplog
        option tcpka
        default_backend ldap-636-origin

backend ldap-636-origin
        # https://www.haproxy.com/documentation/hapee/1-8r1
        server DC-NODE-02 10.0.3.202:636 backup non-stick check ssl verify none fall 1 rise 2 inter 5000 weight 10
        server DC-NODE-01 10.0.3.201:636 check ssl verify none fall 1 rise 2 inter 5000 weight 10
        mode tcp
        balance leastconn
        stick-table type ip size 200k expire 30m
        timeout server 5s
        timeout connect 5s
        option tcpka
        option tcp-check
        tcp-check connect port 389
        tcp-check send-binary 300c0201 # LDAP bind request "<ROOT>" simple
        tcp-check send-binary 01 # message ID
        tcp-check send-binary 6007 # protocol Op
        tcp-check send-binary 0201 # bind request
        tcp-check send-binary 03 # LDAP v3
        tcp-check send-binary 04008000 # name, simple authentication
        tcp-check expect binary 0a0100 # bind response + result code: success
        tcp-check send-binary 30050201034200 # unbind request

````
