# Lab FSSO Coopercica

## Objetivo
Testar autenticação passiva e ativa no Fortigate utilizando o Windows AD como base de usuários.

As tarefas podem ser encontradas em https://app.asana.com/0/1199504416290173/1199529003438156/f

## Equipamentos usados
* Fortigate 40F
* VMware Server 6.5 (192.168.100.3)
* Switch Core (192.168.0.1)
* Firewall ASA (192.168.98.254)

## Configurações de rede
### LAN
**VLAN ID** 302<br/>
**Rede** 10.233.0.0/24<br/>
**Servidor Windows 2019** 10.233.0.7<br/>
**Host Windows 10** DHCP<br/>
**Fortigate 40F** 10.233.0.1<br/>
**Switch Core** 10.233.0.2

### WAN
**Rede** 10.233.1.0/24<br/>
**Fortigate 40F** 10.233.1.1<br/>
**Firewall ASA** 10.233.1.2

## Configurações de domínio
**Nome do domínio** coopercica.com<br/>
**Usuário admin** administrator<br/>
**Senha admin** 0PP0rtunity<br/>
**Usuario local** cassio<br/>
**Senha usuario local** 0PP0rtunity

## Configurações VMware Server
Networking >> Port groups >> Add port group
**Name** lab-coopercica<br/>
**VLAN ID** 302<br/>
**Virtual Switch** vSwitch0

Virtual Machines
2 máquinas virtuais criadas
**lab-coopercica-win10** Host Windows 10<br/>
**lab-coopercica-win2019** Servidor Windows 2019



## Configurações switch Core
```
vlan 302
 name lab-coopercica
!
interface GigabitEthernet1/0/7
 description # Fortigate (Lab-Coopercica)
 switchport access vlan 302
 switchport mode access
 switchport nonegotiate
 spanning-tree portfast edge
!
interface vlan 302
 no ip proxy-arp
 no ip unreachables
 no ip redirects
 ip address 10.233.0.2 255.255.255.0
 no shutdown
```

## Configurações firewall ASA
```
interface GigabitEthernet1/8
 nameif transito-fg
 security-level 0
 ip address 10.233.1.2 255.255.255.0
 no shutdown
```

## Servidor Windows 2019
Serviços Instalados
* DHCP
* AD
* DNS

### DHCP
**Escopo** 10.233.0.50 até 10.233.0.60<br/>
**Lease** 8 dias<br/>
**Gateway** 10.233.0.1<br/>
**DNS** 10.233.0.7<br/>
**Option 121** 192.168.0.0 mask 255.255.255.0 10.233.0.2<br/>

### Rota estática
```powershell
route add 192.168.0.0 mask 255.255.255.0 10.233.0.2 -p
```

## Windows 10
### Usuário local
**user** admin<br/>
**pass** 0PP0rtunity<br/>
A resposta para todas as perguntas de segurança de recuperação de senha é **oppti**

## Fortigate 40F
**Versão** 6.4.4<br/>
**Usuário** admin<br/>
**Senha** admin

```
config system interface
    edit "lan"
        set vdom "root"
        set ip 10.233.0.1 255.255.255.0
        set allowaccess ping https ssh http
        set type hard-switch
        set stp enable
        set device-identification enable
        set lldp-reception enable
        set lldp-transmission enable
        set role lan
    next
    edit "lan3"
        set vdom "root"
        set ip 10.233.1.1 255.255.255.0
        set allowaccess ping
        set type physical
        set lldp-reception enable
        set role wan
        set description "WAN"
    next
end
config router static
    edit 1
        set dst 192.168.0.0 255.255.255.0
        set gateway 10.233.0.2
        set device "lan"
    next
    edit 2
        set gateway 10.233.1.2
        set device "lan3"
    next
end
```
