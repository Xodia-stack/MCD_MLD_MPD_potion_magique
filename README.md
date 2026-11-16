# Potion magique — Base de données

Modélisation et création d'une base de données **PostgreSQL** pour gérer des potions magiques et les ingrédients qui les composent.

Le projet suit la démarche **Merise** : MCD → MLD → MPD (script SQL).

---

## Sommaire

- [Contexte](#-contexte)
- [Règles de gestion](#-règles-de-gestion)
- [MCD — Modèle Conceptuel de Données](#-mcd--modèle-conceptuel-de-données)
- [MLD — Modèle Logique de Données](#-mld--modèle-logique-de-données)
- [MPD — Modèle Physique de Données](#-mpd--modèle-physique-de-données)
- [Installation](#-installation)
- [Exemples de requêtes](#-exemples-de-requêtes)
- [Technologies](#-technologies)

---

## Contexte

Un apothicaire souhaite référencer ses potions et savoir, pour chacune, quels ingrédients entrent dans sa recette et en quelle quantité.

---

## Règles de gestion

- Une **potion** possède un nom unique, une description, un effet et un temps de préparation.
- Un **ingrédient** possède un nom unique, une origine et une rareté.
- Une potion est composée d'**au moins un** ingrédient (cardinalité `1,n`).
- Un ingrédient peut entrer dans la composition de **zéro ou plusieurs** potions (cardinalité `0,n`).
- Pour chaque ingrédient d'une potion, on enregistre la **quantité** utilisée.

---

## MCD — Modèle Conceptuel de Données

![MCD]<img width="868" height="401" alt="MCD" src="https://github.com/user-attachments/assets/9c4f6f29-f5b1-44c7-9e7c-96fc5f0d8206" />


| Entité / Association | Attributs |
|---|---|
| **potion** | <u>id</u>, name, description, effect, preparation_time |
| **ingredient** | <u>id</u>, name, origin, rarity |
| **Compose** *(association)* | quantity |

```
potion (1,n) ──── Compose [quantity] ──── (0,n) ingredient
```

---

## MLD — Modèle Logique de Données

![MLD]<img width="1083" height="494" alt="MLD" src="https://github.com/user-attachments/assets/44e2e61c-3723-48b3-8433-6a6aa20a93db" />


L'association `n,n` **Compose** devient une table de jointure dont la clé primaire est composée des deux clés étrangères.

```
potion     (id_potion, name, description, effect, preparation_time)
ingredient (id_ingredient, name, origin, rarity)
compose    (#id_potion, #id_ingredient, quantity)
```

> Clés primaires soulignées dans le schéma, `#` = clé étrangère.

---

## MPD — Modèle Physique de Données

![MPD]

Script de création (PostgreSQL) — fichier [`script.sql`](script.sql) :

```sql
CREATE TABLE potion (
    id_potion        SERIAL,
    name_potion      VARCHAR(50) NOT NULL,
    description      TEXT,
    effect           VARCHAR(255),
    preparation_time INTEGER,
    PRIMARY KEY (id_potion),
    UNIQUE (name_potion)
);

CREATE TABLE ingredient (
    id_ingredient    SERIAL,
    name_ingredient  VARCHAR(50) NOT NULL,
    origin           VARCHAR(50),
    rarity           VARCHAR(50),
    PRIMARY KEY (id_ingredient),
    UNIQUE (name_ingredient)
);

CREATE TABLE compose (
    id_potion        INTEGER,
    id_ingredient    INTEGER,
    quantity         NUMERIC(8,2) NOT NULL,
    PRIMARY KEY (id_potion, id_ingredient),
    FOREIGN KEY (id_potion)     REFERENCES potion(id_potion),
    FOREIGN KEY (id_ingredient) REFERENCES ingredient(id_ingredient)
);
```

### Choix techniques

| Colonne | Type | Pourquoi |
|---|---|---|
| `id_*` | `SERIAL` | Identifiant auto-incrémenté |
| `name_*` | `VARCHAR(50) NOT NULL` + `UNIQUE` | Pas deux potions / ingrédients avec le même nom |
| `description` | `TEXT` | Texte libre sans limite de longueur |
| `preparation_time` | `INTEGER` | Durée en minutes |
| `quantity` | `NUMERIC(8,2)` | Quantité décimale précise (ex. `2.50`) |

---

## Installation

Prérequis : **PostgreSQL** installé.

```bash
# 1. Cloner le dépôt
git clone https://github.com/Xodia-stack/MCD_MLD_MPD_potion_magique.git
cd MCD_MLD_MPD_potion_magique

# 2. Créer la base
createdb potion_magique

# 3. Exécuter le script
psql -d potion_magique -f script.sql
```

---

## 🔍 Exemples de requêtes

**Insérer des données**

```sql
INSERT INTO potion (name_potion, description, effect, preparation_time)
VALUES ('Potion de soin', 'Un liquide rouge et sucré', 'Restaure la santé', 30);

INSERT INTO ingredient (name_ingredient, origin, rarity)
VALUES ('Racine de mandragore', 'Forêt sombre', 'Rare'),
       ('Eau de source', 'Montagne', 'Commun');

INSERT INTO compose (id_potion, id_ingredient, quantity)
VALUES (1, 1, 2.00),
       (1, 2, 0.50);
```

**Afficher la recette d'une potion**

```sql
SELECT p.name_potion, i.name_ingredient, c.quantity
FROM potion p
JOIN compose c    ON c.id_potion = p.id_potion
JOIN ingredient i ON i.id_ingredient = c.id_ingredient
WHERE p.name_potion = 'Potion de soin';
```

**Ingrédients jamais utilisés**

```sql
SELECT i.name_ingredient
FROM ingredient i
LEFT JOIN compose c ON c.id_ingredient = i.id_ingredient
WHERE c.id_potion IS NULL;
```

---

## Technologies

- **PostgreSQL**
- **Merise** (MCD / MLD / MPD)
- **Excalidraw** pour les schémas

---

## Auteur

**Xodia** — [GitHub](https://github.com/Xodia-stack)
