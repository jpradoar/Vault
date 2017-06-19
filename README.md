<p align="center">
  <img src="VaultLogo.png"/><br>
</p>

OJO, ESTA EN TESTING...!!!


<pre>
VAULT is downloaded as a binary, so it does not need to be compiled and has no dependency.
It is only necessary to download the file, unzip it and run it. :)
In this document, security is not disabled, only in basic functionality.
</pre>

### INFRASTRUCTURE SCHEME

<pre>
Secure_server 	i-1eseeacacf1f523a2   t2.micro      172.31.27.65
Seure_client	  i-1eseeacacf1f523a3   t2.micro      172.31.27.66
</pre>



### INSTALATION AND CONFIGURATION PROCESS

<pre>
Download package
  wget https://releases.hashicorp.com/vault/0.6.5/vault_0.6.5_linux_amd64.zip

Unzip the file
  unzip vault_0.6.5_linux_amd64.zip
    Archive:  vault_0.6.5_linux_amd64.zip

Copy file to /bin/
  cp vault /bin/

check if it's Ok, and show the version
  vault -v
    Vault v0.6.5


Generate a config file, to the vault-server
/root/vault/vault.conf
    #!/bin/bash 
    backend "file" {
      address = "127.0.0.1:8500"
      path = "/tmp/vault/backend"
      mode = 0600
      format = "json"
    }
    listener "tcp" {
      address = "0.0.0.0:8200"
      tls_disable = 1
    }
    disable_mlock = true
</pre>



<pre>
Option 2: 
backend "inmem" {
}
listener "tcp" {
  address = "0.0.0.0:8200"
  tls_disable = 0
}
disable_mlock = true
</pre>



### STARTING PROCESS [ Vault-Server ]

<pre>
Start the server
    vault server -config="/root/vault/vault.conf"

Enable the vault log
    vault audit-enable file path=/var/log/vault_audit.log

Export the vault_addr
    export VAULT_ADDR=http://127.0.0.1:8200


Generate keys:
In this case, I'm generate six keys, and I create a threshold of three.  This threshold is the number of times to use for validate the connection
    vault init -key-shares=6 -key-threshold=3
	Unseal Key 1: XMvVFr+khzTz3vvtlV8fTG48APTgolNVx9HDE8NmoygB
	Unseal Key 2: o3FXJ+mtU5bAfN7O4ES8NAoKA+BBVCciyKuoZ3dquNUC
	Unseal Key 3: EMrdKP+lPcLaC991u8ryUg5ejVAhpfG6tKGIz4Xp/tMD
	Unseal Key 4: s9lT5YLqBfTP9CqExJ5PcxGpkUYdiI41R7BZcm+eNjIE
	Unseal Key 5: AGLZ6pTia6DVgys/nxABFRX9H/Z9eVitO7p52p0dcDQF
	Unseal Key 6: /9hb28LrvwLmIQ4c6guibXHLHOLcjyzaNMASrikRa8kG
	Initial Root Token: 8f5f5e35-b57e-fb53-d33d-87251b501a29

key-shares = amount  keys number
key-threshold = amount  of number of times to validate

In this case, is important declare that the generated keys and the root token are unique. You should not show or lose it.


Validate three times with any previous key
    vault unseal 
	Key (will be hidden):   XMvVFr+khzTz3vvtlV8fTG48APTgolNVx9HDE8NmoygB
	Sealed: true
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 1

    vault unseal 
	Key (will be hidden):   o3FXJ+mtU5bAfN7O4ES8NAoKA+BBVCciyKuoZ3dquNUC
	Sealed: true
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 2

    vault unseal 
	Key (will be hidden):   EMrdKP+lPcLaC991u8ryUg5ejVAhpfG6tKGIz4Xp/tMD
	Sealed: false
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 0



Check the status:
    vault status
	Sealed: false
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 0
	Version: 0.6.2
	Cluster Name: vault-cluster-a2a519d8
	Cluster ID: 69f5d32b-2931-746a-8132-d1ed594c4965



Export the token
export VAULT_TOKEN=8f5f5e35-b57e-fb53-d33d-87251b501a29



Write a secret:
    vault write secret/user_prueba password=pass_user_prueba 
      Success! Data written to: secret/ssh_vault_server



Read the created secret
    vault read secret/user_prueba
	Key             	Value
	---             	-----
	refresh_interval	768h0m0s
	password        	pass_user_prueba




----------------------------------------------------


Create a token
	vault token-create -policy="secret"
		Key            	Value
		---            	-----
		token          	805d878e-ebaa-efee-1f75-59cbb8128455
		token_accessor 	5ceaac2b-8721-3ec9-bf53-ad8d8f3990fd
		token_duration 	768h0m0s
		token_renewable	true
		token_policies 	[default secret]
</pre>


### CLIENT CONFIGURATION PROCESS [ Vault-Client ] 

<pre>
I'm assumed that vault is installed in the client and is the similar version...
Export the IP
    export VAULT_ADDR=http://172.31.27.65:8200


I validate three times, using the keys created in the server
    vault unseal 
	Key (will be hidden):   XMvVFr+khzTz3vvtlV8fTG48APTgolNVx9HDE8NmoygB
	Sealed: true
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 1

    vault unseal 
	Key (will be hidden):   o3FXJ+mtU5bAfN7O4ES8NAoKA+BBVCciyKuoZ3dquNUC
	Sealed: true
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 2

    vault unseal 
	Key (will be hidden):   EMrdKP+lPcLaC991u8ryUg5ejVAhpfG6tKGIz4Xp/tMD
	Sealed: false
	Key Shares: 6
	Key Threshold: 3
	Unseal Progress: 0


Export the token
    export VAULT_TOKEN=8f5f5e35-b57e-fb53-d33d-87251b501a29
    
        
Read the secret 
    vault read secret/ssh_vault_server
	Key             	Value
	---             	-----
	refresh_interval	768h0m0s
	ip              	172.31.27.65
	password        	jprado
	user            	root
    

    
Generate a secret
    vault write secret/ssh/root/Seure_client user=root password=jprado ip=172.31.27.66 tipo=vault_client
	Success! Data written to: secret/ssh/root/Seure_client
</pre>


### CASE OF USE
<pre>
A case of use, for example is:
A tree or path recursive to remember easily.  Of course it is a basic example but you need create your particular case in your infrastructure.

vault write secret/string1/string2/string3 Key=Value Key2=Value2 KeyX=ValueY

secret / {Application} / {Profile} / {TypeAuth} Key = Value             	 	=>         secret/racktables/jprado/http_login
secret / {Application} / {Profile} / {TypeAuth} Key = Value Key2=Value2 KeyX=ValueY    	=>         secret/TestSrv/jprado/ssh_jprado   user=jprado   password=jprado   ip=172.31.27.65

----------------------------------------

export VAULT_ADDR=http://172.31.27.65:8200
export VAULT_TOKEN=8f5f5e35-b57e-fb53-d33d-87251b501a29
vault unseal XMvVFr+khzTz3vvtlV8fTG48APTgolNVx9HDE8NmoygB
vault unseal o3FXJ+mtU5bAfN7O4ES8NAoKA+BBVCciyKuoZ3dquNUC
vault unseal EMrdKP+lPcLaC991u8ryUg5ejVAhpfG6tKGIz4Xp/tMD
</pre>







Fuentes:
<pre>
https://www.vaultproject.io/downloads.html
</pre> 

