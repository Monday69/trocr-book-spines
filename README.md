# TrOCR Book Spines

Génération de données synthétiques et fine-tuning de [TrOCR](https://huggingface.co/microsoft/trocr-base-printed) pour lire les titres et auteurs sur des **tranches de livres** photographiées en rayon.

Ce dépôt est la brique OCR du projet [BIBLIA-TECH](https://github.com/Monday69/BIBLIA-TECH), un scanner de bibliothèque qui détecte les livres mal rangés (ESME, projet de semestre). Il est référencé dans l'annexe E du rapport du projet.

## Pipeline

1. **`synthetic_factory.py`** génère jusqu'à 76 500 images de tranches de livres à partir de `DATASET_OCR_MASTER.csv`.
2. **`train_ocr.py`** fine-tune `microsoft/trocr-base-printed` sur ces images et évalue le CER (Character Error Rate).

## Génération de données (`synthetic_factory.py`)

Chaque image simule une prise de vue difficile en bibliothèque :

- **Typographie variée** : police tirée au hasard, mise en page horizontale, verticale ou inversée.
- **Dégradations réalistes** : bruit numérique, reflets de plastique type flash, ombres, rayures d'usure, flou, variations de contraste et de luminosité, compression JPEG variable.
- **Sortie** : images `synth_XXXXX.jpg` et un fichier `metadata.jsonl` (`file_name`, `text`) compatible avec Hugging Face `datasets`.

La source `DATASET_OCR_MASTER.csv` contient environ 76 600 couples titre / auteur.

## Entraînement (`train_ocr.py`)

| Paramètre | Valeur |
| --- | --- |
| Modèle de base | `microsoft/trocr-base-printed` |
| Stratégie | Encodeur vision gelé, seul le décodeur est entraîné |
| Batch | 4 × 4 étapes d'accumulation = batch effectif de 16 |
| Learning rate | 1e-6 |
| Époques | 30 max, avec early stopping sur le CER |
| Précision | FP16 si CUDA est disponible |
| Matériel visé | RTX 4070 (12 Go de VRAM) |

Le jeu de données est découpé en 80 % entraînement et 20 % validation (`seed=42`).

## Utilisation

### 1. Installation

```bash
pip install -r requirements.txt
```

### 2. Polices

Le dossier `fonts/` n'est pas versionné. Crée-le à la racine et ajoutes-y des polices TrueType (`.ttf`), par exemple depuis [Google Fonts](https://fonts.google.com). Plus les styles sont variés, plus le modèle sera robuste.

### 3. Génération

```bash
python synthetic_factory.py
```

Les images et `metadata.jsonl` sont écrits dans `synth_crops_v3_hardcore/`.

### 4. Entraînement

Dans `train_ocr.py`, fais pointer `DATA_DIR` vers le dossier des images. Le script attend un fichier nommé `metadata_balanced.jsonl` : renomme `metadata.jsonl` (ou ta version rééquilibrée) en conséquence.

```bash
python train_ocr.py
```

Le meilleur modèle est sauvegardé dans `trocr_books_model_final/`.

## Structure

```
.
├── DATASET_OCR_MASTER.csv   # Titres et auteurs source
├── synthetic_factory.py     # Génération des images synthétiques
├── train_ocr.py             # Fine-tuning TrOCR
└── requirements.txt
```

## Licence

MIT
