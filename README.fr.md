# Portail captif MikroTik et supervision réseau avec The Dude

Déploiement d'un portail captif (Hotspot) sur un routeur MikroTik, associé à une supervision réseau en temps réel avec The Dude. Réalisé dans une petite entreprise de services informatiques (En tant que Technicien Supérieur, Réseaux et Sécurité informatique) à Douala, au Cameroun.

🇬🇧 [English version](README.md)

## Pourquoi ce projet ?

Un réseau Wi-Fi invité non contrôlé pose quatre problèmes : accès non autorisés, mauvaise répartition de la bande passante, absence de traçabilité des connexions, et risque de saturation et de pannes. Ce projet y répond avec une seule solution à faible coût : un routeur MikroTik qui authentifie et limite les utilisateurs, et un outil de supervision qui montre l'état des équipements et du trafic.

## Objectifs

- Authentifier les utilisateurs invités par un portail captif.
- Contrôler et limiter l'accès selon des profils prédéfinis.
- Limiter la bande passante par profil, de façon mesurable.
- Conserver les journaux de connexion et les statistiques de session.
- Superviser équipements et trafic en temps réel, avec notification lorsqu'un service tombe.

## Architecture

![Schéma d'architecture](docs/architecture.png)

Le Wi-Fi invité (hotspot) et le réseau local interne sont séparés sur des bridges distincts du même routeur. Le modem de l'opérateur alimente le port WAN du routeur.

| Réseau | Adresse | Rôle |
| --- | --- | --- |
| WAN (ether1) | 10.5.50.0/24 (NAT) | Liaison vers le modem |
| Bridge hotspot | 172.16.0.0/24 | Clients Wi-Fi invités |
| Bridge interne | 172.16.1.0/24 | Administration et serveurs |
| LAN | 192.168.10.0/24 | Réseau de bureau |

### Matériel et logiciels

- MikroTik RB951Ui-2HnD (MIPSBE), RouterOS 6.49.19 : hotspot, DHCP, NAT, pare-feu, User Manager.
- Modem Internet (box) et switch pour le réseau interne.
- PC d'administration avec VirtualBox hébergeant un routeur virtuel RouterOS CHR 7.20.8.
- WinBox, un navigateur web, client The Dude 7.20.8 (ports 2210 et 8291).

**Pourquoi un routeur virtuel ?** Le routeur physique fonctionnait sous une version de RouterOS antérieure à la 7.10, qui ne proposait pas certains services nécessaires à The Dude. Le serveur The Dude tourne donc sur une instance virtuelle RouterOS 7.20.8 sur le PC de l'administrateur, rattachée au bridge interne.

### Planification et coût

Le plan prévoyait environ 55 jours ouvrés : étude de l'environnement (5), achat du matériel (5), installation du routeur et du poste d'administration (10), mise en réseau (10), configuration du portail captif et de The Dude (20), tests (5). Le coût total estimé était d'environ 400 000 FCFA, dont 150 000 FCFA de main-d'œuvre. RouterOS et The Dude n'ajoutent aucun coût logiciel.

## Mise en œuvre

1. **Préparation du routeur.** Bridges pour le hotspot et le réseau interne, adressage IP, client DHCP sur le WAN, affectation des ports aux bridges.
2. **Serveur hotspot.** Un serveur hotspot sur le bridge hotspot avec son propre pool d'adresses. Le profil serveur autorise la connexion par cookie et par cookie MAC avec HTTP CHAP et PAP, pour une durée de cookie de 3 jours.
3. **Profil utilisateur.** Un profil avec délai de session d'1 heure, un utilisateur partagé par compte, une limite de débit de 512 kbps en émission et 1 Mbps en réception, et une page d'état affichée après connexion.
4. **Gestion des utilisateurs, deux méthodes.**
   - Utilisateurs hotspot locaux, chacun associé à un profil.
   - RADIUS via MikroTik User Manager (authentification sur le port 1812, comptabilité sur 1813), le routeur étant déclaré dans User Manager.
   - Tickets imprimables générés par User Manager : code QR, 1 heure de connexion, valables 2 heures, quota illimité, un seul utilisateur partagé.
5. **Page captive personnalisée.** Une page de connexion à l'image de l'entreprise, écrite en HTML, CSS et JavaScript, qui utilise le formulaire de connexion CHAP du hotspot.
6. **Isolation.** Des règles de pare-feu et de NAT séparent les clients du hotspot du réseau interne tout en autorisant l'accès à Internet.
7. **Supervision avec The Dude.** Paquet compatible avec la version du routeur, connexion du client, puis découverte du réseau (types d'équipements, réseaux, liens, structure de couche 2, temps d'interrogation des services). Carte du réseau, sondes de services avec graphiques d'historique, journal de notifications.

## Résultats

- **Côté client.** Un utilisateur qui rejoint le Wi-Fi ouvert est redirigé vers la page de connexion. Après authentification, une page d'état affiche l'adresse IP attribuée, les octets envoyés et reçus, et la durée de session.
- **Supervision.** The Dude découvre et cartographie les équipements : vert quand une sonde répond, rouge en cas de délai dépassé. Les graphiques d'historique montrent la latence dans le temps, et le journal de notifications enregistre, avec l'heure, quand un service tombe ou revient.
- **Observation.** Un graphique de latence montre un pic passager proche de 800 ms. La cause n'a pas été investiguée : ce dépôt n'en avance donc pas.

## Captures d'écran

**Carte du réseau dans The Dude.** Le routeur, les bridges hotspot et interne, et un client, avec le trafic en direct sur chaque lien.

![Carte du réseau dans The Dude](docs/dude-network-map.png)

**Historique d'un service dans The Dude.** Latence ping et NetBIOS dans le temps pour un équipement supervisé, avec le pic passager mentionné plus haut.

![Historique de latence dans The Dude](docs/dude-latency-history.png)

**Tickets imprimables.** Générés par User Manager : code QR, 1 heure de connexion, valables 2 heures, un seul utilisateur partagé. Tous les identifiants sont retirés.

![Tickets User Manager](docs/user-manager-vouchers.png)

## Limites et prochaines étapes

- Le routeur utilise une ancienne version de RouterOS 6 : prévoir la migration vers RouterOS 7.
- Le portail captif est servi en HTTP sans certificat configuré : ajouter HTTPS.
- Les secrets partagés et mots de passe doivent être robustes et renouvelés : les fichiers assainis de ce dépôt utilisent des valeurs factices.
- Le stockage des journaux est limité : ajouter un syslog distant et des règles de rétention.
- Les alertes restent dans The Dude : ajouter des notifications par e-mail ou messagerie.
- La configuration est appliquée à la main : l'exporter, la versionner et automatiser le déploiement (voir le prochain projet de mon portfolio : la configuration réseau comme code).

## Compétences démontrées

Administration RouterOS (bridges, DHCP, NAT, pare-feu, Hotspot, User Manager), authentification par RADIUS, contrôle de la bande passante, personnalisation d'une page captive, découverte et supervision réseau avec The Dude, schémas réseau, documentation technique.

## Structure du dépôt

```
├── README.md          documentation en anglais
├── README.fr.md       documentation en français
├── docs/              schéma d'architecture et captures assainies
├── portal/            page captive (HTML, CSS, JS) avec un logo générique
```

Les captures et exports de ce dépôt ne contiennent ni mots de passe, ni secrets partagés, ni noms d'équipements, ni données personnelles.

## Auteur

Josias Foguieng Kamga, 
[Portfolio](https://josias-kamga.github.io/portfolio/) · [GitHub](https://github.com/Josias-Kamga) · [LinkedIn](www.linkedin.com/in/josiaskamga)

## Licence

MIT, voir [LICENSE](LICENSE).
