# US-02 — Matrice de flux et politique de sécurité cible


**Politique par défaut :** Tout trafic inter-zone non explicitement autorisé est **refusé** et **journalisé**. Les réponses aux connexions autorisées sont permises par le suivi d'état du FortiGate.


## 2. Matrice des flux autorisés

| ID | Source | Destination | Service / Port | Action | Justification | Journalisation |
| :--- | :--- | :--- | :--- | :---: | :--- | :---: |
| **F-01** | Kali `192.168.100.10` | VIP FortiGate (publication WAF) | HTTP / TCP 80 | Autoriser | Accès web au lab depuis le réseau attaquant | Oui |
| **F-02** | Kali `192.168.100.10` | VIP FortiGate (publication WAF) | HTTPS / TCP 443 | Autoriser | Accès web chiffré au lab, si HTTPS est configuré | Oui |
| **F-03** | FortiGate (VIP) | WAF `10.10.10.10` | HTTP / TCP 80 | Autoriser | Transmettre les requêtes publiées au reverse proxy | Oui |
| **F-04** | WAF `10.10.10.10` | DVWA `10.10.20.10` | HTTP / TCP 80 | Autoriser | Le reverse proxy transmet les requêtes autorisées à l'application | Oui |
| **F-05** | FortiGate `10.10.10.1` | Wazuh `10.10.30.10` | Syslog / UDP 514 | Autoriser | Centraliser les logs du pare-feu | Oui |
| **F-06** | WAF `10.10.10.10` | Wazuh `10.10.30.10` | Syslog / UDP 514 | Autoriser | Centraliser les logs du WAF | Oui |
| **F-07** | Poste admin `10.10.30.100` | FortiGate `10.10.30.1` | HTTPS / TCP 443 | Autoriser | Administrer le FortiGate via son interface web | Oui |
| **F-08** | Poste admin `10.10.30.100` | WAF `10.10.10.10` | SSH / TCP 22 | Autoriser | Administration du serveur WAF | Oui |
| **F-09** | Poste admin `10.10.30.100` | DVWA `10.10.20.10` | SSH / TCP 22 | Autoriser | Administration du serveur hébergeant DVWA | Oui |
| **F-10** | Poste admin `10.10.30.100` | Wazuh `10.10.30.10` | HTTPS / TCP 443 | Autoriser | Accès au tableau de bord Wazuh | Oui |
| **F-11** | Kali `192.168.100.10` | FortiGate `192.168.100.254` | ICMP Echo | Autoriser | Vérifier la connectivité de base du lab | Oui |

### Remarques :
* La VIP et sa configuration exacte seront définies lors de la configuration du FortiGate.
* **F-02** est à activer lorsque le HTTPS frontal est configuré.
* **F-08** et **F-09** sont des flux d'administration ; ils ne doivent être autorisés que depuis le poste admin.
* Les ports indiqués sont les ports cibles proposés. Ils devront être vérifiés lors du déploiement.

---

## 3. Flux explicitement interdits

| ID | Source | Destination | Service | Action | Motif |
| :--- | :--- | :--- | :--- | :---: | :--- |
| **D-01** | WAN / Kali | DVWA `10.10.20.10` | Tous | Refuser | Empêcher l'accès direct à l'application |
| **D-02** | WAN / Kali | Wazuh `10.10.30.10` | Tous | Refuser | Protéger la supervision |
| **D-03** | WAN / Kali | Poste admin `10.10.30.100` | Tous | Refuser | Protéger l'administration |
| **D-04** | DMZ / WAF | MGMT, sauf Wazuh `UDP 514` | Tous | Refuser | Limiter les communications du WAF |
| **D-05** | APP / DVWA | WAN | Tous | Refuser | Empêcher les connexions sortantes non nécessaires |
| **D-06** | APP / DVWA | MGMT, sauf flux explicitement ajouté | Tous | Refuser | Isoler l'application de la zone de gestion |
| **D-07** | Toute zone | Toute autre destination non autorisée | Tous | Refuser | Appliquer le refus par défaut |

---

## 4. Politique cible sur le FortiGate

1. Créer une politique par flux autorisé, avec source, destination et service précis.
2. Éviter les services `ALL` et les destinations `ALL` dans les règles d'autorisation.
3. Activer la journalisation des sessions autorisées pertinentes.
4. Activer la journalisation des refus dans la règle de refus final.
5. Placer les règles spécifiques avant la règle de refus final.
6. Vérifier les règles avec des tests positifs et négatifs.

---

## 5. Validation

- [x] Chaque flux autorisé a une justification et un service précis.
- [x] Le refus par défaut est explicite et journalisé.
- [x] Les flux interdits sont testés.
- [x] La matrice est relue et validée avant la phase 3.