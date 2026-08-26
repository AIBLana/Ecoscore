# Vue d'ensemble

## Qu'est-ce que l'EcoScore IA ?

Un outil d'estimation de l'empreinte carbone annuelle d'un usage de LLM (grand modèle de langage). Il produit une note de A à G — comme l'étiquette énergie d'un appareil électroménager — en tenant compte de l'incertitude inhérente aux données disponibles.

## Comment ça marche en 3 étapes

1. **Vous décrivez votre usage** — Nombre d'utilisateurs, fréquence d'utilisation, longueur des échanges, modèle IA utilisé, localisation du datacenter, présence d'un RAG ou d'un fine-tuning.
2. **L'outil teste des milliers de variantes** — Certaines données ne sont connues que par fourchette : la consommation exacte du modèle d'IA n'est pas publiée par les fournisseurs, et le coût de fabrication du matériel est estimé. Plutôt que de choisir une valeur au hasard, l'outil recalcule l'empreinte avec des milliers de combinaisons plausibles de ces hypothèses. Résultat : une **fourchette probable** plutôt qu'un chiffre unique trompeur.
3. **L'empreinte est convertie en classe A–G** — L'estimation centrale détermine la classe, de A (très sobre) à G (critique). Si la fourchette probable chevauche deux classes, l'outil le signale : la note peut alors basculer selon les hypothèses retenues.

## Ce qui est mesuré

- **🌡️ CO₂ équivalent (tCO₂e)** — Empreinte carbone totale en tonnes de CO₂ équivalent par an. C'est l'indicateur principal du score. Il intègre la consommation électrique multipliée par l'intensité carbone du réseau local.
- **⚡ Électricité (kWh)** — Énergie totale consommée par an par le datacenter pour faire tourner votre usage IA. Inclut le coefficient PUE qui tient compte des pertes liées au refroidissement et à l'infrastructure.
- **💧 Eau (litres)** — Consommation d'eau pour refroidir les serveurs (Scope 1 direct) et pour produire l'électricité utilisée (Scope 2 indirect, ex. centrales thermiques).

## Les classes EcoScore

| Classe | Seuil | Exemple |
|---|---|---|
| A | < 1 tCO₂e | Usage très sobre — équivalent à quelques vols courts |
| B | 1 – 5 tCO₂e | Sobre — comparable à la voiture annuelle d'un Français |
| C | 5 – 25 tCO₂e | Modéré — à surveiller et optimiser |
| D | 25 – 100 tCO₂e | Significatif — plan de réduction recommandé |
| E | 100 – 500 tCO₂e | Élevé — révision architecture nécessaire |
| F | 500 – 1 000 tCO₂e | Très élevé |
| G | ≥ 1 000 tCO₂e | Critique — équivalent à des centaines de vols long-courrier |

La sous-graduation (A+, A, A−) indique la position dans la bande : A+ est la partie basse (plus sobre), A− la partie haute.

## Glossaire des termes clés

- **Token** : Unité de base traitée par un LLM. Environ 0,75 mot en français. Un message de 200 mots ≈ 260 tokens. Le modèle consomme de l'énergie pour lire chaque token d'entrée (encoding) et pour générer chaque token de réponse (decoding).
- **LLM** : Large Language Model — grand modèle de langage (GPT-4, Claude, Gemini…). Sa taille (nombre de paramètres) influence directement sa consommation énergétique par token généré.
- **RAG** : Retrieval-Augmented Generation. Technique où le modèle consulte une base documentaire à chaque requête pour enrichir sa réponse. Génère une empreinte "Run" supplémentaire (recherche à chaque appel) et une empreinte "Build" (indexation initiale de la base).
- **Fine-tuning** : Ré-entraînement d'un modèle sur des données spécifiques. Opération ponctuelle (émission "Build") qui peut être coûteuse en énergie selon le volume d'exemples et le nombre d'epochs.
- **PUE** : Power Usage Effectiveness — efficacité énergétique du datacenter. Un PUE de 1,36 signifie que pour 100 kWh consommés par les serveurs, 136 kWh sont prélevés au réseau (36 % de pertes pour le refroidissement, l'alimentation, etc.). Les meilleurs datacenters approchent 1,1.
- **Intensité carbone** : Quantité de CO₂ émise par kWh d'électricité consommée, exprimée en gCO₂/kWh. Varie selon le mix énergétique du pays : France ≈ 105 g (nucléaire), Allemagne ≈ 615 g (charbon/gaz), Suède ≈ 47 g (hydraulique/nucléaire).
- **WUE / EWIF** : Indicateurs de consommation d'eau. WUE (Water Usage Effectiveness) mesure l'eau utilisée directement pour refroidir les serveurs. EWIF (Energy-Water Impact Factor) estime l'eau consommée indirectement pour produire l'électricité.
- **Estimation centrale & fourchette probable** : L'outil affiche une valeur centrale — la plus représentative des milliers de variantes calculées — et une fourchette dans laquelle l'empreinte réelle a de très fortes chances de se situer. Plus la fourchette est étroite, plus l'estimation est fiable ; si elle chevauche deux classes, la note peut basculer.
- **tCO₂e** : Tonne de CO₂ équivalent. Unité standard pour comparer les émissions de gaz à effet de serre. 1 tCO₂e ≈ 1 vol Paris–New York aller-retour, ou 6 000 km en voiture thermique.

## Comment réduire l'empreinte

- **🤖 Choisir un modèle adapté** — Un petit modèle (SLM 1–10B params) consomme 10 à 100× moins qu'un très grand modèle. Si le cas d'usage le permet, préférer un modèle plus léger.
- **📍 Localiser le datacenter** — Héberger en France ou en Suède (mix bas carbone) plutôt qu'en Allemagne ou en Pologne peut diviser l'empreinte carbone par 5 à 10, à volume égal.
- **✂️ Réduire la longueur des échanges** — Les tokens de réponse (decoding) sont le premier poste de dépense. Raccourcir les réponses ou compresser le prompt système a un impact direct proportionnel.
- **📚 Optimiser le RAG** — Limiter le nombre de documents indexés, réduire la fréquence de refresh, et diminuer le nombre de chunks injectés par requête réduit à la fois l'empreinte Build et Run.

---

# Référence technique

> Cette section s'adresse aux profils techniques : elle détaille les équations, hypothèses statistiques et abaques du modèle. La lecture des résultats ne nécessite pas ces notions — voir la section « Vue d'ensemble » ci-dessus.

## Vue d'ensemble du modèle

L'empreinte totale est la somme de deux composantes indépendantes :

- **Run** — énergie consommée à chaque inférence, cumulée sur 1 an (proportionnelle au volume d'usage).
- **Build** — énergie consommée une seule fois pour construire le système (indexation RAG, fine-tuning). Multipliée par un facteur matériel *mat* qui amortit la fabrication des GPU.

`CO2_total = CO2_run + CO2_build × (1 + mat)`

mat ∈ [0.25, 0.50] — surcoût carbone lié à la fabrication des GPU (incertain, tiré par Monte-Carlo)

> 💡 Toutes les variables incertaines sont modélisées par des lois log-normales. N = 5 000 tirages Monte-Carlo produisent une distribution → percentiles P5 / P25 / P50 / P75 / P95.

## Variables d'entrée

### Usage

| Variable | Description | Unité |
|---|---|---|
| n_users | Nombre d'utilisateurs actifs par an | — |
| n_conv | Conversations par utilisateur par an | — |
| n_msg | Messages échangés par conversation | — |
| req_sz | Taille d'un message utilisateur | tokens |
| resp_sz | Taille d'une réponse du modèle | tokens |
| sys_sz | Taille du system prompt (fixe, répété à chaque appel) | tokens |

### Modèle LLM

| Variable | Description | Unité |
|---|---|---|
| dec_min / dec_max | Énergie de décodage — borne basse (P5) et haute (P95) | Wh / 100k tokens |
| enc | Énergie d'encodage = dec / 100 (l'encodage est parallélisable, ~100× moins coûteux) | Wh / 100k tokens |

### RAG (si activé)

| Variable | Description | Unité |
|---|---|---|
| n_docs | Nombre de documents dans la base | — |
| n_pages | Pages moyennes par document | — |
| n_refresh | Nombre de re-indexations complètes par an | — |

### Fine-tuning (si activé)

| Variable | Description | Unité |
|---|---|---|
| ft_ex | Nombre d'exemples d'entraînement | — |
| ft_ep | Nombre d'epochs | — |

## Comptage des tokens

Nombre total de requêtes sur l'année :

`n_req = n_users × n_conv × n_msg`

Tokens d'entrée totaux (sans historique) :

`tok_req = n_req × (req_sz + sys_sz)`

Avec historique conversationnel activé, chaque message reçoit le contexte de tous les messages précédents — la somme est quadratique en n_msg.

Tokens de sortie totaux :

`tok_resp = n_req × resp_sz`

Si RAG activé, 3 chunks de 750 tokens sont injectés dans chaque requête :

`tok_req += 3 × 750 × n_req`

## Énergie Run (annuelle)

Énergie d'encodage (lecture des tokens d'entrée) et de décodage (génération des tokens de sortie) :

`kWh_enc = enc × tok_req / 10^5`
`kWh_dec = dec × tok_resp / 10^5`

Si RAG activé — énergie d'encodage pour le refresh périodique de l'index (12 mois) :

`kWh_RAG_run = enc × n_refresh × n_pages × 750 / 10^5 × 12`

Application du PUE (pertes datacenter) puis conversion en CO₂ et eau :

`kWh_run = (kWh_enc + kWh_dec + kWh_RAG_run) × PUE`
`CO2_run = kWh_run × carbon / 10^6`
`eau_run = (kWh_enc + kWh_dec + kWh_RAG_run) × (WUE + EWIF)`

> ⚠️ Le PUE s'applique à l'énergie *avant* conversion en CO₂ et eau. L'eau est calculée sur l'énergie brute (hors PUE) car WUE et EWIF intègrent déjà les pertes thermiques.

## Énergie Build (one-shot)

Indexation RAG — encodage de tout le corpus documentaire :

`kWh_RAG_build = enc × (n_docs × n_pages × 750) / 10^5`

Fine-tuning — les tokens d'entrée sont encodés, les tokens de sortie décodés (forward + backward pass ≈ 3×) :

`tok_in = ft_ex × ft_ep × req_sz`
`tok_out = ft_ex × ft_ep × resp_sz`
`kWh_FT = (tok_in × dec + tok_out × dec × 3) / 10^5`

Application du PUE, conversion CO₂, et surcoût matériel (tiré par Monte-Carlo) :

`CO2_build = (kWh_RAG_build + kWh_FT) × PUE × carbon / 10^6 × (1 + mat)`

mat ∈ [0.25, 0.50] — tiré par Monte-Carlo à chaque simulation

## Paramètres Datacenter

Valeurs par défaut utilisées pour chaque pays. PUE et intensité carbone sont les principaux leviers d'action.

| Pays | PUE | gCO₂/kWh | WUE S1 (L/kWh) | EWIF S2 (L/kWh) |
|---|---|---|---|---|
| France | 1.36 | 105 | 1.80 | 1.87 |
| Suède | 1.37 | 47 | 1.50 | 4.53 |
| Allemagne | 1.56 | 615 | 1.20 | 0.75 |
| Irlande | 1.20 | 617 | 1.00 | 0.48 |
| Pologne | 1.50 | 635 | 1.20 | 1.31 |
| États-Unis | 1.43 | 368 | 1.80 | 1.33 |
| Canada | 1.31 | 69 | 1.50 | 6.67 |
| Chine | 1.60 | 534 | 2.00 | 4.01 |
| Japon | 1.62 | 516 | 1.50 | 0.81 |

## Méthode Monte-Carlo

Les paramètres incertains (énergie du modèle *dec*, surcoût matériel *mat*) suivent des lois log-normales calibrées sur les bornes P5/P95 fournies :

`μ = (ln P5 + ln P95) / 2`
`σ = (ln P95 − ln P5) / (2 × 1.6449)`
`X ~ LogNormal(μ, σ²)`

N = 5 000 tirages indépendants. Les résultats sont triés pour extraire les percentiles P5, P25, P50, P75, P95. La classe EcoScore est attribuée à chaque tirage, ce qui donne une distribution de probabilité sur les classes A–G.

> La méthode Box-Muller est utilisée pour générer les tirages aléatoires sans dépendance externe.

## Classes A–G et sous-graduation

Seuils en tCO₂e/an. La sous-graduation découpe chaque bande en tiers selon la position logarithmique :

| Classe | Seuil inf. | Seuil sup. | Sous-graduation |
|---|---|---|---|
| A | 0 | < 1 | A+ (log < 33%) · A · A− (log > 67%) |
| B | 1 | < 5 | B+ · B · B− |
| C | 5 | < 25 | C+ · C · C− |
| D | 25 | < 100 | D+ · D · D− |
| E | 100 | < 500 | E+ · E · E− |
| F | 500 | < 1 000 | F+ · F · F− |
| G | 1 000 | +∞ | G |

`position = (log10(v) − log10(lo)) / (log10(hi) − log10(lo))`

position < 0.33 → "+" (tiers bas) · 0.33–0.67 → neutre · > 0.67 → "−" (tiers haut)
