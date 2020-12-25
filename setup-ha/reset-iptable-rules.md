1. reset iptable ip6v4
   ```bash
    iptables -P INPUT ACCEPT
    iptables -P FORWARD ACCEPT
    iptables -P OUTPUT ACCEPT
    iptables -t nat -F
    iptables -t mangle -F
    iptables -F
    iptables -X
   ```
2. reset iptable ipv6
   ```bash
    ip6tables -P INPUT ACCEPT
    ip6tables -P FORWARD ACCEPT
    ip6tables -P OUTPUT ACCEPT
    ip6tables -t nat -F
    ip6tables -t mangle -F
    ip6tables -F
    ip6tables -X
   ```
