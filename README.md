# EcoTrack Assistant

**Équipe** : CHAMSSOUDINE Said · MOHAMED Ibrahim — M2, Ingetis
**Projet** : Embeddings, recherche sémantique et RAG avec Gemini
**Rendu** : vendredi 2 octobre 2026

Un assistant qui répond aux questions des citoyens et des gestionnaires **à partir de la documentation EcoTrack uniquement**. Il **cite ses sources** et **refuse poliment** quand l'information n'est pas dans le corpus.

---

## Contenu du dépôt

```
├── EcoTrack_Assistant.ipynb        Notebook complet, exécutable de bout en bout
├── resultats_test.csv              Résultats du jeu de test (10 questions)
├── fichiers_colab/                 Corpus et données réellement utilisés (noms d'origine)
│   ├── corpus_ecotrack/            Les 7 documents EcoTrack
│   ├── jeu_test_10_questions.csv   Le jeu de test fourni
│   ├── etat_conteneurs.csv         Les données du bonus Tool Use
│   └── bonus_document_injection.txt  Le document piégé du bonus injection
├── fichiers_colab.zip              Les mêmes fichiers, en un seul fichier à téléverser dans Colab
├── cache_gemini.json               Réponses Gemini déjà obtenues (pour relancer sans consommer de quota)
├── presentation/                   Support de présentation
└── docs/
    └── journal_des_difficultes.md  Les erreurs rencontrées, leurs causes et nos corrections
```

## Livrables demandés

| Livrable du sujet | Où le trouver |
|---|---|
| Notebook exécutable de bout en bout | `EcoTrack_Assistant.ipynb` |
| Corpus réellement utilisé, noms conservés | `fichiers_colab/corpus_ecotrack/` |
| Résultats du jeu de test | `resultats_test.csv`, et le tableau de l'étape 7 du notebook |
| Analyse d'erreur | Étape 8 du notebook |
| Support de présentation | `presentation/` |
| Lien vers le dépôt | Ce dépôt |

Aucune clé API n'apparaît dans le dépôt : elle est lue dans les secrets Colab.

---

## Exécuter le notebook

1. Ouvrir `EcoTrack_Assistant.ipynb` dans **Google Colab** (*Fichier > Importer le notebook*).
2. Panneau 🔑 *Secrets* : ajouter un secret nommé `GOOGLE_API_KEY` (une clé Gemini), et activer **Accès au notebook**.
3. Panneau 📁 *Fichiers* : téléverser **`fichiers_colab.zip`**, et **`cache_gemini.json`** pour retrouver les mêmes réponses sans consommer de quota.
4. *Exécution > Tout exécuter*.

Versions de référence (préinstallées dans Colab, aucune mise à jour) : numpy 2.1.3 · pandas 2.2.3 · google-genai 2.12.1 · sentence-transformers 5.7.0.

---

## Fonctionnement

```
Question → embedding (MiniLM, 384 nombres) → 3 passages les plus proches (cosinus) → seuil 0,31 → Gemini rédige et cite
                                                                                       └→ refus poli
```

| Choix | Valeur | Justification |
|---|---|---|
| Découpage | Fenêtre glissante de 600 caractères, chevauchement de 100 → 14 passages | Meilleure précision@3 que le découpage par paragraphe (9/10 contre 8/10) |
| Embeddings | `paraphrase-multilingual-MiniLM-L12-v2` | Multilingue, gratuit, exécuté dans Colab, sans quota |
| Seuil de refus | 0,31 = (0,429 + 0,193) / 2 | Milieu entre la plus faible bonne question et la plus forte question hors sujet |
| Génération | Gemini (`gemini-3.8-flash`, bascule automatique si quota épuisé), température 0,1 | Le modèle du sujet (`gemini-2.5-flash`) n'est plus disponible (erreur 404) |
| Refus | Deux barrières : le seuil (avant Gemini) et la consigne (dans Gemini) | Le seuil seul laisse passer des questions dans le thème mais sans réponse |

---

## Résultats

| Mesure | Résultat |
|---|---|
| **Précision@3** (document attendu dans le top-3) | **9 / 10** · question ratée : Q05 |
| Réponses correctes | 7 / 10 · Q03 et Q09 : bon document, mais pas le bon passage |
| Analyse d'erreur (Q05) | **Dilution** : la phrase-réponse seule obtient 0,545, mais noyée dans son passage, 0,359 (rang 5) |
| Bonus A · Tool Use | ✓ Gemini appelle `etat_conteneurs(zone)` pour l'état d'une zone, et pas pour une question de tri |
| Bonus B · Reflection | ✓ Mesurée : 7/10 → 6/10. Elle corrige la rédaction, pas la recherche (réserve : changement de modèle en cours de route) |
| Bonus C · Injection documentaire | ✓ Document piégé reçu en 1re position, l'assistant n'obéit pas à l'instruction malveillante |

Les difficultés rencontrées (quotas de l'API Gemini, environnement Colab, données) et leurs corrections sont détaillées dans [`docs/journal_des_difficultes.md`](docs/journal_des_difficultes.md).
