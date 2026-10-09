# Journal des essais IA

Ce journal trace chaque utilisation de l'IA dans le projet : outil, prompt, résultat, observations et décision.

## Essai 0 – 9 octobre 2026

Outil et modèle : Gemini (abonnement Pro), nouvelle conversation

Objectif : générer des schémas Pandera à partir de la structure des trois tables (prompt A)

Prompt : identique au prompt de l'essai 1

Sortie : refus, l'outil répond qu'il n'est pas en mesure d'aider sur cette demande

Observations : refus injustifié sur une demande technique légitime. Les outils d'IA générative ne sont pas toujours fiables ni reproductibles. Par ailleurs, l'outil était rattaché à un compte professionnel, ce qui n'est pas adapté à un projet de formation.

Décision : essai abandonné, passage à ChatGPT avec un compte personnel.

## Essai 1 – 9 octobre 2026

Outil et modèle : ChatGPT, version gratuite (modèle par défaut), nouvelle conversation

Objectif : mesurer ce qu'une IA produit avec un prompt minimal, sans contexte métier

Prompt :

> Je travaille sur trois exports Excel d'un caviste en ligne (ERP, site web, table de liaison), chargés avec pandas. Voici leur structure : [structure des tables, colonnes et types uniquement]. Écris des schémas Pandera pour valider la qualité de ces données.

<details>
<summary>Sortie complète</summary>

```text
Pour valider tes trois DataFrames avec Pandera, je te conseille de définir un schéma par table, en distinguant :

Les contraintes de structure : colonnes attendues, types, colonnes obligatoires.

Les contraintes de qualité : identifiants uniques, valeurs positives, booléens codés en 0 et 1, valeurs manquantes.

La cohérence entre tables : correspondance entre les identifiants ERP, les identifiants de liaison et les SKU du site web.

Attention : les nombres de lignes fournis sont utiles pour détecter des anomalies, mais je ne les rendrais pas obligatoires dans les schémas, car ils peuvent évoluer au fil des mises à jour.

1. Schémas Pandera

Voici une implémentation compatible avec l'API moderne de Pandera. Elle suppose que tes DataFrames s'appellent erp, web et liaison.

Installation si nécessaire :

pip install pandera

import pandas as pd
import pandera.pandas as pa


# -------------------------
# 1. Table ERP
# -------------------------

schema_erp = pa.DataFrameSchema(
    columns={
        "product_id": pa.Column(
            pa.Int64,
            nullable=False,
            unique=True,
            checks=pa.Check.ge(1),
        ),
        "onsale_web": pa.Column(
            pa.Int64,
            nullable=False,
            checks=pa.Check.isin([0, 1]),
        ),
        "price": pa.Column(
            pa.Float64,
            nullable=False,
            checks=pa.Check.ge(0),
        ),
        "stock_quantity": pa.Column(
            pa.Int64,
            nullable=False,
            checks=pa.Check.ge(0),
        ),
        "stock_status": pa.Column(
            pa.String,
            nullable=False,
            checks=pa.Check.isin(["instock", "outofstock"]),
        ),
        "purchase_price": pa.Column(
            pa.Float64,
            nullable=False,
            checks=pa.Check.ge(0),
        ),
    },
    strict=True,
    coerce=False,
)


# -------------------------
# 2. Table Web
# -------------------------

schema_web = pa.DataFrameSchema(
    columns={
        "sku": pa.Column(pa.String, nullable=True),
        "virtual": pa.Column(
            pa.Int64, nullable=False,
            checks=pa.Check.isin([0, 1]),
        ),
        "downloadable": pa.Column(
            pa.Int64, nullable=False,
            checks=pa.Check.isin([0, 1]),
        ),
        "rating_count": pa.Column(
            pa.Int64, nullable=False,
            checks=pa.Check.ge(0),
        ),
        "average_rating": pa.Column(
            pa.Float64, nullable=False,
            checks=pa.Check.in_range(0, 5),
        ),
        "total_sales": pa.Column(
            pa.Float64, nullable=False,
            checks=pa.Check.ge(0),
        ),
        "tax_status": pa.Column(pa.String, nullable=False),
        "tax_class": pa.Column(pa.Float64, nullable=True),
        "post_author": pa.Column(pa.Float64, nullable=True),
        "post_date": pa.Column(pa.DateTime, nullable=True),
        "post_date_gmt": pa.Column(pa.DateTime, nullable=True),
        "post_content": pa.Column(pa.Float64, nullable=True),
        "product_type": pa.Column(pa.String, nullable=True),
        "post_title": pa.Column(pa.String, nullable=True),
        "post_excerpt": pa.Column(pa.String, nullable=True),
        "post_status": pa.Column(pa.String, nullable=False),
        "comment_status": pa.Column(pa.String, nullable=False),
        "ping_status": pa.Column(pa.String, nullable=False),
        "post_password": pa.Column(pa.Float64, nullable=True),
        "post_name": pa.Column(pa.String, nullable=True),
        "post_modified": pa.Column(pa.DateTime, nullable=True),
        "post_modified_gmt": pa.Column(pa.DateTime, nullable=True),
        "post_content_filtered": pa.Column(
            pa.Float64, nullable=True
        ),
        "post_parent": pa.Column(pa.Float64, nullable=True),
        "guid": pa.Column(pa.String, nullable=True),
        "menu_order": pa.Column(pa.Float64, nullable=True),
        "post_type": pa.Column(pa.String, nullable=False),
        "post_mime_type": pa.Column(pa.String, nullable=True),
        "comment_count": pa.Column(
            pa.Float64,
            nullable=True,
            checks=pa.Check.ge(0),
        ),
    },
    strict=True,
    coerce=False,
)


# -------------------------
# 3. Table de liaison
# -------------------------

schema_liaison = pa.DataFrameSchema(
    columns={
        "id_web": pa.Column(
            pa.String,
            nullable=False,
        ),
        "product_id": pa.Column(
            pa.Int64,
            nullable=False,
            unique=True,
            checks=pa.Check.ge(1),
        ),
    },
    strict=True,
    coerce=False,
)
Quelques choix importants

strict=True détecte les colonnes inattendues ou manquantes.

coerce=False évite de convertir silencieusement les données pour faire passer la validation.

nullable=True autorise les valeurs manquantes dans les champs où elles peuvent être légitimes.

unique=True garantit l'unicité des identifiants concernés.

pa.Check exprime les contraintes métier, comme un prix positif ou une note comprise entre 0 et 5.

À vérifier sur tes données : j'ai supposé que stock_status ne pouvait prendre que les valeurs instock et outofstock, et que le stock ne pouvait pas être négatif. Vérifie les modalités réelles avant de conserver ces contraintes. De même, plusieurs colonnes Web sont de type float64 parce qu'elles peuvent contenir des valeurs manquantes ; il faut confirmer leur type métier attendu avant d'ajouter des contrôles plus stricts.

2. Exécuter les validations

Pour obtenir les erreurs de plusieurs colonnes en une seule exécution, utilise lazy=True.


schemas = {
    "ERP": (schema_erp, erp),
    "Web": (schema_web, web),
    "Liaison": (schema_liaison, liaison),
}

for nom, (schema, df) in schemas.items():
    try:
        schema.validate(df, lazy=True)
        print(f"{nom} : validation réussie")
    except pa.errors.SchemaErrors as e:
        print(f"{nom} : erreurs de validation")
        print(e.failure_cases)
3. Contrôler la cohérence entre les tables

Les schémas vérifient chaque table indépendamment. Il reste à vérifier que les clés de liaison existent bien dans les tables concernées.


# Un produit ERP ne doit apparaître qu'une fois dans la liaison.
assert liaison["product_id"].is_unique

# Chaque product_id de la liaison doit exister dans l'ERP.
ids_erp = set(erp["product_id"])
ids_liaison = set(liaison["product_id"])

assert ids_liaison.issubset(ids_erp), (
    "Des product_id de la liaison sont absents de l'ERP"
)

# Chaque id_web doit correspondre à un SKU présent sur le site.
skus_web = set(web["sku"].dropna())
ids_web = set(liaison["id_web"].dropna())

assert ids_web.issubset(skus_web), (
    "Des id_web de la liaison sont absents des SKU du site"
)

print("Contrôles de cohérence terminés.")

Ce dernier contrôle suppose que id_web et sku utilisent le même format. Si les SKU sont dupliqués dans le site web, il faudra vérifier que ces doublons sont légitimes avant de conclure.

4. Contrôles complémentaires recommandés

Tu peux aussi ajouter des contrôles pour :

détecter les SKU manquants ou dupliqués ;

vérifier que les 825 produits ERP sont bien représentés dans la table de liaison ;

comparer onsale_web avec la présence d'un produit publié sur le site ;

repérer les écarts entre le stock ERP et le statut de stock du site.

Ces derniers contrôles sont des règles métier inter-tables : ils sont généralement plus faciles à gérer séparément des schémas Pandera.
```

</details>

Observations : syntaxe Pandera moderne, validation lazy et hypothèses signalées honnêtement. En revanche, les contrôles restent génériques (types, bornes) : pas de vente à perte (R4), pas de cohérence stock/statut (R3), pas de détection HTML ni de distinction bloquant/avertissement. Risque de faux positifs, par exemple un id_web exigé pour les produits vendus uniquement en magasin. Les contrôles inter-tables utilisent des assert, qui s'arrêtent à la première erreur.

Décision : non retenu comme base. Sert de point de comparaison pour mesurer l'apport du contexte métier dans le prompt.

## Essai 2 – 9 octobre 2026

Outil et modèle : ChatGPT, version gratuite (modèle par défaut), nouvelle conversation

Objectif : mesurer l'apport d'un prompt structuré (rôle, contexte, catalogue de règles, contraintes)

Prompt :

> Tu es un data analyst expert en qualité des données. Contexte : BottleNeck, caviste en ligne, consolide chaque mois trois exports Excel chargés avec pandas : l'ERP (prix, stock, prix d'achat), l'export WordPress du site (ventes, descriptions) et une table de liaison entre product_id (ERP) et sku (site).
> Structure des tables : [structure des tables, colonnes et types uniquement]
> Règles à implémenter : [catalogue des 15 règles, voir document Tâche 2]
> Contraintes : Pandera (DataFrameSchema) avec lazy=True ; un schéma par table plus des contrôles inter-tables (R5, R6, R7, R12, R13) ; distinction bloquant/avertissement ; chaque contrôle nommé avec son identifiant de règle ; commentaires en français ; détecter sans corriger ; signaler les règles difficiles à exprimer avec Pandera.

<details>
<summary>Sortie complète</summary>

```text
Implémentation des règles de qualité des données avec Pandera

Pour BottleNeck, je recommande une validation en trois étapes :

Validation individuelle des tables erp, web et liaison avec un DataFrameSchema Pandera par table.

Contrôles métier inter-tables après les jointures, pour vérifier la cohérence entre les produits ERP et ceux du site.

Rapport consolidé qui recense toutes les anomalies, en distinguant les erreurs bloquantes des avertissements, sans modifier les données sources.

Le point important est que lazy=True permet à Pandera de collecter plusieurs erreurs au cours d'une validation, plutôt que de s'arrêter à la première. Pour les contrôles inter-tables et certaines règles conditionnelles, des fonctions pandas complémentaires sont plus adaptées.

1. Correspondance entre les règles et les contrôles

D'après ton catalogue, voici l'organisation proposée.

Règle | Contrôle | Niveau
R1 | Prix de vente strictement positif | Bloquant
R2 | Quantité en stock positive ou nulle | Bloquant
R3 | Cohérence entre quantité et statut de stock | Avertissement
R4 | Prix de vente supérieur ou égal au prix d'achat | Bloquant, sauf déstockage autorisé
R5 | Format numérique cohérent des identifiants web | Bloquant
R6 | Un produit en vente sur le site doit avoir un identifiant web | Avertissement
R7 | Un produit retiré de la vente web ne doit plus avoir d'identifiant web actif | Avertissement
R8 | Unicité des produits dans l'export web | Bloquant
R9 | Référence du site présente et ventes non négatives | Bloquant
R10 | Absence de colonnes vides ou inutilisées | Avertissement
R11 | Absence de code HTML dans les titres et descriptions | Avertissement
R12 | Toute référence de liaison existe sur le site | Avertissement
R13 | Tout produit vendu sur le site existe dans l'ERP | Bloquant
R14 | Type dédié pour les produits qui ne sont pas des vins | Avertissement
R15 | Libellé ERP renseigné | Avertissement

Hypothèses à expliciter : la règle R7 suppose que l'on peut identifier un produit retiré de la vente web grâce à onsale_web dans l'ERP. La règle R4 nécessite également une liste des déstockages autorisés par Nicolas. Enfin, pour R6 et R7, il faut préciser si un identifiant « actif » désigne un sku non nul dans la table de liaison ou une publication effectivement active sur WordPress.

2. Code Python complet

Le code ci-dessous suppose que les trois DataFrames erp, web et liaison sont déjà chargés. Il utilise pandera.DataFrameSchema, collecte les anomalies et conserve les données intactes.

Installation, si nécessaire :

pip install pandera pandas
A. Préparer les contrôles et le rapport d'anomalies
import pandas as pd
import pandera as pa
from pandera import Column, Check, DataFrameSchema
import re
from html import unescape

# Liste des product_id dont le prix inférieur au prix d'achat
# a été explicitement autorisé par Nicolas.
# À renseigner avec les identifiants réellement validés.
DESTOCKAGES_AUTORISES = set()

# Stockage des anomalies détectées.
anomalies = []


def enregistrer_anomalie(
    regle, gravite, table, lignes, message
):
    """Enregistre les lignes en anomalie sans modifier les données."""
    for index in lignes:
        anomalies.append({
            "Règle": regle,
            "Gravité": gravite,
            "Table": table,
            "Index": index,
            "Message": message
        })


def executer_schema(schema, df, nom_table, gravites):
    """
    Exécute un schéma en mode lazy et transforme les erreurs
    Pandera en lignes de rapport.
    """
    try:
        schema.validate(df, lazy=True)
    except pa.errors.SchemaErrors as exc:
        details = exc.failure_cases

        for _, erreur in details.iterrows():
            check = str(erreur.get("check", ""))
            colonne = erreur.get("column", None)
            index = erreur.get("index", None)

            # Les identifiants R1, R2... sont présents dans le nom
            # du contrôle pour permettre de retrouver la règle.
            match = re.search(r"R\d+", check)
            if not match:
                continue

            regle = match.group(0)
            gravite = gravites.get(regle, "Avertissement")

            anomalies.append({
                "Règle": regle,
                "Gravité": gravite,
                "Table": nom_table,
                "Index": index,
                "Colonne": colonne,
                "Message": str(erreur.get("failure_case", check))
            })
B. Schéma ERP

Les règles R1 à R4 concernent les prix et les stocks. Les contrôles conditionnels sont exprimés avec des Check personnalisés.

def prix_positif(df):
    """R1 : le prix de vente doit être strictement positif."""
    return df["price"].notna() & (df["price"] > 0)


def stock_non_negatif(df):
    """R2 : le stock ne doit jamais être négatif."""
    return (
        df["stock_quantity"].notna()
        & (df["stock_quantity"] >= 0)
    )


def statut_stock_coherent(df):
    """R3 : statut en stock cohérent avec la quantité disponible."""
    statut = (
        df["stock_status"]
        .astype("string")
        .str.strip()
        .str.lower()
    )

    # Adapter ces libellés aux valeurs réelles de l'ERP.
    en_stock = statut.isin(["instock", "en stock"])
    rupture = statut.isin([
        "outofstock", "out of stock", "rupture"
    ])

    return (
        (en_stock & (df["stock_quantity"] > 0))
        | (rupture & (df["stock_quantity"] == 0))
    )


def prix_superieur_achat(df):
    """
    R4 : le prix de vente doit être au moins égal au prix d'achat,
    sauf pour les produits de déstockage autorisés.
    """
    exception = df["product_id"].isin(DESTOCKAGES_AUTORISES)

    return (
        df["price"].notna()
        & df["purchase_price"].notna()
        & (
            (df["price"] >= df["purchase_price"])
            | exception
        )
    )


schema_erp = DataFrameSchema(
    columns={
        "product_id": Column(int, nullable=False, unique=True),
        "onsale_web": Column(
            int, Check.isin([0, 1]), nullable=False
        ),
        "price": Column(float, nullable=False),
        "stock_quantity": Column(int, nullable=False),
        "stock_status": Column(str, nullable=False),
        "purchase_price": Column(float, nullable=False),
    },
    checks=[
        Check(prix_positif, name="R1_prix_positif"),
        Check(stock_non_negatif, name="R2_stock_non_negatif"),
        Check(statut_stock_coherent, name="R3_statut_stock"),
        Check(prix_superieur_achat, name="R4_prix_achat"),
    ],
    strict=False,
    coerce=False
)

Attention : unique=True sur product_id constitue ici un contrôle d'unicité complémentaire. Dans le catalogue fourni, il n'a pas d'identifiant R dédié : si cette unicité doit faire partie du rapport métier, il faudra lui attribuer une règle distincte. Le code de rapport ci-dessus ne rattache pas automatiquement ce contrôle à R1–R15.

C. Schéma de l'export web
def reference_numerique(df):
    """R5 : les SKU renseignés doivent avoir un format numérique."""
    sku = df["sku"].astype("string").str.strip()
    return sku.notna() & sku.str.fullmatch(r"\d+", na=False)


def reference_presente(df):
    """R9 : chaque produit doit avoir une référence."""
    sku = df["sku"].astype("string").str.strip()
    return sku.notna() & sku.ne("")


def ventes_non_negatives(df):
    """R9 : les ventes cumulées ne peuvent pas être négatives."""
    return (
        df["total_sales"].notna()
        & (df["total_sales"] >= 0)
    )


def titres_descriptions_sans_html(df):
    """R11 : les titres et descriptions ne doivent pas contenir de HTML."""
    colonnes = [
        c for c in ["post_title", "post_excerpt"]
        if c in df.columns
    ]

    resultat = pd.Series(True, index=df.index)

    for colonne in colonnes:
        texte = df[colonne].astype("string")
        resultat &= ~texte.str.contains(
            r"<[^>]+>", regex=True, na=False
        )

    return resultat


schema_web = DataFrameSchema(
    columns={
        "sku": Column(object, nullable=False),
        "total_sales": Column(float, nullable=False),
        "post_title": Column(str, nullable=True),
        "post_excerpt": Column(str, nullable=True),
    },
    checks=[
        Check(reference_numerique, name="R5_sku_numerique"),
        Check(reference_presente, name="R9_sku_present"),
        Check(ventes_non_negatives, name="R9_ventes_non_negatives"),
        Check(titres_descriptions_sans_html, name="R11_html_absent"),
    ],
    strict=False,
    coerce=False
)

Pour R8, le contrôle d'unicité doit s'appliquer au SKU. Comme sku est une référence textuelle, il vaut mieux contrôler les valeurs manquantes et les doublons séparément plutôt que d'utiliser une contrainte d'unicité qui pourrait échouer avant de rendre le rapport détaillé.

D. Schéma de la table de liaison
def id_web_numerique(df):
    """R5 : les identifiants web de la liaison sont numériques."""
    identifiant = df["id_web"].astype("string").str.strip()
    return (
        identifiant.isna()
        | identifiant.eq("")
        | identifiant.str.fullmatch(r"\d+", na=False)
    )


schema_liaison = DataFrameSchema(
    columns={
        "id_web": Column(object, nullable=True),
        "product_id": Column(int, nullable=False),
    },
    checks=[
        Check(id_web_numerique, name="R5_id_web_numerique"),
    ],
    strict=False,
    coerce=False
)

Le format des identifiants doit être harmonisé avec la réalité du site : si certains SKU sont alphanumériques, le contrôle numérique R5 devra être adapté. Il faut éviter de convertir automatiquement les identifiants en nombres, car cela pourrait modifier leur représentation.

E. Contrôles inter-tables et règles complémentaires

Ces contrôles sont exécutés après les trois schémas. Ils permettent de vérifier les correspondances sans effectuer de correction ni de jointure destructive.

# Gravité issue du catalogue de règles.
GRAVITES = {
    "R1": "Bloquante",
    "R2": "Bloquante",
    "R3": "Avertissement",
    "R4": "Bloquante",
    "R5": "Bloquante",
    "R6": "Avertissement",
    "R7": "Avertissement",
    "R8": "Bloquante",
    "R9": "Bloquante",
    "R10": "Avertissement",
    "R11": "Avertissement",
    "R12": "Avertissement",
    "R13": "Bloquante",
    "R14": "Avertissement",
    "R15": "Avertissement",
}


def ajouter(regle, table, index, message):
    """Ajoute une anomalie au rapport central."""
    anomalies.append({
        "Règle": regle,
        "Gravité": GRAVITES[regle],
        "Table": table,
        "Index": index,
        "Message": message,
    })


def verifier_inter_tables(erp, web, liaison):
    """Exécute les contrôles de cohérence entre les trois tables."""

    # Normalisation temporaire des références pour comparer les tables.
    # Les DataFrames d'origine ne sont pas modifiés.
    e = erp.copy()
    w = web.copy()
    l = liaison.copy()

    e["_product_id"] = e["product_id"].astype("string").str.strip()
    l["_product_id"] = l["product_id"].astype("string").str.strip()
    l["_id_web"] = l["id_web"].astype("string").str.strip()
    w["_sku"] = w["sku"].astype("string").str.strip()

    # Références web réellement renseignées.
    ids_liaison = set(
        l.loc[
            l["_id_web"].notna() & l["_id_web"].ne(""),
            "_id_web"
        ]
    )
    skus_web = set(
        w.loc[
            w["_sku"].notna() & w["_sku"].ne(""),
            "_sku"
        ]
    )

    # R5 : un identifiant renseigné doit être numérique dans les deux tables.
    for table, df, colonne in [
        ("web", w, "_sku"),
        ("liaison", l, "_id_web"),
    ]:
        masque = (
            df[colonne].notna()
            & df[colonne].ne("")
            & ~df[colonne].str.fullmatch(r"\d+", na=False)
        )
        for idx in df.index[masque]:
            ajouter("R5", table, idx, "Identifiant web non numérique")

    # R6 et R7 : contrôle entre le statut de vente ERP et la liaison.
    liaison_par_produit = l.groupby("_product_id")["_id_web"].apply(
        lambda s: s.notna().any() and s.ne("").any()
    )

    for idx, ligne in e.iterrows():
        pid = ligne["_product_id"]
        a_un_id_web = bool(liaison_par_produit.get(pid, False))

        if ligne["onsale_web"] == 1 and not a_un_id_web:
            ajouter(
                "R6", "ERP / liaison", idx,
                "Produit en vente web sans identifiant web"
            )

        if ligne["onsale_web"] == 0 and a_un_id_web:
            ajouter(
                "R7", "ERP / liaison", idx,
                "Produit hors vente web avec identifiant web renseigné"
            )

    # R8 : chaque SKU doit apparaître une seule fois dans l'export.
    sku_valide = w["_sku"].notna() & w["_sku"].ne("")
    doublons = w.loc[sku_valide, "_sku"].duplicated(keep=False)

    for idx in w.loc[sku_valide].index[doublons]:
        ajouter(
            "R8", "web", idx,
            f"SKU dupliqué : {w.at[idx, '_sku']}"
        )

    # R10 : colonnes entièrement vides dans l'export.
    # Une colonne numérique remplie de NaN est également considérée vide.
    for colonne in web.columns:
        serie = web[colonne]
        vide = (
            serie.isna()
            | serie.astype("string").str.strip().eq("").fillna(False)
        )

        if vide.all():
            ajouter(
                "R10", "web", colonne,
                f"Colonne entièrement vide : {colonne}"
            )

    # R11 : détecte du HTML dans les titres et descriptions.
    for colonne in ["post_title", "post_excerpt"]:
        if colonne not in web.columns:
            continue

        masque_html = (
            web[colonne]
            .astype("string")
            .str.contains(r"<[^>]+>", regex=True, na=False)
        )

        for idx in web.index[masque_html]:
            ajouter(
                "R11", "web", idx,
                f"Code HTML détecté dans {colonne}"
            )

    # R12 : chaque identifiant de liaison doit exister dans l'export web.
    for idx, ligne in l.iterrows():
        ref = ligne["_id_web"]

        if pd.notna(ref) and ref != "" and ref not in skus_web:
            ajouter(
                "R12", "liaison", idx,
                f"Référence absente de l'export web : {ref}"
            )

    # R13 : tout produit ayant des ventes doit pouvoir être relié à l'ERP.
    produits_erp = set(e["_product_id"].dropna())

    ventes = pd.to_numeric(web["total_sales"], errors="coerce")
    refs_vendues = set(w.loc[ventes > 0, "_sku"].dropna())

    liaison_vendue = l[l["_id_web"].isin(refs_vendues)]

    refs_reliees = set(liaison_vendue["_id_web"].dropna())
    for idx, ligne in liaison_vendue.iterrows():
        if ligne["_product_id"] not in produits_erp:
            ajouter(
                "R13", "ERP / liaison", idx,
                f"Produit vendu absent de l'ERP : {ligne['_product_id']}"
            )

    # R13 : les SKU vendus doivent aussi avoir une correspondance ERP.
    refs_sans_liaison = refs_vendues - refs_reliees
    for ref in refs_sans_liaison:
        ajouter(
            "R13", "web / liaison", ref,
            f"Produit vendu sans correspondance ERP : {ref}"
        )

    # R14 : le contrôle nécessite de connaître les types autorisés
    # et les produits considérés comme non-vins.
    # Voir les limites expliquées sous le code.


def verifier_regles_erp(erp):
    """Contrôles complémentaires sur les règles ERP."""

    # R1 à R4 sont exécutées par le schéma Pandera.
    # R15 ne peut être vérifiée que si une colonne de libellé existe.
    colonnes_libelle = [
        c for c in ["libelle", "product_name", "nom_produit", "name"]
        if c in erp.columns
    ]

    if not colonnes_libelle:
        print(
            "R15 non exécutable : aucune colonne de libellé "
            "n'est présente dans le schéma ERP fourni."
        )
        return

    colonne = colonnes_libelle[0]
    masque = (
        erp[colonne].isna()
        | erp[colonne].astype("string").str.strip().eq("").fillna(False)
    )

    for idx in erp.index[masque]:
        ajouter("R15", "ERP", idx, "Libellé produit vide ou manquant")
F. Lancer les validations
# 1. Valider chaque table indépendamment.
# Les erreurs de chaque schéma sont collectées en mode lazy.
executer_schema(schema_erp, erp, "ERP", GRAVITES)
executer_schema(schema_web, web, "Web", GRAVITES)
executer_schema(schema_liaison, liaison, "Liaison", GRAVITES)

# 2. Appliquer les contrôles complémentaires.
verifier_inter_tables(erp, web, liaison)
verifier_regles_erp(erp)

# 3. Produire un rapport consolidé.
rapport = pd.DataFrame(anomalies)

if rapport.empty:
    print("Aucune anomalie détectée par les contrôles exécutés.")
else:
    rapport = rapport.drop_duplicates()

    # Afficher les erreurs bloquantes en premier.
    ordre = {"Bloquante": 0, "Avertissement": 1}
    rapport["_ordre"] = rapport["Gravité"].map(ordre)

    rapport = (
        rapport.sort_values(["_ordre", "Règle"])
        .drop(columns="_ordre")
        .reset_index(drop=True)
    )

    print("Nombre d'anomalies :", len(rapport))
    print("\nRépartition par règle :")
    print(rapport.groupby(["Règle", "Gravité"]).size())

    print("\nDétail des anomalies :")
    display(rapport)

    # Distinguer les anomalies bloquantes des avertissements.
    bloquantes = rapport[rapport["Gravité"] == "Bloquante"]
    avertissements = rapport[
        rapport["Gravité"] == "Avertissement"
    ]

    print("\nAnomalies bloquantes :", len(bloquantes))
    print("Avertissements :", len(avertissements))
3. Règles nécessitant une adaptation

Certaines règles ne peuvent pas être implémentées de manière fiable sans informations métier supplémentaires.

Règle | Limite et solution recommandée
R3 | Les libellés exacts de stock_status doivent être confirmés. Le contrôle proposé couvre les cas « en stock » et « rupture », mais pas nécessairement tous les statuts.
R4 | La liste des déstockages autorisés doit être fournie par Nicolas. Sans cette liste, il est impossible de distinguer une exception validée d'une anomalie.
R5 | Vérifier que les identifiants sont numériques ne garantit pas qu'ils ont exactement le même format dans les deux tables. Une comparaison de leur représentation normalisée est préférable.
R6–R7 | Le statut de vente et l'identifiant web doivent être interprétés selon les conventions métier. La présence d'un SKU dans la liaison ne prouve pas à elle seule que le produit est publié sur WordPress.
R10 | Une colonne entièrement vide est détectable automatiquement. En revanche, une colonne « inutilisée » nécessite une liste de colonnes autorisées ou une documentation de l'export.
R11 | La détection par expression régulière repère les balises HTML classiques. Elle ne constitue pas un analyseur HTML complet.
R13 | La règle est contrôlée à partir des produits ayant total_sales > 0 et de la table de liaison. Il faut aussi décider comment traiter les ventes sans SKU ou les doublons de liaison.
R14 | La règle nécessite une colonne de catégorisation fiable et la liste des catégories autorisées pour les cadeaux non-vins.
R15 | Aucune colonne de libellé n'est présente dans la structure ERP communiquée. Il faut identifier le nom réel de cette colonne avant d'activer le contrôle.

4. Point important sur lazy=True

lazy=True collecte les échecs de validation d'un schéma donné, mais ne garantit pas que toutes les règles métier seront évaluées si une structure est incompatible avec le schéma. C'est pourquoi le code sépare les validations Pandera et les contrôles inter-tables.

Pour aller plus loin, je recommande de faire évoluer le rapport afin qu'il contienne systématiquement :

l'identifiant de la règle ;

la gravité ;

la table et la ligne concernées ;

la valeur observée ;

le message d'anomalie ;

le nombre d'occurrences par règle.

Enfin, il faudra vérifier la version de Pandera utilisée et tester les schémas sur les exports réels : certaines colonnes du site, comme post_content, sont déclarées float64, ce qui peut indiquer qu'elles sont entièrement vides dans l'export. Il vaut mieux ne pas imposer de conversion de type avant d'avoir confirmé leur contenu réel.
```

</details>

Observations : les 15 règles sont couvertes ou discutées, la gravité est attribuée, les contrôles sont nommés R1 à R15 et le rapport est consolidé. L'IA identifie à juste titre que R15 n'est pas exécutable, faute de colonne de libellé dans l'ERP : la règle est à reformuler. Défauts à confirmer par les tests : les erreurs sans identifiant de règle sont ignorées silencieusement dans le rapport, les lignes « attachment » ne sont pas filtrées (risque de faux doublons pour R8), R10 ignore les colonnes remplies de zéros, R11 ignore les entités HTML (&amp;), R14 n'est pas implémentée, l'import Pandera utilise une ancienne syntaxe, et le code fait environ 300 lignes.

Décision : retenu comme base de travail, sous réserve des tests sur données réelles (ticket #5).


## Essai 3 – 9 octobre 2026

Outil et modèle : code généré à l'essai 2 (ChatGPT, version gratuite), exécuté sans modification dans le notebook test_chatgpt_promptB.ipynb

Objectif : tester le code de l'IA sur l'export réel d'octobre et le comparer aux règles construites pas à pas (notebook bottleneck_qualite.ipynb)

Environnement : pandas 3.0.2, Pandera 0.34.1

Résultat : le code s'exécute sans erreur en 0,2 seconde, mais annonce 5 916 anomalies contre environ 1 170 pour la construction guidée.

| Règle | ChatGPT | Construction guidée | Explication de l'écart |
|---|---|---|---|
| R1 | 18 | 3 | Une ligne par colonne (3 produits × 6 colonnes) |
| R2 | 12 | 2 | Même effet (2 × 6) |
| R3 | 24 | 2 | Stocks négatifs comptés en doublon de R2 |
| R4 | 42 | 4 | Prix négatifs comptés en doublon de R1 |
| R5 | 401 | 3 + 2 | Lignes d'images et lignes vides comptées |
| R8 | 1 428 | 797 | Faux doublons produit/image, vrai problème non nommé |
| R9 | 295 | 2 | Lignes d'images et lignes vides comptées |
| R10 | 4 | 4 + 12 | Colonnes constantes non détectées |
| R11 | 3 648 | 183 | Lignes d'images comptées, entités HTML ignorées |
| R6, R7, R12, R13 | 3, 21, 20, 0 | 3, 21, 20, 0 | Identique |
| R14 | Non implémentée | 1 | – |
| R15 | Non exécutable | 111 | – |

Observations : le code de l'IA est juste sur les règles inter-tables, mais gonfle les autres pour trois raisons : le format d'erreur de Pandera (une ligne par colonne), l'absence de filtrage des lignes non-produits et des règles qui se chevauchent. Un rapport de 5 916 lignes serait inutilisable par l'équipe métier.

| Critère | ChatGPT (prompt B) | Construction guidée |
|---|---|---|
| Erreurs réelles détectées | Toutes sauf R14, R15 et entités HTML | Toutes |
| Faux positifs et doublons | Très nombreux | Aucun constaté |
| Lisibilité du résultat | Faible | Bonne |
| Temps de mise en œuvre | Très court | Plusieurs heures |
| Taille du code | Environ 300 lignes | Environ 100 lignes |
| Maintenabilité | Faible | Bonne |

Décision : la construction guidée est retenue. Le prompt structuré reste utile pour amorcer le travail et a permis de repérer le défaut de la règle R15, mais le code généré ne peut pas être utilisé sans relecture ni test sur les données réelles.
