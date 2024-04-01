# SR-Lista-Firewall

## Regras Firewall
### Libere qualquer tráfego para interface de loopback no firewall.
iptables -A INPUT -i lo -j ACCEPT iptables -A OUTPUT -o lo -j ACCEPT
### Estabeleça a política DROP (restritiva) para as chains INPUT e FORWARD da tabela filter.
iptables -P INPUT DROP iptables -P FORWARD DROP
### Possibilite que usuários da rede interna possam acessar o serviço WWW, tanto na porta (TCP) 80 como na 443. Não esqueça de realizar NAT já que os usuários internos não possuem um endereço IP válido.
iptables -A FORWARD -i eth0 -o eth2 -p tcp --dport 80 -j ACCEPT iptables -A FORWARD -i eth0 -o eth2 -p tcp --dport 443 -j ACCEPT 
iptables -t nat -A POSTROUTING -o eth2 -s 10.1.1.0/24 -j MASQUERADE
### Faça LOG e bloqueie o acesso a qualquer site que contenha a palavra “games”.
iptables -A FORWARD -m string --string "games" --algo bm --to 65535 -j LOG --log-prefix "Blocked Games: " 
iptables -A FORWARD -m string --string "games" --algo bm --to 65535 -j DROP
### Bloqueie acesso para qualquer usuário ao site www.jogosonline.com.br, exceto para seu chefe, que possui o endereço IP 10.1.1.100.
iptables -A FORWARD -d www.jogosonline.com.br -j DROP 
iptables -A FORWARD -d www.jogosonline.com.br -s 10.1.1.100 -j ACCEPT
### Permita que o firewall receba pacotes do tipo ICMP echo-request (ping), porém, limite a 5 pacotes por segundo.
iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 5/s -j ACCEPT
### Permita que tanto a rede interna como a DMZ possam realizar consultas ao DNS externo, bem como, receber os resultados das mesmas.
iptables -A FORWARD -p udp --dport 53 -i eth0 -j ACCEPT 
iptables -A FORWARD -p udp --dport 53 -i eth1 -j ACCEPT
### Permita o tráfego TCP destinado à máquina 192.168.1.100 (DMZ) na porta 80, vindo de qualquer rede (Interna ou Externa).
iptables -A FORWARD -p tcp -d 192.168.1.100 --dport 80 -j ACCEPT
### Redirecione pacotes TCP destinados ao IP 200.20.5.1 porta 80, para a máquina 192.168.1.100 que está localizado na DMZ.
iptables -t nat -A PREROUTING -p tcp -d 200.20.5.1 --dport 80 -j DNAT --to-destination 192.168.1.100:80
### Faça com que a máquina 192.168.1.100 consiga responder os pacotes TCP recebidos na porta 80 corretamente.
iptables -A FORWARD -p tcp -s 192.168.1.100 --sport 80 -m state --state ESTABLISHED,RELATED -j ACCEPT
