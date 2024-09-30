# Documentation Technique pour la Connexion à Fidecoin  
## Protocole CAISSE_AP_V3.20

### 1. Introduction
Le protocole **CAISSE_AP_V3.20** facilite la communication entre la caisse enregistreuse (Caisse) et le terminal de paiement (TPE) dans le système Fidecoin. Ce document détaille les méthodes de connexion, le format des messages TLV (Type-Length-Value), et les intégrations disponibles via API.

### 2. Méthodes de Connexion
#### 2.1 Connexion Série RS232
- Utilise des connecteurs standard **DB9** ou **DB25**.
- Les messages sont encapsulés avec les balises `[STX]` et `[ETX]` et des contrôles de redondance longitudinale (LRC).

#### 2.2 Connexion IP Ethernet
- Communication TCP/IP sans chiffrement SSL entre la caisse et le TPE.
- La caisse initie la connexion et le terminal écoute sur le port **8888**.
- Nécessite la configuration d'une adresse IP statique locale pour le terminal.

#### 2.3 Connexion Wi-Fi
- Similaire à **IP Ethernet** mais utilise un réseau Wi-Fi. Le protocole TCP/IP reste le même.

#### 2.4 Connexion Bluetooth
- Fonctionne selon un modèle maître-esclave avec TCP/IP sur Bluetooth.

### 3. Format des Messages (TLV)
Le protocole utilise un format **TLV** (Type-Length-Value) où chaque message a un type, une longueur et une valeur. La longueur est définie par 3 caractères, et la valeur est codée en ASCII.

#### Exemple de message TLV :
```plaintext
CZ0040310CJ012247300123456CA00201CB0040300CD0011CE003978BH01118295910399BI020your.email@gmail.comCK003120
```
- **CZ0040310**  
  - **Type**: `CZ` (Version Protocol)  
  - **Length**: 004  
  - **Value**: 0125 (Version 3.10)

- **CJ012247300123456**  
  - **Type**: `CJ` (Cashbox Identifier)  
  - **Length**: 012  
  - **Value**: 247300123456

- **CA00201**  
  - **Type**: `CA` (Cashbox Number)  
  - **Length**: 002  
  - **Value**: 01 (Cashbox 01)

- **CB0040300**  
  - **Type**: `CB` (Amount)  
  - **Length**: 004  
  - **Value**: 0300 (Amount: 3.00 EUR)

- **CD0011**  
  - **Type**: `CD` (Action Type)  
  - **Length**: 001  
  - **Value**: 1 (Credit Action)

- **CE003978**  
  - **Type**: `CE` (Currency)  
  - **Length**: 003  
  - **Value**: 978 (Currency: EUR)

- **BH011133674790730**  
  - **Type**: `BH` (Client Phone Number)  
  - **Length**: 011  
  - **Value**: 33674790730

- **BI020your.email@gmail.com**  
  - **Type**: `BI` (Client Email)  
  - **Length**: 020  
  - **Value**: your.email@gmail.com

- **CK003120**  
  - **Type**: `CK` (Cashbox Receipt)  
  - **Length**: 003  
  - **Value**: 120 (Receipt reference)


#### 3.1 Règles d'Utilisation
- Les tags de longueur nulle ne sont pas transmis.
- Le tag de la version du protocole (CZ) doit être présent dans tous les messages.
- L'ordre des tags (sauf CZ) n'est pas critique.

### 4. Dictionnaire des Tags

| **Tag** | **Description** | **Obligatoire** | **Exemple de Contenu Valide** |
| ------- | ---------------- | --------------- | ----------------------------- |
| CZ      | Version du Protocole | Obligatoire | `CZ0040320` (Version 3.20) |
| CJ      | Identifiant du Protocole de la Caisse | Obligatoire | `CJ012247300123456` |
| CA      | Numéro de Caisse | Obligatoire | `CA00201` (Caisse 01) |
| CB      | Montant de la Transaction | Obligatoire | `CB003150` (1,50 EUR) |
| CD      | Type d'Action | Obligatoire | |
| CE      | Code de la Monnaie | Obligatoire | `CE003978` (EUR) |
| AE      | État de l'Action | Obligatoire | `AE00210` (Opération réalisée) |
| AF      | Complément d'État | Optionnel | `AF02904148HOLDER INVALID` |
| AA      | Numéro PAN | Optionnel | `AA0165017677122000674` |
| AB      | Date d'Expiration | Optionnel | `AB0041603` (Mars 2016) |
| AC      | Numéro d'Autorisation | Optionnel | `AC006A00395` |
| AD      | Piste CMC7 | Optionnel | `AD035D0416926D031017807908F011519431044B` |
| AG      | Réponse FNCI | Optionnel | `AG0040935` |
| AH      | Réponse de Garantie | Optionnel | `AH006123456` |
| AI      | AID de la Carte | Optionnel | `AI014A0000000421010` |
| AM      | Numéro de Téléphone du Client | Optionnel | `AM0100625698925` |
| AN      | Email du Client | Optionnel | `AN023example@gmail.com` |
| BB      | Demande d'Autorisation | Optionnel | `BB0011` (Autorisation obligatoire) |
| BF      | Acceptation de Paiement Partiel | Optionnel | `BF0011` (Paiement partiel accepté) |
| CI      | Mode de Lecture | Optionnel | `CI0011` (Lecture par puce) |
| CK      | Gestion de Ticket | Optionnel | `CK003100` (Ticket client en ASCII codifié en base 64) |
| ZT      | Ticket Construit | Optionnel | `ZT413` (Ticket de 413 octets) |
| OD      | Détails de la commande | Optionnel | Champ personnalisé pour les détails de commande à traiter par Fidecoin. |

#### Exemple pour OD :
```json
{
  "orderId": "string", 
  "orderDate": "string",
  "orderAmount": "string",
  "orderCurrency": "string",
  "orderDescription": "string",
  "orderReference": "string",
  "orderStatus": "string",
  "orderItems": [
    {
      "itemDescription": "string",
      "itemAmount": "string",
      "itemQuantity": "string",
      "itemTotalAmount": "string"
    }
  ]
}
```

#### Types d'Action (CD) :
- **CD0010** : Débit.
- **CD0011** : Crédit.
- **CD0012** : Annulation.
- **CD0013** : Consultation.
- **CD0014** : Préautorisation.
- **CD0015** : Réautorisation.
- **CD0015** : Fermeture de Lot.
- **CD0016** : Paiement de Service.
- **CD0017** : Transfert.

### 5. Exemple de Transaction Complète

#### Demande depuis la Caisse :
```plaintext
CZ0040300CJ012247300123456CA00201CB0042500CD0010CE003978CK003100OD556yJv[...]AwIn1dfQ==
```
- **CZ0040300** : Version du protocole 3.00.
- **CJ012247300123456** : Identifiant du protocole.
- **CA00201** : Numéro de caisse 01.
- **CB0042500** : Montant de 25,00 EUR.
- **CD0010** : Action de débit.
- **CE003978** : Code de la monnaie (EUR).
- **CK003100** : Ticket client en ASCII codifié en base 64.
- **OD556[...]1dfQ==** : Order Data (max 900 octets).

#### Réponse depuis le Terminal :
```plaintext
CZ0040300CJ012789300987456AE00210CA00201CB0042500CC003001CD0010CE003978CG0071999403AI014A0000000421010CI0011ZT413CK003100AK400[...]
```
- **CZ0040300** : Version du protocole 3.00.
- **AE00210** : État de l'action (réalisée).
- **CA00201** : Numéro de caisse 01.
- **CB0042500** : Montant de 25,00 EUR.
- **CC003001** : Application de paiement (carte bancaire).
- **AI014A0000000421010** : AID de la carte.
- **CI0011** : Mode de lecture (puce).
- **ZT413** : Ticket de 413 octets.

### Flux de Connexion
1. **Démarrage de la Connexion** : La Caisse initie la communication avec le TPE.
2. **Envoi du Message** : La Caisse envoie un message TLV au TPE.
3. **Traitement au TPE** : Le TPE traite la demande.
4. **Réponse du TPE** : Le TPE envoie une réponse à la Caisse.
5. **Finalisation de la Transaction** : Fin de la communication et prêt pour la prochaine transaction.

### 6. Intégration via API
- **RS232** ou **TCP/IP** pour les transactions en temps réel.
- **Chargement de fichiers** pour le traitement par lots.
- Intégration basée sur les transactions en format TLV.

### 7. Compatibilité avec les Versions Antérieures
Le protocole CAISSE_AP est rétrocompatible. Il est recommandé de vérifier la compatibilité des tags en fonction de la version utilisée.

### Contrôle des Versions

| **Version** | **Date** | **Description** | **Auteur** |
| ----------- | -------- | --------------- | ---------- |
| 1.0         | 23/09/2024 | Version initiale du document pour la connexion à Fidecoin via le protocole CAISSE_AP_V3.20 | Sergio Pichardo |
| 1.1         | 26/09/2024 | Saisie des détails de la commande | Enmanuel Decamps |
```
