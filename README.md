# Real Estate Market Analysis in Morocco

Projet Fil Rouge — Formation Data Analyst, INT-Maroc

## Le projet en bref

L'idée de départ : aider un investisseur immobilier à savoir où investir au Maroc pour maximiser sa rentabilité locative. Le marché marocain est en pleine croissance, mais les données sont éparpillées entre Avito et Mubawab, sans aucune structuration. Ce projet construit le pipeline complet pour transformer ce bazar en quelque chose d'exploitable : de la collecte brute jusqu'à un dashboard décisionnel.

La problématique centrale : dans quelles villes et zones investir pour maximiser le ROI, en comparant location longue durée et vente ?

## Où j'en suis

### Collecte
Scraping de quatre sources distinctes :
- Avito : annonces de vente (`apparts_vente.csv`) et de location (`apparts_location.csv`)
- Mubawab : mêmes deux catégories (`mubawab_listings.csv` pour la vente, `mubawab_rentals.csv` pour la location)

Ces fichiers bruts vivent dans `Bronze/`. Rien n'y est touché, c'est la donnée telle que sortie du scraper.

### Nettoyage
Direction `Silver/`. Pour chaque source, j'ai dû régler plusieurs problèmes concrets avant de pouvoir comparer quoi que ce soit entre Avito et Mubawab :

- **Les prix** sont des chaînes de texte sales (`"1 050 000 DH"`, espaces insécables compris côté Mubawab) — converties en valeurs numériques propres.
- **Les surfaces** pareil, du texte type `"75 m²"` à parser, sauf côté Mubawab où c'était déjà numérique.
- **Le "1 DH"** qu'on retrouve dans une poignée d'annonces Avito n'est pas un vrai prix, c'est le marqueur que le vendeur a mis "prix sur demande". Traité comme une valeur manquante, pas comme un prix réel.
- **Les dates de publication** : Mubawab n'a pas de champ date direct, mais certains noms de fichiers image contiennent une date exploitable (captures WhatsApp, photos avec timestamp). Extraction par regex là où c'est possible — environ un tiers des annonces seulement, le reste reste vide plutôt que d'être deviné.
- **Ville et quartier** : chez Mubawab le champ `location` se découpe proprement en quartier + ville. Chez Avito cette info n'existe pas dans les fichiers actuels, donc le quartier reste vide pour cette source pour l'instant.
- **Doublons** : des milliers d'annonces réapparaissent d'un scrape à l'autre (même URL). Dédupliquées en gardant la version la plus récente.

Après nettoyage, les deux sources partagent exactement les mêmes colonnes avec les mêmes types — condition nécessaire avant de pouvoir les fusionner.

### Fusion
Une fois les schémas alignés, Avito et Mubawab sont concaténés en deux tables : une pour la vente, une pour la location, chacune avec une colonne `listing_type` pour ne pas perdre cette distinction.

### Une fausse piste évitée
Au départ je voulais détecter si un loyer était journalier ou mensuel, en pensant qu'il y avait peut-être des annonces de type Airbnb mélangées dans les données de location. En creusant les prix les plus bas du dataset, il s'est avéré qu'il ne s'agit pas de location courte durée mais de prix manquants ou mal scrapés (des appartements meublés à "150 DH/mois", ce qui n'a aucun sens). Pas de vraie location courte durée dans les données actuelles donc pas de classificateur à construire pour l'instant — un signal `price_suspect` suffit à isoler ces cas plutôt que d'inventer une distinction que la donnée ne supporte pas.

## Ce qui reste à faire

- Modélisation en schéma en étoile (table de faits + dimensions Ville, Type de bien, Temps) pour la couche `Gold`
- Mise en place PostgreSQL
- Calcul des KPI (prix moyen au m², loyer moyen, ROI estimé, classement des villes)
- Analyse exploratoire complète : distributions, valeurs aberrantes, corrélations
- Tests statistiques (t-test, corrélations)
- Dashboard Power BI
- Automatisation du pipeline

## Structure du repo

```
Bronze/     fichiers bruts issus du scraping, non modifiés
Silver/     données nettoyées, types corrigés, dédupliquées, schémas alignés
Gold/       à venir : modèle en étoile prêt pour la BI
```

## Notes pour moi-même

Le piège principal sur ce projet a été de vouloir aller trop vite sur les dates et de combler les valeurs manquantes par interpolation au lieu de simplement assumer qu'on ne sait pas. Une colonne remplie à 100% mais à moitié devinée est plus dangereuse qu'une colonne honnêtement incomplète, surtout sur un projet qui doit être défendu à l'oral.