# Recon map: Fresha (web) — v1 brouillon

Scope (hypothèse) : la boucle cœur — un client réserve un service en ligne
auprès d'un établissement, l'établissement gère son agenda, son équipe et
encaisse. Marketplace, inventaire avancé et marketing automatisé : plus tard.
For : à confirmer (ton propre business, une niche, ou un produit à vendre).
Date : 2026-10-10

> v1 faite **uniquement à partir de sources publiques** (recherche web).
> Pas de captures d'écran ni de compte pro consulté. Les écrans et le modèle
> de données sont donc en partie déduits : chaque élément « guess » doit être
> confirmé avec captures (`replica/screens/`) ou ton propre compte.

## Sources

| # | source | URL | notes |
| --- | --- | --- | --- |
| 1 | page features officielle | https://fresha.com/for-business/features | agenda multi-appareils, réservation avec formulaires et prépaiement, checkout, liste d'attente, notifications auto, ressources, RDV de groupe, click & collect, automatisations fidélité |
| 2 | help center : politiques de paiement | https://www.fresha.com/help-center/knowledge-base/payments/613-payments-policies-overview | acompte, capture de carte, ciblage par groupe client / service / valeur du RDV |
| 3 | help center : configurer les politiques | https://www.fresha.com/help-center/knowledge-base/payments/101660-set-up-payment-policies | frais d'annulation par service, carte requise |
| 4 | presse : lancement des acomptes (2022) | https://www.salontoday.com/1080582/fresha-launches-new-deposits-feature | acompte déduit de la facture, conservé si no-show / annulation tardive |
| 5 | guide prix tiers (2026) | https://pabau.com/blog/fresha-pricing/ | Independent ~19,95 $/mois, Team ~14,95 $/membre/mois, 20 % (min 6 $) sur nouveaux clients marketplace |
| 6 | guide prix tiers (avr. 2026) | https://schedulingkit.com/pricing-guides/fresha-pricing | contredit #5 : cœur à 0 $ + frais de transaction. **À vérifier sur la page prix officielle** |
| 7 | avis Capterra | https://www.capterra.com/p/142138/Shedul-com/reviews/?page=2 | frais nouveaux clients, support |
| 8 | avis Trustpilot | https://www.trustpilot.com/review/fresha.com?page=2 | frais marketplace sur clients non apportés par Fresha |
| 9 | à faire | fiche App Store / Google Play « Fresha for business » | captures de chaque écran clé |
| 10 | à faire | ton propre compte Fresha (si tu en as un) | vérifier les CGU avant usage pour un concurrent |

## Core loop

Un client trouve l'établissement, choisit un service, un pro et un créneau,
confirme (avec acompte ou carte si requis) ; le pro voit le RDV dans son
agenda et encaisse après la prestation.

## Screens

| ID | screen | route / how to reach | purpose | key components | states seen |
| --- | --- | --- | --- | --- | --- |
| S01 | Page publique établissement | lien partagé, site, Google, réseaux | présenter services, équipe, adresse, avis | header, liste services, carte, bouton Réserver | guess |
| S02 | Choix du service | S01 → Réserver | sélectionner 1+ services | liste groupée par catégorie, prix, durée, panier | guess |
| S03 | Choix du professionnel | après S02 | choisir un pro ou « sans préférence » | cartes membres | guess |
| S04 | Choix date / heure | après S03 | créneaux dispo | sélecteur de date, grille de créneaux | guess (vide, complet) |
| S05 | Connexion / identification client | avant confirmation | identifier le client | email, téléphone, OAuth | guess |
| S06 | Récap + paiement | après S05 | politique d'annulation, acompte ou capture de carte | récap, formulaire carte, notes | source 2, 3 |
| S07 | Confirmation | après S06 | RDV confirmé, ajout au calendrier | récap, actions | guess |
| S08 | Mes rendez-vous (client) | lien email / compte | voir, annuler, replanifier | liste, actions | guess |
| S09 | Agenda pro | back-office | vue jour / semaine par membre et ressource | grille horaire, colonnes membres, blocs RDV | source 1 |
| S10 | Créer / éditer un RDV | clic sur S09 | RDV manuel (téléphone, walk-in) | modal, recherche client, service, membre | guess |
| S11 | Catalogue de services | paramètres | services, durées, prix, membres habilités | table, formulaire | guess |
| S12 | Équipe et horaires | paramètres | membres, horaires de travail, congés | planning hebdo | guess |
| S13 | Fiche client | recherche client | historique, notes, alertes | profil, timeline | guess |
| S14 | Caisse / checkout | RDV → Encaisser | services, produits, cartes cadeaux, pourboire | panier, moyens de paiement | source 1 |
| S15 | Paramètres réservation + politiques | paramètres | règles de réservation, acomptes, annulation | formulaires | source 2, 3 |
| S16 | Liste d'attente | back-office | clients en attente d'un créneau | liste, notification | source 1 |
| S17 | Rapports | back-office | ventes, RDV, performance équipe | tableaux, filtres | guess |
| S18 | Onboarding pro | inscription | établissement, horaires, premiers services | wizard | guess |

## Flows

```
F01 Client réserve en ligne
    S01 -> S02 -> S03 -> S04 -> S05 -> S06 -> S07
    happy path clicks: ~7 (à mesurer)
    edge: créneau pris pendant la saisie, fuseau horaire, plusieurs services
          enchaînés avec des pros différents, acompte refusé (carte déclinée)

F02 Client annule ou replanifie
    email -> S08 -> S04 -> S07
    edge: annulation hors délai -> acompte / frais retenus (source 3, 4)

F03 Pro crée un RDV (téléphone, walk-in)
    S09 -> S10 -> S09
    edge: chevauchement, membre en congé, client inexistant -> création à la volée

F04 Pro encaisse après la prestation
    S09 -> S14
    edge: acompte déjà payé déduit, pourboire, carte cadeau partielle, remboursement

F05 Onboarding d'un établissement
    S18 -> S11 -> S12 -> S15 -> S01 (lien public)

F06 Liste d'attente
    S04 (complet) -> S16 -> notification -> S06
```

## Components

| component | variants | states | used on |
| --- | --- | --- | --- |
| Button | primary, secondary, ghost, danger | default, hover, focus, disabled, loading | all |
| Service row | simple, avec options | sélectionné, indispo | S01, S02, S11 |
| Staff card | membre, « sans préférence » | sélectionné, indispo | S03, S12 |
| Date picker | semaine, mois | jour vide, complet | S04 |
| Slot grid | — | libre, pris, sélectionné | S04 |
| Calendar grid | jour, semaine, par membre, par ressource | vide, chargé, chevauchement | S09 |
| Appointment block | confirmé, arrivé, terminé, no-show, annulé | drag, resize | S09 |
| Modal / drawer | — | ouvert, erreur | S10, S14 |
| Card payment form | acompte, capture de carte | erreur, 3DS | S06, S14 |
| Toast | succès, erreur | — | all |

## Inferred data model

```
Business      id, name, address, timezone, currency, booking_settings (json)
              evidence: S01, S15        confidence: high

StaffMember   id, business_id, name, role, color, active
              evidence: S03, S09, S12   confidence: high

WorkingHours  id, staff_id, weekday, start_time, end_time   (+ TimeOff: staff_id, start_at, end_at)
              evidence: S12             confidence: medium

Service       id, business_id, category_id, name, duration_min, price_cents,
              buffer_min, staff_ids[], requires_card, cancellation_fee_cents
              evidence: S02, S11, source 3   confidence: high

Client        id, business_id, name, email, phone, notes, source (online|walkin|marketplace)
              evidence: S05, S13, sources 7-8 (« nouveau client »)   confidence: high

Appointment   id, business_id, client_id, status (booked|confirmed|arrived|completed|no_show|cancelled),
              start_at, end_at, channel, notes
AppointmentItem  appointment_id, service_id, staff_id, resource_id?, start_at, end_at, price_cents
              evidence: S09, F01 multi-services   confidence: medium

PaymentPolicy business_id, type (deposit|card_capture), amount|percent,
              non_refundable_after, scope (all|service|client_group|min_value)
              evidence: sources 2, 3, 4   confidence: high

Sale/Payment  id, appointment_id?, client_id, lines, tip_cents, method, status, refunded_cents
              evidence: S14            confidence: medium

GiftCard, Product, Membership, Resource, WaitlistEntry, Review
              evidence: source 1       confidence: guess (champs)
```

Relationships : Business 1-n StaffMember, Service, Client ; Appointment 1-n
AppointmentItem ; Service n-n StaffMember ; Appointment 0-1 Sale.

## Feature matrix

Voir `features.csv`. Must : 14, should : 10, could : 8, skip : 4.

## Out of scope (cannot or should not be cloned)

- La **marketplace** Fresha et son réseau de clients : c'est leur audience,
  pas une fonctionnalité.
- **Fresha Payments** comme processeur : on utilise Stripe (Checkout, Terminal plus tard).
- Avis clients existants, listings, contenu et marque Fresha.
- Financement / services financiers.

## Size

Screens 18, flows 6, entities ~13. Hard parts : calcul de disponibilité
(horaires × congés × services multi-pros × buffers), pas de double réservation
(contrainte DB), acomptes et frais d'annulation (Stripe), fuseaux horaires,
rappels SMS/email.

Size : **M** pour la boucle cœur (F01–F05), **L** avec caisse complète,
produits, cartes cadeaux, adhésions et marketing.
