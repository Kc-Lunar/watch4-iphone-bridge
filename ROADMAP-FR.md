# ROADMAP — Galaxy Watch4 (SM-R870) + iPhone 13, 100 % gratuit, 100 % local

Objectif final : notifications iPhone sur la montre **et** données de la montre (pas, FC, sommeil)
synchronisées vers l'iPhone, visibles sur les deux, sans serveur, sans cloud, sans compte payant.

---

## Phase 0 — Fondations (TERMINÉ)

| Tâche | État |
|---|---|
| ADB sans fil opérationnel (`tools\wad.bat` → `192.168.0.71:38551`) | ✅ |
| JDK 17 + Android SDK 35 + Gradle 8.13 | ✅ |
| WearBridge compilé + patché (`FEATURE_TELECOM` absent sur montre BT) | ✅ |
| APK installé et lancé sur la montre | ✅ |
| Diagnostic : l'app iOS est obligatoire (filtre UUID `a0e3d5b1-…`) | ✅ |
| Workflow CI iOS (`.github/workflows/ios-build.yml`) | ✅ écrit |
| `project.yml` corrigé (team ID retiré, scheme partagé ajouté) | ✅ |

---

## Phase 1 — Compiler l'app iOS dans le cloud

| # | Tâche | Qui | Durée |
|---|---|---|---|
| 1.1 | Créer un compte GitHub | **Toi** | 5 min |
| 1.2 | Créer le repo `wearbridge-fork` **en public** (obligatoire pour min. macOS gratuites) | **Toi** | 2 min |
| 1.3 | M'authentifier pour pousser le code (token GitHub) | **Toi** + Moi | 5 min |
| 1.4 | Pousser le fork + le workflow | **Moi** | 5 min |
| 1.5 | Lancer le workflow, lire les erreurs, corriger | **Moi** | 10–20 min |
| 1.6 | Récupérer l'`WearBridge-unsigned.ipa` en artefact | **Moi** | 2 min |

**Livrable :** un fichier `.ipa` non signé, gratuit, sans Mac.

---

## Phase 2 — Signer et installer sur l'iPhone

| # | Tâche | Qui | Durée |
|---|---|---|---|
| 2.1 | Créer un **Apple ID secondaire** (jamais le principal) sur appleid.apple.com | **Toi** | 5 min |
| 2.2 | Activer le **mode développeur** sur l'iPhone (Réglages → Confidentialité et sécurité → Mode développeur → redémarrer) | **Toi** | 3 min |
| 2.3 | Installer **Sideloadly** sur Windows (sideloadly.io) | **Toi** | 3 min |
| 2.4 | Brancher l'iPhone en USB, faire confiance à l'ordinateur | **Toi** | 1 min |
| 2.5 | Glisser l'IPA dans Sideloadly, saisir l'Apple ID secondaire, `Start` | **Toi** | 3 min |
| 2.6 | Faire confiance au certificat : Réglages → Général → VPN et gestion de l'appareil | **Toi** | 1 min |
| 2.7 | Dépanner si besoin (bundle ID en conflit, certificat, compte bloqué) | **Moi** | — |

**Livrable :** WearBridge installé sur l'iPhone 13.
⚠️ Le certificat expire **tous les 7 jours** → refaire 2.5 chaque semaine (2 min), ou passer à AltStore pour l'automatiser.

---

## Phase 3 — Appairer et valider les notifications

| # | Tâche | Qui | Durée |
|---|---|---|---|
| 3.1 | Ouvrir WearBridge sur l'iPhone, laisser l'app au premier plan | **Toi** | 1 min |
| 3.2 | Sur la montre : app WearBridge → *Pair New Device* → *Start Scanning* → choisir l'iPhone → accepter l'appairage iOS | **Toi** | 1 min |
| 3.3 | Vérifier connexion + logs (`adb logcat`) | **Moi** | 5 min |
| 3.4 | Tester : message iMessage / WhatsApp / appel entrant | **Toi** | 2 min |
| 3.5 | Corriger ce qui ne marche pas (reconnexion, icônes, haptique) | **Moi** | — |
| 3.6 | Mesurer l'impact batterie sur 24 h | **Toi** | 24 h |

**Livrable :** notifications iPhone sur la montre. 🎉

---

## Phase 4 — Le vrai projet : le canal santé

| # | Tâche | Qui | Durée |
|---|---|---|---|
| 4.1 | Module Wear OS : `Health Services` → buffer `Room` (pas/FC/sommeil) | **Moi** | 1 j |
| 4.2 | Serveur GATT custom sur la montre (advertise + notify) | **Moi** | 0,5 j |
| 4.3 | UI montre Compose : aujourd'hui / semaine / état de synchro | **Moi** | 0,5 j |
| 4.4 | App iOS : central BLE + stockage local + SwiftUI (graphes) | **Moi** | 1 j |
| 4.5 | Protocole delta + watermark/ack (zéro perte, zéro doublon) | **Moi** | 0,5 j |
| 4.6 | Cadence : annonce 30 s / 10–15 min, pull, déconnexion | **Moi** | 0,5 j |
| 4.7 | Étendre le workflow CI, rebuild, réinstaller | **Moi** | 0,5 j |
| 4.8 | Tests réels, mesures batterie, ajustements | **Toi** + Moi | — |

**Livrable :** tes pas (puis FC/sommeil) visibles sur la montre **et** sur l'iPhone, sans cloud.

---

## Phase 5 — Autonomie

| # | Tâche | Qui |
|---|---|---|
| 5.1 | AltStore / AltServer sur Windows → refresh auto du certificat 7 j en Wi-Fi | **Toi** |
| 5.2 | Désactiver le débogage ADB quand on ne développe pas (sécurité) | **Toi** |
| 5.3 | Doc finale : installation, dépannage, réinstallation après reset | **Moi** |

---

## Points de blocage connus (à vérifier avant de commencer)

| Risque | Impact | Parade |
|---|---|---|
| **iOS de l'iPhone 13 < 26.0** (le projet cible iOS 26) | le build échoue | baisser `deploymentTarget` à 17.0 dans `project.yml` |
| Runner `macos-latest` sans Xcode 26 | build échoue sur le SDK | épingler un runner plus récent / adapter |
| Apple ID gratuit : 3 apps max, 10 App IDs / semaine, certificat 7 j | limite | un seul Apple ID dédié au sideloading |
| Apple ID gratuit banni (erreur `0xe8008024`) | signature refusée | utiliser un ID secondaire, jamais le principal |
| Montre : 1,6 Go libres, batterie 40 % | install lente / échec | charger la montre avant chaque install |
| Port ADB et IP changent au redémarrage de la montre | `wad.bat` obsolète | relancer `adb mdns services` puis mettre à jour le `.bat` |

---

## Ce qu'il me faut pour démarrer la Phase 1

1. **Version d'iOS de l'iPhone** → Réglages → Général → À propos → Version
2. **Compte GitHub** (créé ou existant) + un **token** (Settings → Developer settings → Personal access tokens → repo)
3. **Confirmation** que tu as bien un Apple ID secondaire (ou tu le crées en Phase 2)
