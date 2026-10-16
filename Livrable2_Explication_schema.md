# Commerces+ : explication du schéma de la base de données

Ce document accompagne le schéma de la base de données de Commerces+. Le schéma existe en trois versions : `MLD_Commerces+.drawio` (draw.io), `MLD_Commerces+.png` et `Livrable2_dbdiagram.dbml` (dbdiagram.io).

## Lire le schéma

Chaque rectangle est une table. Le nom de la table est dans le bandeau bleu, ses champs sont listés en dessous. Le champ souligné est la clé primaire : il identifie chaque ligne de façon unique. Les champs en rouge, précédés d'un #, sont des clés étrangères. Ils contiennent l'identifiant d'une ligne d'une autre table, et c'est ce qui crée le lien entre les deux.

Les traits relient chaque clé étrangère à la table qu'elle désigne. Le bout du trait indique la cardinalité. Une patte d'oie veut dire « plusieurs », une barre veut dire « un seul », et un rond veut dire que le lien peut être vide.

## Une table au centre : COMMERCE

Commerces+ héberge plusieurs dizaines de commerces dans la même base. La table COMMERCE est donc au centre du schéma, et presque toutes les autres tables portent un `#id_commerce`. C'est ce champ qui sépare les données de chaque commerce. À chaque requête, le serveur vérifie que l'identifiant du commerce correspond à celui de l'utilisateur connecté. Un commerçant ne peut donc pas lire les appels d'un autre commerce en changeant un numéro dans l'adresse.

COMMERCE contient aussi les réglages généraux du répondeur : le message d'accueil, le ton des réponses et l'état actif ou en pause.

## Les tables et leur rôle

UTILISATEUR contient les comptes qui se connectent à l'application. Le mot de passe y est stocké sous forme hachée, jamais en clair. Le champ `tentatives_echouees` sert à bloquer un compte après plusieurs essais ratés. Un commerçant est rattaché à son commerce. Un administrateur de la plateforme n'est rattaché à aucun commerce, donc son `#id_commerce` reste vide.

ABONNEMENT décrit la formule souscrite et son état. Un abonnement suspendu coupe le répondeur. Chaque commerce a exactement un abonnement, c'est pourquoi `#id_commerce` est unique dans cette table.

HORAIRE, FERMETURE, SERVICE, QUESTION_FREQUENTE et PLAGE_RDV contiennent tout ce que le commerçant saisit dans « Mon répondeur ». Le répondeur s'en sert pour répondre aux clients. J'ai séparé les horaires dans leur propre table plutôt que d'ajouter sept colonnes à COMMERCE. Un commerce peut ainsi avoir deux plages le même jour, par exemple le matin et l'après-midi avec une pause à midi.

APPEL garde la trace de chaque appel traité : la date, la durée, le numéro de l'appelant, la transcription, le résumé et un statut. Le champ `id_operateur` relie l'appel à celui de l'opérateur téléphonique. Le champ `date_suppression` sert à effacer automatiquement l'appel au bout de la durée de conservation prévue par le RGPD.

RENDEZ_VOUS contient les rendez-vous pris par le répondeur ou ajoutés à la main par le commerçant. Le champ `origine` fait la différence entre les deux.

DEMANDE_DEMO stocke les demandes envoyées depuis le formulaire du site public. Elle n'est reliée à aucune autre table, parce que la personne qui la remplit n'a pas encore de compte ni de commerce enregistré.

## Les relations

La plupart des relations partent de COMMERCE : un commerce a plusieurs horaires, services, appels ou rendez-vous, et chacun d'eux appartient à un seul commerce.

Trois relations méritent une explication. Un rendez-vous peut concerner un service, mais pas forcément : le commerçant peut noter un rendez-vous sans préciser la prestation. Un rendez-vous peut aussi venir d'un appel. Dans ce cas, `#id_appel` garde le lien, ce qui permet de relire la conversation. Ce champ est unique, puisqu'un appel ne crée qu'un seul rendez-vous, et il reste vide pour un rendez-vous ajouté à la main. La troisième relation est celle des utilisateurs, déjà expliquée plus haut : elle peut être vide pour un administrateur.

## Le lien avec le cahier des charges

| Fonctionnalité du cahier des charges | Tables utilisées |
|---|---|
| Configurer le répondeur | COMMERCE, HORAIRE, FERMETURE, SERVICE, QUESTION_FREQUENTE, PLAGE_RDV |
| Répondre aux appels et en garder la trace | APPEL |
| Prendre et gérer les rendez-vous | RENDEZ_VOUS, PLAGE_RDV, SERVICE |
| Comptes et connexion sécurisée | UTILISATEUR |
| Abonnement actif ou suspendu | ABONNEMENT |
| Formulaire de démonstration du site | DEMANDE_DEMO |
| Administration des commerces | UTILISATEUR (rôle admin), COMMERCE, ABONNEMENT |

## Les contraintes prévues

Un même créneau ne doit pas être réservé deux fois. La base vérifiera donc qu'aucun rendez-vous non annulé n'en chevauche un autre dans le même commerce. Ce contrôle sera fait dans la base elle-même, pas seulement dans l'application. L'adresse e-mail d'un utilisateur est unique, pour qu'un même compte ne soit pas créé deux fois. Quand un commerce est supprimé, toutes les lignes qui portent son `#id_commerce` sont supprimées avec lui, comme le prévoit le cahier des charges.

## Ce qui n'est pas dans la base

La base ne contient aucune coordonnée bancaire, puisque le paiement en ligne est hors périmètre. Elle ne garde ni l'adresse ni l'e-mail des clients des commerces : un nom et un numéro suffisent pour un rendez-vous. Les clés d'accès aux services d'IA et à l'opérateur téléphonique ne sont pas dans la base non plus, elles restent dans des variables d'environnement sur le serveur.
