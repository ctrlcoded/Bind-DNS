# BIND DNS for Local Network

note: there need to be static IPs for this configuration to run for time being

- to make sure that requests are routed through the DNS you can:
    - Make the DNS server's ip the primary DNS solver on each device(or)
    - Configure DNS server's ip as primary resolver in the main network configuration

## SetUP
```sh
mkdir bind-dns
cd bind-dns
```
clone the repository and move into its directory
```sh
git clone https://github.com/Illucious/BIND-DNS-server.git
cd BIND-DNS-server-for-Local-Network 
```
### Changes in files
1. in 'named.conf' add the subnet of your local netowrk and server's IP
```cofig
acl internal {
    192.168.0.0/24; // add ip ranges of your local network and any other networks you want to allow queries from
    172.21.30.0/24; // also add the ip of your DNS server
    172.18.0.0/24; // add the IPs with subnet mask
    172.21.0.0/24;
};
```
2. change the zone name from 'example.home' as per your choice and also change the bind file name accordingly
```config
zone "example.home" IN {
    type master;
    file "/etc/bind/example-home.zone";
};
```
3. changes the values from 'example.home' to the required values.
    - ```ns          IN      A       172.21.30.44``` here change ip to th ip of your server
    - ```
        ; -- add dns records below

        suhani    IN      A       192.168.158.72
        ```
        change "suhani" and its ip to your desired values and add A name records as per your need

```config
$TTL 10d

$ORIGIN example.home.

@           IN      SOA     ns.example.home.        info.example.home (
                            2024052900      ; serial
                            12h             ; refresh
                            15m             ; retry
                            3w              ; expire
                            2h              ; minimum ttl 
                            )

            IN      NS      ns.example.home.
ns          IN      A       172.21.30.44

; -- add dns records below

suhani    IN      A       192.168.0.132
```


## Run the container
```bash
docker-compose up 
```
note: it will create two folder i.e. .cache and .records when the script is run for the first time. If they are not made and an error is thrown create them yourself.

## 
