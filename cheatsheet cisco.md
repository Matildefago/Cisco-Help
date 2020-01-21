# 🤯 Cheat sheet cisco 🤯

### Subnetting hacks

        /16 00000000          0 
    /25 /17 10000000        128
    /26 /18 11000000        192
    /27 /19 11100000        224
    /28 /20 11110000        240
    /29 /21 11111000        248
    /30 /22 11111100        252
        /23 11111110        254
        /24 11111111        255


### Configurazione ssh

1. Dare un nome al dispositivo
   ```
   switch(config)# hostname nome_dispositivo
   ```
   
2. Dare un nome al dominio
   ```
   switch(config)# domain nome_dispositivo
   ```

3. Generare le chiavi

    ```
    switch(config)# crypto key generate RSA
    ```

4. Definire database locale di utenti autorizzati

     ```
     switch(config)# username utente1
     switch(config)# password cisco1
     switch(config)# line vty 0 15
     switch(config)# transort input ssh
     switch(config-line)# login local
     switch(config-line)# do wr
     ```
### Configurazione RIP

```
router(config)# router rip
router(config)# network 20.0.0.0 //per ogni netwrk
router(config)# passive interface fa0/0
```

### VLSM(Con RIPV2)
```
router(config)# router rip
router(config)# version 2
```


### Configurazione NAT

1. Definire parte inside e outside, sul router dove viene configurato nat
   ``` 
   router(config)# interface fa0/0
   router(config-if)# ip nat inside
   router(config-if)# interface fa0/1
   router(config-if)# ip nat outside
   ```
   
2. Definire il pool di Indirizzi pubblici (anche solo 1) utilizzati per la traduzione

   ```
   router(config)# ip nat pool internet 150.0.0.1 150.0.0.1 netmask 255.255.0.0
   ```
   
3. Specificare chi è autorizzato alla traduzione 

   ```
   router(config)# access-list NUMERO deny/permit IPMACCHINA WILDCARD(OPPOSTO DELLE SMASK)
   router(config)# access-list NUMERO deny/permit any
   ```

4. Unire passi 2 e 3

   ```
   router(config)# ip nat inside source list 15 pool internet overload
   ```

5. RICORDARSI DI CREARE UNA DEFAULT ROUTE STATICA CHE DICE: SE NON SAI DOVE ANDARE VAI VERSO OUTSIDE 

   ```
        router(config)# ip route 0.0.0.0 0.0.0.0 fa0/0(nell'esempio RIP-NAT.pkt)
   ```


###  Aprire una porta nel router

   ```

        ip nat inside source static tcp 10.0.0.69(ip_hostinterno) 80(porta interna) 150.0.0.1 8080(porta esterna)
   ```

### Access List

Normali

   ```
router(config)# access-list (1-99) [deny | permit] [host (se solo 1 host) ip_host || wildcard(degli host da inserire) || any] 
   ```

Extended

   ```
router(config)# access-list (100-199) [deny | permit] [protocollo(es. tcp)][source_ip  wildcard(degli host da inserire)]  [destination_ip  wildcard(degli host da inserire)] [eq || neq || premi ?] [porta]
   ```


Named

Al posto del numero posso mettere un nome e valgono sia come standard access list sia come extended

### VTP

### Ether Channel

   ```

switch1(config)# interface range fa0/1-3
switch1(config-if-range)# channel-group 1 mode active
switch1(config-if-range)# interface port-channel 1
switch1(config-if)# switchport mode trunk
//switch1(config-if)# switchport trunk allowed vlan 1,2,20 //solo se voglio permettere solo ad   alcune vlan di passare
   ```


   ```
switch2(config)# interface range fa0/1-3
switch2(config-if-range)# channel-group 1 mode passive
switch2(config-if-range)# interface port-channel 1
switch2(config-if)# switchport mode trunk
   ```


Comandi controllo/verifica Ether Channel
   ```

switch1# show interface (Opzionale id interfaccio fa0/1) port-channel (Opzionale numero port-channel)1
switch1# show etherchannel summary
   ```


Se devo passare da interfaccia in 'on' a 'desirable' 
   ```

switch1(config)#no interface port-channel 1
switch1(config)#interface range fa0/1-2
switch1(config-if-range)#channel-group 1 mode desirable
switch1(config-if-range)#no shutdown
switch1(config-if-range)#interface port-channel 1
switch1(config-if)#switchport mode trunk
   ```


### Router Fallover(Router virtuale)

R1 Sarà il router più importante perchè ha priority 150 e con  preempt se non funziona, e dopo ritorna a funzionare ritorna lui in vigore

Router che voglio come principale

   ```

R1(config)# interface g0/1
R1(config-if)# ip address (ipaddress router 1) (subnetmask)
R1(config-if)# standby version 2 
R1(config-if)# standby 1 ip (ip router virtuale)
R1(config-if)# standby 1 priority 150
R1(config-if)# standby 1 preempt
R1(config-if)# no shutdown
   ```


Router(s) secondari di backup
   ```

R2(config)# interface g0/1
R2(config-if)# ip address (ipaddress router 2) (subnetmask)
R2(config-if)# standby version 2
R2(config-if)# standby 1 ip (ip router virtuale)
R2(config-if)# no shutdown
   ```
