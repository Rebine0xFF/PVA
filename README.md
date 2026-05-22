**AI-generated summary of the project brainstorming session.**

---

# Pocket Voice Assistant

*Futur projet expérimental mêlant électronique embarquée, IA vocale et interaction minimaliste.*

## Concept

Créer un appareil portable, compact et rapide, pensé comme un “talkie-walkie intelligent”, spécialisé dans la conversion voix → texte à l’aide d’un modèle d’IA type Whisper.

L’objectif est de proposer une manière ultra fluide de :

* capturer une idée,
* prendre une note,
* retranscrire une phrase,
* ou enregistrer rapidement une information,

sans devoir sortir son téléphone ni ouvrir d’application. Le projet repose sur une philosophie simple : **réduire au maximum la friction entre une pensée et son enregistrement.**

---

## Fonctionnalités envisagées

### 1. Speech-to-text instantané

* Bouton “push-to-talk” et capture vocale rapide.
* Transcription automatique avec ponctuation et gestion du contexte.
* Support du bruit ambiant (grâce aux capacités de robustesse de Whisper).
* Affichage du texte sur écran ou envoi direct à un appareil connecté (PC/Smartphone).
* **Objectif :** Expérience ultra fluide, latence globale très faible (inférieure à 0.5s) et qualité de transcription maximale (équivalente au modèle Whisper Large).

### 2. Notes vocales intelligentes

Ajout instantané de notes structurées dans :

* Google Keep, Notion, Todoist ou d'autres systèmes de productivité personnels.
* *Exemple :* “Ajouter : acheter des composants ESP32” → création automatique d’une note textuelle triée via Webhook.

### 3. Mode cours / réunion

Mode de retranscription rapide permettant d’enregistrer une phrase dictée, de rattraper un passage manqué, ou de conserver un historique texte temporaire sur l’écran ou le PC d’accueil.

---

## Architecture technique envisagée

### Appareil portable (Le boîtier de poche)

Le boîtier physique contient le strict nécessaire pour maximiser l'autonomie et l'ergonomie :

* Microphone numérique (type I2S pour éviter les parasites analogiques).
* Bouton physique principal (Push-to-talk).
* Écran OLED minimaliste pour les retours d'état et le texte.
* Batterie rechargeable Li-ion avec circuit de protection.
* Connectivité Wi-Fi / Bluetooth.
* Retour sonore minimal (buzzer) ou LED d'état (RGB).

### Traitement IA déporté (Architecture Hybride)

Faire tourner un modèle Whisper Large directement dans un microcontrôleur de poche est impossible. L’architecture est donc décentralisée selon deux options :

```
[ Appareil de poche (ESP32) ]
             ↓ (Wi-Fi ou Bluetooth / Relais Smartphone)
[ Script PC d'accueil / Passerelle ]
             ↓
    ┌────────┴────────┐
    ▼                 ▼
[ Option A : CLOUD ] [ Option B : LOCAL ]
API Groq / OpenAI    Serveur local (Vulkan / AMD)

```

* **Option A : CLOUD (Prioritaire & Plug & Play) :** La passerelle reçoit l'audio et l'envoie instantanément à l'API Groq. La transcription revient en moins de 0,3 seconde.
* **Option B : LOCAL (Serveur Maison) :** Le PC de la maison fait tourner un serveur léger capable d'exploiter la carte graphique pour l'inférence locale.

---

## Réflexion sur l’inférence IA & Choix Hardware

* **Le problème du CPU :** Utiliser Whisper sur le processeur classique d'un PC est extrêmement lent et frustrant pour de la capture instantanée.
* **Le cas AMD débloqué :** L'inférence locale n'est pas réservée qu'à NVIDIA. Les cartes graphiques AMD sont parfaitement capables de transcrire Whisper à la vitesse de l'éclair à condition d'utiliser les bons outils comme `whisper.cpp` compilé avec **Vulkan**, ou des serveurs d'IA locaux modernes qui gèrent nativement l'accélération matérielle AMD.
* **L'atout ultime du Cloud (Groq) :** Pour passer d'un PC à un autre sans aucune configuration de pilotes graphiques complexes, l'externalisation via l'API Groq (LPU) s'impose comme la solution de référence en termes de vitesse de développement et d'exécution.

---

## Points importants, Coûts & Limites de l'API

L'analyse de l'API Groq pour le modèle **Whisper Large v3 Turbo** valide la viabilité économique et technique du projet :

### Viabilité en usage normal (Plan Gratuit)

Pour un usage de prise de notes quotidiennes, **le plan gratuit de Groq ne présente aucune limite contraignante dans la réalité**. Les quotas se structurent ainsi :

| Métrique de limite | Quota Plan Gratuit | Impact réel sur le projet |
| --- | --- | --- |
| **Poids du fichier** | **25 Mo** max par envoi | Largement suffisant pour des notes vocales de plusieurs minutes. |
| **Volume Horaire (ASH)** | **7 200 secondes** / heure | Permet de transcrire jusqu'à **2 heures d'audio cumulées par heure**, un seuil impossible à atteindre en dictée de notes simples. |
| **Volume Journalier (ASD)** | **28 800 secondes** / jour | Équivaut à **8 heures de dictée par jour**. |
| **Fréquence (RPM / RPD)** | 20 requêtes/min | 2 000/jour | Parfaitement calibré pour des flux de notes successives. |

> ⚠️ **Subtilité technique :** Groq applique un minimum de facturation de **10 secondes par appel**. Si le boîtier envoie un enregistrement de 3 secondes, le quota journalier/horaire sera déduit de 10 secondes. Cela reste totalement indolore face aux 8 heures quotidiennes disponibles.

### Évolutivité (Plan Payant)

Si le projet doit évoluer vers un mode "enregistrement continu" (réunions/cours complets), le passage au plan *Developer* payant élimine la barrière des 25 Mo (passage à 100 Mo) pour un coût dérisoire d'environ **0,04 $ par heure d'audio** (soit quelques centimes par mois).

---

## Technologies retenues

### Hardware

* **Microcontrôleur :** ESP32 (Wi-Fi/Bluetooth intégrés, faible coût, gestion du deep-sleep).
* **Audio :** Microphone MEMS I2S (ex: **INMP441**) pour un signal numérique propre.
* **Affichage :** Écran OLED I2C standard (SSD1306).
* **Énergie :** Batterie Li-ion + régulateur de charge (TP4056) + switch de coupure ou circuit d'auto-maintien.

### Software & Environnement de dev

* **IA / Transcription :** API Groq (`whisper-large-v3-turbo`) configurée en format texte brut et forcée en langue française (`fr`) pour optimiser la précision.
* **Code Embarqué :** C++ (Arduino IDE / ESP-IDF) pour capturer le flux I2S et le packager.
* **Passerelle PC de Test :** Script Python encapsulé dans un environnement virtuel isolé (`.venv`) pour garantir la portabilité du code sans polluer le système global, utilisant la bibliothèque officielle `groq`.

---

## État actuel du projet & Prochaines étapes

Le projet a validé son architecture théorique et ses choix financiers.

**Prochaine étape logique :** Mettre en place le dossier de développement, générer le `.venv` Python local, et exécuter le premier script de test d'envoi d'un fichier `.mp3` ou `.wav` vers l'API Groq pour mesurer la latence réelle de l'infrastructure Cloud.
