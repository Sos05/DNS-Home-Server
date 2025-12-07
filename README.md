# DNS-Home-Server
### Serveur DNS empêchant toutes les publicités et les traceurs sur le réseau domestique.

## Pré-requis

- Raspberry Pi 4 à jour, connectée à Internet et avec accès root/sudo.
- Configurée en IP statique.
- Aucun service utilisant le port 80 (Apache ou Nginx désactivé ou supprimé si conflit).

---

## 1. Préparation du système

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install curl nano -y
```
Pour mettre à jour et installer les outils qui vont servir à l'installation et à la configuration comme curl pour installer Pi-hole et nano pour éditer des fichiers.

## 2. Vérification de l’IP et configuration réseau

Ensuite, vérifier son adresse IP avec `ip a` et regarder si elle est bien statique et si ce n'est pas le cas modifier le fichier de configuration suivant.

```bash
sudo nano /etc/dhcpcd.conf
```
Puis y mettre ces lignes avec les bonnes informations.

```bash
interface INTERFACE
static ip_address=IP_RPI/MASQUE
static routers=IP_BOX
static domain_name_servers=IP_BOX
```
Puis appliquer les modifications en redémarrant.

```bash
sudo systemctl restart dhcpcd
```

## 3. Installation de Pi-hole

Installation de Pi-hole à l'aide de la commande `curl`.

```bash
curl -sSL https://install.pi-hole.net | bash
```
La commande télécharge silencieusement le script d’installation de Pi-hole `curl -sSL` puis l’exécute directement via `bash` grâce au pipe `|`, sans l’enregistrer.

Il y aura plusieurs choix à faire durant l'installation :

- Sélectionner l'interface réseau : Celle écrite précédemment (Souvant eth0).

- Choisir pour DNS upstream : Cloudflare, Google ou Quad9.

- Activer l'interface web d’administration.

- Lighttpd sera installé automatiquement si aucun serveur web est actif sur le port 80.

## 4. Vérification du fonctionnement

Si l'installation a réussie, Pi-hole est en fonctionnement (running) et le nslookup ne montre pas d'erreur. 
```bash
systemctl status pihole-FTL
nslookup google.com
```

## 5. Accès à l’interface web

À la fin de l'installation, aller sur l'URL `http://<ip de la rpi>/admin` (l'interface web) et un mot de passe est demandé.
Le mot de passe se trouve dans les dernières lignes de sortie d'installation mais il peut être modifier via la commande suivante.

```bash
sudo pihole setpassword
```

Sur l'interface web, aller dans `Lists` et mettre des adresses de listes à bloquer, comme `https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts` mais il en existe d'autres.

Si le bloquage fonctionne, le nslookup devrait ramener vers 0.0.0.0 (à faire sur la Raspberry).

```bash
nslookup doubleclick.net 127.0.0.1
```

Maintenant, les publicités qui étaient présentes sur certains sites ne sont plus là et sur l'interface web on peut voir ce qui a été bloqué et où.

## 6. Maintenance et mise à jour

Mettre à jour Pi-hole et les listes :

```bash
pihole -up
pihole -g
```

Vérifier le service.

```bash
systemctl status pihole-FTL
```

Redémarrer le service DNS si nécessaire.
```bash
sudo systemctl restart pihole-FTL
```

## 7. Ressources et documentation

[Lien vers la documentation de Pi-Hole](https://docs.pi-hole.net/)

[Lien vers des blacklists utiles](https://discourse.pi-hole.net/t/best-blacklists/27393)
