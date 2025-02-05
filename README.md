# Ghid Utilizare SSH pentru OpenStack

> Oficial: <https://ocw.cs.pub.ro/courses/isc/info/virtualmachine>



Topologie:


```
Localhost 🤓   ->   FEP   ->   OpenStack si guacamole.grid.pub.ro
```

## Setup


Generare cheie pentru fep

```sh
ssh-keygen -t ed25519 -N "" -f ~/.ssh/fep
```

> **FEP** = frontend processor machine 

Generare cheie pentru OpenStack

```sh
ssh-keygen -t ed25519 -N "" -f ~/.ssh/open-stack
```

Copiere cheie publica pe **fep**

```sh
ssh-copy-id -i ~/.ssh/fep.pub <moodle-username>@fep.grid.pub.ro
```

In terminal o sa apara un link (<https://login.upb.ro/auth/realms/UPB/device>) si un cod.
Copiaza codul in browser, apoi 2-factor-authentication, apoi da **Yes**.


Cheia publica pentru OpenStack se da paste in interfata grafica de pe OpenStack.




Tzeapa cu comanda `ssh -J moodle.username@fep.grid.pub.ro student@10.9.X.Y` ca nu merge asa usor pe **WSL** :(.



```sh
nano -l ~/.ssh/config
```


```conf
Host fep
  User <moodle-username>
  HostName fep.grid.pub.ro
  IdentityFile ~/.ssh/fep

Host open-stack
  User student
  IdentityFile ~/.ssh/open-stack
  ProxyJump fep
```

Pentru **fep**:

```sh 
ssh fep
```

Pentru **OpenStack**, e un pic mai complicat
(de vreme ce pt fiecare laborator/tema, vom avea un IP diferit pt VM):


```sh
ssh -o HostName=<IP-VM> open-stack
```


Solutia ar fi sa facem niste **"alias"**-uri:

```sh
nano -l ~/.bashrc    # sau ~/.zshrc or whatever
```


```sh
function open_stack_connect() {
	if [[ $# != 1 ]] ; then
		echo "[ERROR] Expected a single argument: the IP address of the OpenStack VM!"
		return
	fi

	IP=$1
	ssh -o HostName="$IP" open-stack
}


function open_stack_download() {
	if [[ $# != 2 ]] ; then
		echo "[ERROR] Expected 2 arguments:"
		echo "1. the IP address of the OpenStack VM"
		echo "2. the path to remote FILE"
		return
	fi

	IP=$1
	PATH_REMOTE_FILE=$2
	PATH_LOCAL_FILE="$(basename "$PATH_REMOTE_FILE")"
	rm -rf "$PATH_LOCAL_FILE"
	scp -o HostName="$IP" open-stack:"$PATH_REMOTE_FILE" "$PATH_LOCAL_FILE"
}



function open_stack_upload() {
	if [[ $# != 3 ]] ; then
		echo "[ERROR] Expected 2 arguments:"
		echo "1. the IP address of the OpenStack VM"
		echo "2. the path to LOCAL file"
		echo "3. the path to remote dir"
		return
	fi

	IP=$1
	PATH_LOCAL_FILE=$2
	PATH_REMOTE_DIR=$3
	scp -o HostName="$IP" "$PATH_LOCAL_FILE" open-stack:"$PATH_REMOTE_DIR"
}
```


> **Try and error** sa scri comenzile astea :).




## Exemple de utilizare

> Cu [setup](#setup)-ul de mai devreme, comenzile vor merge si pe **WSL** :).

```sh
open_stack_connect 10.9.4.210
open_stack_download 10.9.4.210 request.txt         # Descarca /home/student/request/txt
open_stack_download 10.9.4.210 info/hash.txt       # Descarca /home/student/info/hash.txt
open_stack_upload 10.9.4.210 response.txt /home/student/
open_stack_upload 10.9.4.210 response.txt /home/student/info/
```

> Descarcarile se vor face din directorul `/home/student/` de pe VM-ul remote.


## Conectare la **guacamole.grid.pub.ro**


Examenele practice (si de USO, si RL si ISC) le-am dat pe site-ul asta,
un terminal vai de el in browser, in care nu merge copy-paste-ul cu Ctrl-Shift-C/V.

Well, la exam **guacamole.grid.pub.ro** ne da un VM din aceeasi "oala" cu OpenStack-ul.
Conectarea se poate face si de pe **local**, cu aceleasi comenzi pentru OpenStack.


**Pasul 1**: obtinem OpenStackadresa IP de pe VM, ruland una din comenzile de mai jos

```sh
ip -br a
```

sau

```sh
ip -br a s dev eth0
```


**Pasul 2**: adaugam manual cheia publica pentru OpenStack in fisierul `~/.ssh/authorized_keys` de pe VM.

Pe VM:

```sh
nano -l ~/.ssh/authorized_keys  # sau vim
```

Acum ne putem conecta (din **localhost**):

```sh
open_stack_connect <IP-VM>
```



## Port Forwarding

> Nu alea cu `iptables` :).


For ISC try hards. 

Facem **port forwarding** cand avem un proces care ruleaza pe VM si vrem sa il accesam de pe local.

Caz concret: VM-ul ruleaza un serviciu web si e musai nevoie sa il si vedem in browser-ul nostru.
Alegem un port (random, nu prea conteaza) pe calculatorul nostru care sa fie mapat
la portul (asta e batut in cuie) pe care ruleaza procesul de pe VM.



Template:

```sh
ssh -J <moodle-username>@fep.grid.pub.ro -L <port-local>:<IP-VM>:<port-VM> -T -N student@<IP-VM> 
```

Accesarea se poate face acum local, din browser, la urmatorul URL: `localhost:<port-local>`.


Pe langa **port forwarding**, comanda va deschide si un shell, ncsf.


Exemplu:

```sh
ssh -J <moodle-username>@fep.grid.pub.ro -L 9090:10.9.4.210:8080 -T -N student@10.9.4.210 
```


In browser: `localhost:9090`.

