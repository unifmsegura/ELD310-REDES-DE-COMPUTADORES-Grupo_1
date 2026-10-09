# Guia Completo: Projeto de Interconexão de Redes no Cisco Packet Tracer

## 📋 Resumo do Projeto

Você precisa criar 3 LANs com subredes específicas, configurar serviços (DHCP, DNS, HTTP) e implementar controle de acesso (ACLs).

---

## 🔧 PARTE 1: CÁLCULO DE SUBREDES

### LAN1 (Amarela) - 192.168.1.0/24

**Sub-rede 01** (110 máquinas):
- Máscara: `/25` (255.255.255.128)
- Rede: 192.168.1.0
- Primeiro host: 192.168.1.1
- Último host: 192.168.1.126
- Broadcast: 192.168.1.127

**Sub-rede 02** (31 máquinas):
- Máscara: `/26` (255.255.255.192)
- Rede: 192.168.1.128
- Primeiro host: 192.168.1.129
- Último host: 192.168.1.190
- Broadcast: 192.168.1.191

---

### LAN2 (Azul) - 172.16.0.0/16

Dividir em 4 subredes iguais (use as 2 primeiras):

**Sub-rede 01**:
- Máscara: `/18` (255.255.192.0)
- Rede: 172.16.0.0
- Primeiro host: 172.16.0.1
- Último host: 172.16.63.254
- Broadcast: 172.16.63.255

**Sub-rede 02**:
- Máscara: `/18` (255.255.192.0)
- Rede: 172.16.64.0
- Primeiro host: 172.16.64.1
- Último host: 172.16.127.254
- Broadcast: 172.16.127.255

---

### LAN3 (Verde) - 10.0.0.0/18

**Única sub-rede** (sem divisão):
- Máscara: `/18` (255.255.192.0)
- Rede: 10.0.0.0
- Primeiro host: 10.0.0.1
- Último host: 10.0.63.254
- Broadcast: 10.0.63.255

---

## 🏗️ PARTE 2: TOPOLOGIA NO CISCO PACKET TRACER

### Passo 1: Criar a Topologia Básica

1. Abra Cisco Packet Tracer
2. Adicione **3 Routers Cisco 2911** (para Router1, Router2, Router3)
3. Conecte os routers em cascata:
   - **Router1 ↔ Router2** (via Serial DCE/DTE)
   - **Router2 ↔ Router3** (via Serial DCE/DTE)

### Passo 2: Configurar Conexões das LANs

**LAN1 (Amarela) - conectada ao Router1:**
- 1 Switch (para sub-rede 01 com PCs)
- 1 Switch (para sub-rede 02 com Servidores)

**LAN2 (Azul) - conectada ao Router2:**
- 2 Switches (cada um com PCs)

**LAN3 (Verde) - conectada ao Router3:**
- 1 Switch (com PCs, Laptops e Servidor DHCP)
- 1 Access Point (WPA2-PSK)

---

## ⚙️ PARTE 3: CONFIGURAÇÃO DOS ROUTERS

### Router1 - Configurações

```
enable
configure terminal

! Interface para LAN1
interface FastEthernet 0/0
ip address 192.168.1.254 255.255.255.128
no shutdown

interface FastEthernet 0/1
ip address 192.168.1.254 255.255.255.192
no shutdown

! Interface serial para Router2
interface Serial 0/0/0
ip address 10.0.0.1 255.255.255.252
clock rate 64000
no shutdown

! Configurar RIP
router rip
version 2
network 192.168.1.0
network 10.0.0.0
exit
```

### Router2 - Configurações

```
enable
configure terminal

! Interface serial de Router1
interface Serial 0/0/0
ip address 10.0.0.2 255.255.255.252
no shutdown

! Interface serial para Router3
interface Serial 0/0/1
ip address 10.0.0.5 255.255.255.252
clock rate 64000
no shutdown

! Interface para LAN2
interface FastEthernet 0/0
ip address 172.16.0.254 255.255.192.0
no shutdown

interface FastEthernet 0/1
ip address 172.16.64.254 255.255.192.0
no shutdown

! Configurar RIP
router rip
version 2
network 10.0.0.0
network 172.16.0.0
exit
```

### Router3 - Configurações

```
enable
configure terminal

! Interface serial de Router2
interface Serial 0/0/0
ip address 10.0.0.6 255.255.255.252
no shutdown

! Interface para LAN3
interface FastEthernet 0/0
ip address 10.0.0.254 255.255.192.0
no shutdown

! Configurar RIP
router rip
version 2
network 10.0.0.0
network 10.0.0.0
exit
```

### Adicionar ACLs no Router3 (Bloquear LAN3 → LAN2)

```
access-list 101 deny ip 10.0.0.0 0.0.63.255 172.16.0.0 0.0.255.255
access-list 101 permit ip any any

interface FastEthernet 0/0
ip access-group 101 out
exit
```

---

## 📱 PARTE 4: CONFIGURAÇÃO DOS HOSTS

### LAN1 - Configuração Manual

**Sub-rede 01 (PCs):**
- PC1: 192.168.1.1 / 255.255.255.128 / Gateway: 192.168.1.254
- PC2: 192.168.1.2 / 255.255.255.128 / Gateway: 192.168.1.254
- PC3: 192.168.1.3 / 255.255.255.128 / Gateway: 192.168.1.254
- PC4: 192.168.1.4 / 255.255.255.128 / Gateway: 192.168.1.254

**Sub-rede 02 (Servidores):**
- Servidor HTTP: 192.168.1.129 / 255.255.255.192 / Gateway: 192.168.1.254
- Servidor DNS: 192.168.1.130 / 255.255.255.192 / Gateway: 192.168.1.254

### LAN2 - Configuração Manual

**Sub-rede 01:**
- PC1: 172.16.0.1 / 255.255.192.0 / Gateway: 172.16.0.254
- PC2: 172.16.0.2 / 255.255.192.0 / Gateway: 172.16.0.254

**Sub-rede 02:**
- PC3: 172.16.64.1 / 255.255.192.0 / Gateway: 172.16.64.254
- PC4: 172.16.64.2 / 255.255.192.0 / Gateway: 172.16.64.254

### LAN3 - Configuração DHCP

**Servidor DHCP:**
- IP estático: 10.0.0.1 / 255.255.192.0 / Gateway: 10.0.0.254

**Configurar DHCP no servidor:**
1. Clique no servidor
2. Vá até **Services** → **DHCP**
3. Configure o pool DHCP:
   - Default Gateway: 10.0.0.254
   - DNS Server: 192.168.1.130
   - Start IP Address: 10.0.0.10
   - Subnet Mask: 255.255.192.0

**Laptops e PCs - Receber IP por DHCP:**
1. Clique em cada dispositivo
2. Vá até **Desktop** → **IP Configuration**
3. Selecione **DHCP**

**Access Point - Configurar WPA2:**
1. Clique no Access Point
2. Vá até **Config** → **Wireless**
3. Configure SSID: `ProjetoRedes`
4. Authentication: **WPA2-PSK**
5. PSK Passphrase: `senha123`

---

## 🌐 PARTE 5: CONFIGURAR SERVIÇOS

### Servidor HTTP (LAN1)

1. Clique no **Servidor HTTP**
2. Vá até **Services** → **HTTP**
3. Clique em **index.html**
4. Edite a página (deixar a padrão do Cisco Packet Tracer)

### Servidor DNS (LAN1)

1. Clique no **Servidor DNS**
2. Vá até **Services** → **DNS**
3. Adicione registros:
   - **Name:** `index.html`
   - **Type:** `A Record`
   - **Address:** `192.168.1.129` (IP do servidor HTTP)

---

## ✅ PARTE 6: TESTES

### Teste 1: Conectividade com Ping

```
- De PC1 (LAN1) → PC1 (LAN2)
- De PC1 (LAN2) → PC1 (LAN3)
- De PC1 (LAN1) → PC1 (LAN3)
- De qualquer PC → Servidor DNS
```

### Teste 2: Acesso ao Servidor HTTP

1. Abra o **Web Browser** em qualquer PC
2. Digite: `http://index.html` ou `http://192.168.1.129`
3. Você deve ver a página padrão do Cisco Packet Tracer

### Teste 3: Testar ACL (Bloquear LAN3 → LAN2)

```
- De qualquer dispositivo em LAN3 → fazer ping para LAN2
- Resultado esperado: FALHA (bloqueado)
```

### Teste 4: DHCP em LAN3

```
- Laptops e PCs em LAN3 devem receber IPs automaticamente
- Range esperado: 10.0.0.10 até 10.0.63.254
```

---

## 📄 Entrega Final

1. **Salve o arquivo** como: `ProjetoRedes_Grupo1.pkt`
2. **Crie um PDF** com a tabela de subredes (conforme solicitado)
3. **Envie no Moodle:**
   - Arquivo `.pkt`
   - PDF do projeto de IPs

---

## 🎯 Checklist Final

- [ ] 3 Routers configurados com RIP
- [ ] LAN1 com 2 subredes e servidores
- [ ] LAN2 com 2 subredes e PCs
- [ ] LAN3 com DHCP, Access Point WPA2 e Servidor
- [ ] ACL bloqueando LAN3 → LAN2
- [ ] Ping funcionando entre LANs
- [ ] HTTP respondendo em todas as LANs
- [ ] DNS resolvendo nomes
- [ ] DHCP distribuindo IPs em LAN3
- [ ] Access Point com WPA2 ativo

---

**Boa sorte com o projeto! 🚀**
