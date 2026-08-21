# Site — Test civique (à venir)

Page de préfiguration pour la seconde application du studio : préparation à l'examen
civique, obligatoire depuis le 1er janvier 2026 pour la carte de séjour pluriannuelle, la
carte de résident et la naturalisation.

L'application n'existe pas encore. Ce dépôt ne contient qu'**une page statique unique** —
pas de pages légales, pas de générateur — volontairement : inventer une politique de
confidentialité ou des mentions légales pour un produit qui n'a ni code ni traitement de
données serait prématuré et potentiellement inexact.

Destination : `https://test-civique.hasakistudio.fr`

## Organisation

```
public/                SEUL dossier publié sur le web
  index.html            La page, en HTML/CSS pur — aucun JavaScript
  assets/site.css        Feuille de style, reprise de la palette du premier site
  _headers               En-têtes de sécurité (CSP, etc.)
  robots.txt
  favicon.svg, favicon.ico, apple-touch-icon.png   Identité visuelle du studio, identique
                                                     à naturalisation.hasakistudio.fr
```

Pas de `content/`, pas de `build/` : une seule page ne justifie pas de générateur. Le jour
où l'application existe, reprendre la structure de
[`hasaki-studio/site-nat-civ`](https://github.com/hasaki-studio/site-nat-civ) — générateur
Markdown, pages légales, sitemap — plutôt que de faire grossir celui-ci.

## Déployer

Cloudflare Pages, sans étape de build.

*Workers & Pages → Create application → Pages → Connect to Git* → dépôt `site-civique`.

| Champ | Valeur |
|---|---|
| Production branch | `main` |
| Framework preset | `None` |
| Build command | *(vide)* |
| Build output directory | `public` |

Puis *Custom domains* → `test-civique.hasakistudio.fr`. La zone `hasakistudio.fr` est déjà
chez Cloudflare (voir `site-nat-civ`), l'enregistrement DNS et le certificat se créent
automatiquement.

## Quand l'application sera prête

Trois choses à ajouter, dans cet ordre :

1. **Les pages légales réelles** — mentions légales et politique de confidentialité,
   spécifiques à cette application (nom, éditeur, données traitées). Ne pas dupliquer
   celles de `site-nat-civ` telles quelles : les traitements ne seront pas forcément les
   mêmes (Firebase, AdMob, achats intégrés — à vérifier au cas par cas).
2. **Un lien réciproque** avec `naturalisation.hasakistudio.fr` — l'examen civique précède
   souvent l'entretien de naturalisation de deux ans, les deux applications visent en
   partie le même public à des moments différents.
3. **Les URL Play Console / AdMob**, une fois les pages légales rédigées — ne les saisir
   qu'après avoir vérifié la page en navigation privée.
