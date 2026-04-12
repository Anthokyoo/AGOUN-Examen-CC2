# TP EXAMEN CC2

Ce dépôt regroupe les travaux du **CC2** portant sur l'analyse de données massives avec **Hadoop MapReduce**.

### Auteurs
- **SON Anthony**
- **TABOADA Martin**

### Objectifs du TP
- Mise en place d'un environnement **Hadoop pseudo-distribué** (Sandbox HDP).
- Développement de scripts **Python (mrjob)** pour analyser 25 millions de tags.
- Gestion de la robustesse des traitements (blocs `try/except`).
- Manipulation de la configuration **HDFS** (taille des blocs à 64 Mo).

### Fichiers clés
- **Scripts :** `tags_per_movie.py`, `tags_per_user.py`, `count_by_tag.py`, `tags_per_movie_and_user.py`.
- **Rapport :** Le détail des commandes et analyses est disponible dans le fichier **[SON_TABOADA_CC2.md](./SON_TABOADA.md)**.
- **Résultats :** Fichiers `.txt` contenant les sorties des jobs Hadoop.
