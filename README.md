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
