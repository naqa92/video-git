# [TUTO] Git/Github : Comprendre les 3 stratégies principales

## Pré-requis

- CLI Git
- Clé SSH

git lg alias :

```bash
git config --global alias.lg "log --oneline --graph --all --decorate --color"
```

## Trunk based

- C'est quoi ? : 1 seule branche principale (main). C'est le workflow le plus simple
- Quand l'utiliser ? : On travaille seul (ex: mini-projet perso, pas besoin de reviews) ou pipeline CI/CD avec automatisations avancées

Création d'un repo local + distant

On fait une modification

```bash
git status
```

puis on Stage/commit/push

```bash {"name":"trunk-based"}
git add . # staging (permet à git de tracker les fichiers, c'est une préparation au commit)
```

```bash
git commit -m "update1" # commit (sauvegarde des changements avec message. c'est ça qui permet d'identifier l'ID, la date et l'auteur)
```

```bash
git push # on pousse les changements
```

## Feature branch

- C'est quoi ? : Les nouvelles modifications se font dans une branche dédiée, qui sera ensuite fusionnée dans la branche principale (main) via une Pull Request.
- Quand l'utiliser ? : à chaque fois qu'on développe une nouvelle fonctionnalité ou corrige un bug, afin d’isoler les changements du reste du projet.

✅ Étape 1 : Création de la branche de feature

```bash
git pull # Update branche main locale (Prenez l'habitude de commencer par un pull, ça serait dommage de commencer avec un repo local désynchro)
```

```bash
git switch -c zone-infra-01 # Branche temporaire
```

✍️ Étape 2 : Modification et push dans zone-infra-01

On fait une modification puis on push

```bash
git add .
```

```bash
git commit -m "update2"
```

```bash
git push -u origin zone-infra-01 # push vers origin (repo distant) la branche zone-infra-01 / Cliquer sur l'URL de PR
```

> On retrouve le commit message en titre / On peut mettre une description
> On peut demander review Copilot
> Assignee pour mettre quelqu'un ou s'assigner
> On voit la diff plus bas
> On peut créer un brouillon pour bloquer le merge temporairement

> Les PR sont importants pour la revue de code, c'est une pratique essentielle lorsqu'on utilise les Features branch (1 PR = 1 fonctionnalité)

🔁 Étape 3 : Récupérer les changements sur main et nettoyage

```bash
git switch main
```

```bash
git pull # Update de la branche main en local pour récupérer les changements
```

```bash
git branch
git branch -d zone-infra-01 # nettoyage de la branche local
```

```bash
git lg
```

- Problème : Si une autre personne fait une modif entre temps et on push : erreur rejected (il y a un écart entre le repo local et distant)
- Solution : Utiliser rebase (alternative : merge)

| Action | Commande          | Avantage            | Inconvénient            |
| ------ | ----------------- | ------------------- | ----------------------- |
| Merge  | `git merge main`  | Simple à utiliser   | Historique moins propre |
| Rebase | `git rebase main` | Historique linéaire | Réécriture d’historique |

## Feature branch avec rebase

✅ Étape 1 : Création de la branche de feature

```bash
git pull # Update branche main locale
```

```bash
git switch -c zone-infra-01
```

✍️ Étape 2 : Modification dans zone-infra-01

On fait une modification puis on commit

```bash
git add .
```

```bash
git commit -m "update3"
```

🔁 Étape 3 : Avant de pousser, imaginez qu'en parrallèle, un collègue a modifié main

```bash
git switch main
git pull
git switch -c zone-infra-02
```

```bash
vim fichier.txt # Modif sur main entre-temps
```

```bash
git add .
```

```bash
git commit -m "Modification sur main par un autre développeur"
```

```bash
git push
```

```bash
git lg
```

⚠️ Étape 4 : Rebase

Récupère les dernières modifications de origin (main distant) + Rebaser la branche actuelle (zone-infra-01) sur main

```bash
git switch zone-infra-01
git pull --rebase origin main # Un conflit dans fichier.txt va se produire ici.
```

🛠️ Étape 5 : Résolution du conflit

```bash
git status # rebase in progress
```

```bash
git add .
```

```bash
git rebase --continue # commit message
```

```bash
git push -u origin zone-infra-01
```

```bash
git lg
```

> Donc: avant chaque push, il faut toujours faire `git pull --rebase origin main` (pas de conflit : OK / conflit : On gère ça dans un IDE comme vu précédemment)

⚠️ Bon à savoir :

Si tu as déjà poussé ta branche et que tu fais un rebase, Git refusera de la mettre à jour (car l'historique ne correspond plus). Utilise `git push --force-with-lease` pour forcer la mise à jour mais uniquement si t'es sûr que personne d'autre ne travaille sur sur la même branche (la branche de feature). Cette commande est plus sûre que `--force` car elle vérifie d'abord si des changements ont été ajoutés par d'autres personnes.

> Généralement, on veut éviter d'utiliser force et on veut faire un rebase avant de pousser une branche de feature.

## Fork de projet : https://github.com/firstcontributions/first-contributions

- C'est quoi ? : Chaque contributeur crée un fork (copie) du dépôt principal, travaille sur sa propre branche, puis propose ses changements via une Pull Request.
- Quand l'utiliser ? : Utilisé surtout pour l’open source

```bash
# 1) Fork (UI) -> Fork du repo officiel sur GitHub

# 2) Cloner ton fork
git clone git@github.com:naqa92/first-contributions.git

cd nom-du-depot

# 3) Créer une branche pour ta modification
git switch -c add-naqa92

# 4) Travailler / commit
# modifier fichiers...

git add Contributors.md
git commit -m "Add naqa92 to Contributors list"

# 5) Pousser ta branche sur ton fork
git push -u origin add-naqa92

# 6) Ouvrir la Pull Request (depuis ton fork vers le repo officiel)
```

> Les mainteneurs du projet vont ensuite examiner la PR avant approbation

👉 Et si jamais le dépôt officiel a bougé depuis ton fork :

```bash
# mettre ton main à jour depuis le dépôt officiel
git switch main
git pull https://github.com/propriétaire-original/nom-du-depot.git main

# Revenir sur ta branche et fusionner les nouveautés
git switch ma-nouvelle-fonctionnalité
git rebase main # (ou merge, plus simple mais historique moins propre)

git push -u origin ma-nouvelle-fonctionnalité
```
