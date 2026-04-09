# Compte Rendu - Examen CC2
**Noms :** SON Anthony | TABOADA Martin

## Lien GitHub
*Les fichiers de résultats massifs sont disponibles ici :* https://github.com/Anthokyoo/AGOUN-Examen-CC2

---

## 1. Installation et Déploiement de l'Environnement

L'objectif de cette première étape est de mettre en place un cluster Hadoop local fonctionnant sur un seul nœud (pseudo-distribué) pour exécuter nos jobs MapReduce.

**Prérequis et Démarrage :**
* **Environnement :** Sandbox Hortonworks Data Platform (HDP) déployée via une machine virtuelle (VirtualBox/Oracle). 
* **Ressources :** 4 Go de RAM minimum (8 Go recommandés pour des performances optimales).
* **Initialisation :** Le lancement de la VM construit une image Docker et exécute le conteneur du cluster.

**Accès aux interfaces et connexion :**
* **Terminal Web (Shell-in-a-box) :** `http://localhost:4200`
* **Accès SSH (privilégié pour nos traitements) :** `ssh maria_dev@localhost -p 2222` ou `ssh raj_ops@localhost -p 2222`

---

## 2. Préparation des données et du Dataset

Avant de lancer les traitements MapReduce, il a été nécessaire de préparer l'environnement HDFS avec le dataset MovieLens 25M.

**Commandes de préparation :**

```bash
# Téléchargement du dataset (en ignorant les certificats SSL obsolètes)
wget --no-check-certificate https://files.grouplens.org/datasets/movielens/ml-25m.zip

# Décompression de l'archive (Si les fichiers sont déjà présent, il faudra les écraser "All" afin d'être sûr qu'on ai la dernière version)
unzip ml-25m.zip

# Création du répertoire de travail sur HDFS
hdfs dfs -mkdir -p ml-25m

# Transfert du fichier tags.csv vers HDFS
hdfs dfs -put ml-25m/tags.csv ml-25m/

# On va vérifier que notre fichier est bien arrivé
hdfs dfs -ls ml-25m/

# Création d'un échantillon pour les tests locaux
head -n 100 ml-25m/tags.csv > sample_tags.csv
```

### 3. Configuration par défaut de Hadoop

#### 3.1 Combien de tags chaque film possède-t-il ?

**Commandes utilisées :**

```bash
# Création du script Python
nano tags_per_movie.py

# Exécution du job MapReduce sur le cluster via Hadoop Streaming
python tags_per_movie.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar ml-25m/tags.csv -o output_q3_1

# Affichage d'un aperçu des résultats
hdfs dfs -cat output_q3_1/part-* | head -n 20

# Téléchargement du fichier de résultats pour le dépôt GitHub
hdfs dfs -get output_q3_1/part-00000 ./resultats_films_tags.txt
```

Lien Github du résultat : https://github.com/Anthokyoo/AGOUN-Examen-CC2/blob/main/resultats_films_tags.txt

Le fichier `tags_per_movie.py` ressemble à cela :

```python
# -*- coding: utf-8 -*-
from mrjob.job import MRJob

class TagsPerMovie(MRJob):

    def mapper(self, _, line):
        try:
            # Separation par virgule (format CSV de ml-25m)
            parts = line.split(',')
            
            # On verifie la structure et on ignore l'en-tete
            if len(parts) == 4 and parts[0] != 'userId':
                movieId = parts[1]
                yield movieId, 1
        except Exception:
            pass

    def reducer(self, movieId, counts):
        # Somme des tags pour ce film
        yield movieId, sum(counts)

if __name__ == '__main__':
    TagsPerMovie.run()
```

Le script python utise la bibliotèque 'mrjob'.

Le Mapper va analyser le fichier .csv, séparer les colonnes par des virgules et extrait le 'movieId'. Emet la paire (movieId, 1). Un bloc 'try...except encapsule l'instruction pour ignorer la ligne d'en tête et les potentielles erreurs de formatage.

Le Reducer va agrèger les données par clé ('movieId') et faire la somme des occurences pour obtenir le nombre total de tags associés à chaque film.

Résultat (Aperçu des 20 premières lignes) :

```text
"1"     697
"10"    137
"100"   18
"1000"  10
"100001"        1
"100003"        3
"100008"        9
"100017"        9
"100032"        2
"100034"        19
"100036"        1
"100038"        4
"100042"        2
"100044"        12
"100046"        3
"100048"        1
"100052"        4
"100054"        6
"100060"        10
"100062"        2
```

#### 3.2 Combien de tags chaque utilisateur a-t-il ajoutés ?

**Commandes utilisées :**

```bash
# Création du script Python
nano tags_per_user.py

# Exécution du job MapReduce sur le cluster via Hadoop Streaming
python tags_per_user.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar ml-25m/tags.csv -o output_q3_2

# Affichage d'un aperçu des résultats
hdfs dfs -cat output_q3_2/part-* | head -n 20

# Téléchargement du fichier de résultats pour le dépôt GitHub
hdfs dfs -get output_q3_2/part-00000 ./resultats_utilisateurs_tags.txt
```

Lien Github du résultat : https://github.com/Anthokyoo/AGOUN-Examen-CC2/blob/main/resultats_utilisateurs_tags.txt

Le fichier `tags_per_user.py` ressemble à cela :

```python
# -*- coding: utf-8 -*-
from mrjob.job import MRJob

class TagsPerUser(MRJob):

    def mapper(self, _, line):
        try:
            # Separation par virgule (format CSV de ml-25m)
            parts = line.split(',')
            
            # On verifie la structure et on ignore l'en-tete
            if len(parts) == 4 and parts[0] != 'userId':
                userId = parts[0] # <-- C'EST ICI QUE ÇA CHANGE
                yield userId, 1
        except Exception:
            pass

    def reducer(self, userId, counts):
        # Somme des tags pour cet utilisateur
        yield userId, sum(counts)

if __name__ == '__main__':
    TagsPerUser.run()
```

On utilise la même logique que la question précédente mais le script diffère uniquement au niveau du Mapper : au lieu d'extraire le 'movieId', nous allons extraire le 'userId' et émettre la paitre ('userId', 1). Le Reducer fait la somme des tags pour chaque utilisateur.

Résultats (Aperçu des 20 premières lignes) :

```text
"100001"        9
"100016"        50
"100028"        4
"100029"        1
"100033"        1
"100046"        133
"100051"        19
"100058"        5
"100065"        2
"100068"        19
"100076"        4
"100085"        3
"100087"        8
"100088"        13
"100091"        29
"100101"        3
"100125"        3
"100130"        2
"100140"        5
"100141"        26
```

## 4. Configuration de Hadoop et Nouveaux Traitements

### 4.1 Taille des blocs (Défaut vs 64 Mo)

**Commandes utilisées :**
```bash
# Vérification avec la taille de bloc par défaut (128 Mo)
hdfs fsck /user/maria_dev/ml-25m/tags.csv -files -blocks

# Transfert avec taille de bloc forcée à 64 Mo
hdfs dfs -D dfs.blocksize=67108864 -put ml-25m/tags.csv ml-25m/tags_64M.csv

# Vérification du nouveau fichier
hdfs fsck /user/maria_dev/ml-25m/tags_64M.csv -files -blocks
```

Résultats :

Dans les deux cas, la commande fsck nous indique que le fichier occupe 1 seul bloc.
Cela s'explique par la taille totale du fichier tags.csv (environ 38 Mo). Puisque 38 Mo est inférieur à la fois à 128 Mo (défaut) et à 64 Mo (forcé), Hadoop n'a pas besoin de fragmenter le fichier sur plusieurs blocs.

### 4.2 Combien de fois chaque tag a-t-il été utilisé pour taguer un film ?

On veut isoler la colonne du 'tag'.

**Commandes utilisées :**

```bash
# Création du script Python
nano count_by_tag.py

# Lancement du job sur le fichier configuré avec des blocs de 64 Mo
python count_by_tag.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar hdfs:///user/maria_dev/ml-25m/tags_64M.csv -o output_q4_2

# Affichage d'un aperçu des résultats
hdfs dfs -cat output_q4_2/part-* | head -n 20

# Téléchargement du fichier de résultats pour le dépôt GitHub
hdfs dfs -get output_q4_2/part-00000 ./resultats_tags_utilises.txt
```

Lien Github du résultat : https://github.com/Anthokyoo/AGOUN-Examen-CC2/blob/main/resultats_tags_utilises.txt

Le fichier `count_by_tag.py` ressemble à cela :

```python
# -*- coding: utf-8 -*-
from mrjob.job import MRJob

class CountByTag(MRJob):

    def mapper(self, _, line):
        try:
            parts = line.split(',')
            # On verifie le format et on s'assure de ne pas prendre l'en-tete
            if len(parts) >= 4 and parts[0] != 'userId':
                # Le tag est la 3eme colonne (index 2)
                # On le met en minuscules pour eviter les doublons (ex: "Funny" et "funny")
                tag = parts[2].lower()
                yield tag, 1
        except Exception:
            pass

    def reducer(self, tag, counts):
        yield tag, sum(counts)

if __name__ == '__main__':
    CountByTag.run()
```

Le Mapper extrait la colonne 3 (parts[2]) qui correspond au tag. Pour éviter les doublons liés à la casse, on va utiliser la méthode '.lower()' sur la chaine de caractères avant d'émettre la paire ('tag', 1).

Le Reducer reçoit les tags identiques groupés et somme leurs occurences pour obtenir la fréquence d'utilisation totale.

Résultats (Aperçu des 20 premières lignes) :

```text
" alexander skarsg\u00e5rd"     1
" ballet school"        1
" breakup"      1
" difficult to find it" 1
" filmes antigos "      2
" filmes antigos"       2
" kartik aaryan"        1
" kriti sanon"  1
" laurel canyon"        1
" luis brandoni"        1
" masami nagasawa"      1
" mental illness"       2
" o'shea jackson jr."   1
" sherlock holmes "     1
" spin off"     1
" the lost boys series" 3
"!950's superman tv show"       1
"#1 prediction" 3
"#adventure"    1
"#antichrist"   1
```

### 4.3 Pour chaque film, combien de tags le même utilisateur a-t-il introduits ?

**Commandes utilisées :**

```bash
# Création du script Python
nano tags_per_movie_and_user.py

# Lancement du job sur le fichier configuré avec des blocs de 64 Mo
python tags_per_movie_and_user.py -r hadoop --hadoop-streaming-jar /usr/hdp/current/hadoop-mapreduce-client/hadoop-streaming.jar hdfs:///user/maria_dev/ml-25m/tags_64M.csv -o output_q4_3

# Affichage d'un aperçu des résultats
hdfs dfs -cat output_q4_3/part-* | head -n 20

# Téléchargement du fichier de résultats pour le dépôt GitHub
hdfs dfs -get output_q4_3/part-00000 ./resultats_films_utilisateurs_tags.txt
```

Lien Github du résultat : https://github.com/Anthokyoo/AGOUN-Examen-CC2/blob/main/resultats_films_utilisateurs_tags.txt

Le fichier `tags_per_movie_and_user.py` ressemble à cela :

```python
# -*- coding: utf-8 -*-
from mrjob.job import MRJob

class TagsPerMovieAndUser(MRJob):

    def mapper(self, _, line):
        try:
            parts = line.split(',')
            if len(parts) >= 4 and parts[0] != 'userId':
                userId = parts[0]
                movieId = parts[1]
                
                # La clé est une chaîne de caractères combinant le film et l'utilisateur
                complex_key = "Film: " + movieId + " | User: " + userId
                
                yield complex_key, 1
        except Exception:
            pass

    def reducer(self, complex_key, counts):
        yield complex_key, sum(counts)

if __name__ == '__main__':
    TagsPerMovieAndUser.run()
```

Nous avons du croiser deux informations.

Le Mapper extrait l'identifiant de l'utilisateur ('userId' - index 0) et l'identifiant du fil ('movieId' - index 1). On fusionne ensuite ces données pour créer une clé unique (par exemple : "Film: 1 | User: 42"), et émet la paire.

Le Reducer agglomère ensuite les données basées sur cette clé et fait la somme, ce qui nous donne le nombre exact de tags qu'un utilisateur spécifique a introduits pour un film spécifique.

Résultats (Aperçu des 20 premières lignes) :

```text
"Film: 1 | User: 100538"        4
"Film: 1 | User: 10231" 2
"Film: 1 | User: 102568"        4
"Film: 1 | User: 102901"        1
"Film: 1 | User: 103368"        1
"Film: 1 | User: 103371"        1
"Film: 1 | User: 103883"        3
"Film: 1 | User: 104394"        9
"Film: 1 | User: 1048"  1
"Film: 1 | User: 105717"        1
"Film: 1 | User: 105809"        5
"Film: 1 | User: 107432"        2
"Film: 1 | User: 109146"        2
"Film: 1 | User: 109258"        1
"Film: 1 | User: 110339"        3
"Film: 1 | User: 110966"        1
"Film: 1 | User: 111033"        3
"Film: 1 | User: 111139"        1
"Film: 1 | User: 111183"        1
"Film: 1 | User: 112824"        3
```