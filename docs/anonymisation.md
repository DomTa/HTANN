# Documentation du Script d'Anonymisation JSON

## 1. Introduction

Ce document présente le développement itératif d'un script Python conçu pour anonymiser des fichiers JSON selon des règles spécifiques. Le script a été développé de manière collaborative avec des ajustements successifs pour répondre précisément aux besoins exprimés.

### Objectif du script
- Anonymiser les données sensibles dans des fichiers JSON
- Préserver la structure et la cohérence des données
- Appliquer des règles spécifiques selon le type d'objet
- Générer des logs détaillés pour la traçabilité

## 2. Spécifications Fonctionnelles

### 2.1 Structure du fichier JSON
Le script supporte deux types de structures principales :
- **Tableau d'objets** : `[{...}, {...}]`
- **Objet unique** : `{...}`

Le script détecte automatiquement la structure du fichier d'entrée.

### 2.2 Règles d'anonymisation
Les règles d'anonymisation varient selon le type d'objet identifié par la clé `"Type"` :

| Type | Anonymisation | Préfixe | Exceptions |
|------|---------------|---------|------------|
| ViewInfo | Oui | VI_ | - |
| OptionSetInfo | Oui | OSI_ | - |
| ServiceInfo | Non | - | Toujours |
| StaticDataSourceInfo | Non | - | Toujours |
| NativeCDSDataSourceInfo | Conditionnelle | TAB_ | Name="Utilisateurs" ou "Divisions" : non anonymisé |

### 2.3 Mapping des valeurs
Le script garantit la cohérence des remplacements :
- Une valeur donnée est toujours remplacée par la même valeur anonymisée
- Les références croisées entre objets sont préservées
- La structure hiérarchique est maintenue

## 3. Architecture Technique

### 3.1 Structure du code
Le script utilise une approche en deux passes principales :

1. **Première passe** : Analyse complète du JSON pour collecter tous les mappings
2. **Deuxième passe** : Application des transformations avec les mappings établis

### 3.2 Algorithmes
- **Détection de structure** : Analyse automatique du type de données racine
- **Collecte des mappings** : Parcours récursif pour identifier toutes les valeurs à anonymiser
- **Application des transformations** : Remplacement des valeurs selon les mappings établis

### 3.3 Gestion des cas complexes
- **Dictionnaires imbriqués** : Traitement récursif de tous les niveaux
- **Clés et valeurs** : Distinction claire entre anonymisation de clés et de valeurs
- **Références entre objets** : Maintien des liens sémantiques

## 4. Règles d'Anonymisation Détaillées

### 4.1 Par Type d'Objet

#### ViewInfo (VI_)
- Préfixe : `VI_`
- Format : `VI_XX` pour les Name principaux
- Champs anonymisés : Toutes les clés contenant "Name" et champs de référence

#### OptionSetInfo (OSI_)
- Préfixe : `OSI_`
- Format : `OSI_XX` pour les Name principaux
- Champs anonymisés : DatasetName, DisplayName, RelatedEntityName, etc.
- Cas spécial : OptionSetInfoNameMapping (valeurs anonymisées)

#### NativeCDSDataSourceInfo (TAB_)
- Préfixe : `TAB_` (sauf exceptions)
- Format : `TAB_XX` pour les Name principaux
- Exceptions : Name="Utilisateurs" ou "Divisions" ne sont pas anonymisés
- Cas spécial : NativeCDSDataSourceInfoNameMapping (clés ET valeurs anonymisées)

### 4.2 Logique de Nommage
- **Name principaux** : `PREFIX_XX` (ex: OSI_01, VI_02)
- **Champs dépendants** : `PARENT_ABBREV_XX` (ex: OSI_01_DN_01, OSI_01_REN_01)
- **Abréviations** :
  - DatasetName → DN
  - DisplayName → DN
  - RelatedEntityName → REN
  - RelatedColumnInvariantName → RCIN
  - OptionSetReferenceEntityName → OSREN
  - OptionSetReferenceColumnName → OSRCN

### 4.3 Cas Particuliers

#### NativeCDSDataSourceInfoNameMapping
- Les **clés** sont remplacées par le Name anonymisé de l'objet référencé
- Les **valeurs** sont anonymisées avec la structure standard
- Exemple : `"smallapp_familleespece"` → `"OSI_68"` (si l'OptionSetInfo correspondant devient OSI_68)

#### OptionSetInfoNameMapping
- Les **valeurs** sont anonymisées uniquement
- Les **clés** restent inchangées (identifiants numériques)
- Structure : `PARENT_OSNM_XX`

#### OptionSetReference
- Les valeurs des champs de référence sont liées aux anonymisations correspondantes
- Cohérence maintenue entre références et objets cibles

## 5. Évolution du Script

### 5.1 Problèmes identifiés et corrigés

#### Itération 1-2 : Problèmes de base
- Erreurs de portée de variables
- Anonymisation incorrecte des clés
- Structure de nommage inadaptée

#### Itération 3-4 : Problèmes de cohérence
- Anonymisation de clés qui ne devaient pas l'être ("Type")
- Incohérences dans les logs
- Mauvaise gestion des structures imbriquées

#### Itération 5-6 : Problèmes avancés
- Non-anonymisation des cas d'exception
- Mauvaise gestion des dictionnaires spéciaux
- Absence de lien entre clés de NativeCDSDataSourceInfoNameMapping et objets référencés

### 5.2 Améliorations itératives

#### Amélioration de la détection
- Auto-détection de la structure (tableau vs objet unique)
- Logs détaillés de la structure détectée

#### Raffinement des règles
- Implémentation correcte des exceptions
- Logique de nommage cohérente
- Gestion des références croisées

#### Optimisation de l'algorithme
- Approche en deux passes pour garantir la cohérence
- Mapping pré-établi des Name principaux
- Gestion des cas particuliers (dictionnaires spéciaux)

## 6. Utilisation du Script

### 6.1 Prérequis
- Python 3.x
- Fichier JSON d'entrée nommé `input.json`
- Permissions d'écriture pour générer `anonymized_output.json`

### 6.2 Exécution
```bash
python anonymize_json.py
```

### 6.3 Résultats
- **Fichier de sortie** : `anonymized_output.json`
- **Logs détaillés** : Affichage console des transformations
- **Traçabilité** : Mapping complet des remplacements

## 7. Exemples

### 7.1 Avant/After

#### Avant anonymisation :
```json
{
  "Type": "OptionSetInfo",
  "Name": "smallapp_familleespece",
  "DatasetName": "default.cds",
  "DisplayName": "Familles Espèce",
  "RelatedEntityName": "Cuves",
  "RelatedColumnInvariantName": "smallapp_familleespece",
  "OptionSetInfoNameMapping": {
    "579940000": "Crocodile",
    "579940001": "Lézard"
  }
}
```

#### Après anonymisation :
```json
{
  "Type": "OptionSetInfo",
  "Name": "OSI_01",
  "DatasetName": "OSI_01_DN_01",
  "DisplayName": "OSI_01_DN_02",
  "RelatedEntityName": "OSI_01_REN_01",
  "RelatedColumnInvariantName": "OSI_01_RCIN_01",
  "OptionSetInfoNameMapping": {
    "579940000": "OSI_01_OSNM_01",
    "579940001": "OSI_01_OSNM_02"
  }
}
```

### 7.2 Cas complexes

#### Liaison NativeCDSDataSourceInfo ↔ OptionSetInfo :
```json
// OptionSetInfo
{
  "Type": "OptionSetInfo",
  "Name": "smallapp_familleespece",
  // ... devient OSI_68
}

// NativeCDSDataSourceInfo
{
  "NativeCDSDataSourceInfoNameMapping": {
    "smallapp_familleespece": "valeur_quelconque"
    // ... devient {"OSI_68": "TAB_01_NCDSDSINM_01"}
  }
}
```

## 8. Limitations et Améliorations Futures

### Contraintes actuelles
- Format d'entrée fixé (`input.json`)
- Pas de paramétrage des règles via configuration externe
- Gestion limitée des erreurs de format JSON

### Évolutions possibles
- **Configuration externe** : Fichier de règles YAML/JSON
- **Interface CLI** : Paramètres en ligne de commande
- **Validation** : Vérification de la cohérence post-anonymisation
- **Performance** : Optimisation pour les très gros fichiers
- **Formats** : Support d'autres formats de sortie (XML, CSV)

---

**Date** : 27/11/2025
**Auteur** : DomTa