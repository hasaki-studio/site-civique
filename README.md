# Site — Réussir mon examen civique

Site public de l'application Android « Réussir mon examen civique » (Hasaki Studio). Il
porte la **politique de confidentialité** et les **mentions légales** exigées par la Play
Console, par AdMob et par le droit français.

Site statique, sans dépendance et sans requête vers un service tiers. Structure et
générateur repris de [`hasaki-studio/site-nat-civ`](https://github.com/hasaki-studio/site-nat-civ)
(le site de « Réussir mon entretien de naturalisation »).

Destination : `https://examencivique.hasakistudio.fr`

## URL à déclarer

| Où | URL |
|---|---|
| Play Console → Contenu de l'application → Politique de confidentialité | `https://examencivique.hasakistudio.fr/confidentialite` |
| AdMob → Confidentialité et messages (RGPD) | `https://examencivique.hasakistudio.fr/confidentialite` |
| Écran « Mentions légales » de l'app | `https://examencivique.hasakistudio.fr/mentions-legales` |

Ces adresses sont revérifiées périodiquement par Google. **Ne les changez plus** une fois
saisies : une URL modifiée ou tombée en 404 peut suspendre la fiche Play Store.

## Organisation

```
content/                Sources éditoriales en Markdown — c'est ici qu'on écrit
build/build.py          Générateur des pages (Python 3, aucune dépendance)
build/index.body.html   Corps de la page d'accueil, conservé tel quel

public/                 SEUL dossier publié sur le web
  *.html                Pages générées — ne pas éditer directement
  assets/site.css       Feuille de style — copie de celle de site-nat-civ, ne PAS
                        remplacer par une version allégée : le gabarit ci-dessous en
                        réutilise les classes telles quelles.
  assets/site.js        Bascule des blocs repliables (bouton « Voir l'explication »)
  sitemap.xml           Généré
  robots.txt            Généré
  _headers              En-têtes de sécurité — script-src et style-src 'unsafe-inline'
                        nécessaires : le gabarit copié du site sœur utilise des styles
                        en ligne et un script pour l'interactivité de la carte d'accueil.
  favicon.svg, favicon.ico, apple-touch-icon.png   Identité visuelle du studio
```

Le contenu se modifie dans `content/*.md`, jamais dans les `.html` de `public/`, qui sont
écrasés à chaque génération.

**Ne pas remplacer `public/assets/site.css`** par une feuille de style écrite à la main :
ce fichier est un export Tailwind *purgé* — il ne contient que les classes utilitaires
réellement utilisées quelque part dans le site au moment de sa génération, pas un
Tailwind complet. `build/index.body.html` et le gabarit de `build/build.py` (en-tête,
pied de page) dépendent de classes précises de ce fichier ; les remplacer par une CSS
maison casserait silencieusement leur mise en page.

## Générer

```bash
python3 build/build.py
```

Régénère les 6 pages, `sitemap.xml` et `robots.txt`. Le domaine, l'adresse de contact, le
nom et le package de l'application sont définis en tête de `build/build.py` — un seul
endroit à modifier.

## Prévisualiser

```bash
cd public && python3 - <<'EOF'
import http.server, os, socketserver
class H(http.server.SimpleHTTPRequestHandler):
    def translate_path(self, path):
        p = super().translate_path(path)
        if not os.path.exists(p) and not p.endswith(('/', '.html')) and os.path.exists(p + '.html'):
            return p + '.html'
        return p
socketserver.TCPServer.allow_reuse_address = True
socketserver.TCPServer(("127.0.0.1", 8899), H).serve_forever()
EOF
```

`_headers` n'est appliqué que par Cloudflare : cette prévisualisation locale ne révèle pas
un problème de CSP (styles ou scripts bloqués). Il faut regarder le site déployé pour ça.

## Déployer

Cloudflare Pages, sans étape de build.

*Workers & Pages → Create application → Pages → Connect to Git* → dépôt `site-civique`.

**Attention à ne pas partir sur un Worker par erreur** — deux pièges rencontrés en le
mettant en place :
- La galerie de « templates » crée un **nouveau** dépôt GitHub plutôt que d'utiliser
  celui-ci ; choisir explicitement l'import d'un dépôt existant.
- Selon l'écran, Cloudflare peut proposer un flux « Create a Worker » avec
  `npx wrangler deploy` plutôt qu'un vrai flux Pages (build output directory). Si c'est le
  seul chemin proposé, ce dépôt fonctionne aussi en Worker à ressources statiques —
  `wrangler.jsonc` sert exactement à ça (voir plus bas).

| Champ (flux Pages) | Valeur |
|---|---|
| Production branch | `main` |
| Framework preset | `None` |
| Build command | *(vide)* |
| Build output directory | `public` |

Puis *Custom domains* → `examencivique.hasakistudio.fr`. La zone `hasakistudio.fr` est
déjà chez Cloudflare (voir `site-nat-civ`), l'enregistrement DNS et le certificat se
créent automatiquement.

### Variante Worker

Si le flux qui se présente est « Create a Worker » plutôt que Pages, ce dépôt a aussi un
`wrangler.jsonc` à la racine (assets → `./public`) qui fonctionne avec
`npx wrangler deploy`. C'est ce qui a effectivement servi au premier déploiement de ce
site.

## Avant la mise en ligne

- [ ] **Durée de conservation Firebase Analytics** (politique, §3.2) — relever le réglage
      dans la console Firebase (2 ou 14 mois)
- [ ] **Liens officiels** de `content/conseils.md` — les cliquer un par un ; ils n'ont pas
      pu être vérifiés automatiquement dans cet environnement
- [ ] **Vérifier les traitements de données** décrits dans `content/confidentialite.md` et
      `content/mentions-legales.md` une fois l'application terminée : ce texte a été
      recopié de `site-nat-civ` (Firebase Analytics, AdMob bandeau + récompense, achat
      Premium unique) sur la base qu'ils seraient identiques. Si l'app finale diffère —
      pas de pub, abonnement au lieu d'achat unique, pas d'Analytics — corriger avant de
      déclarer l'URL à la Play Console : une politique de confidentialité qui ne décrit
      pas les traitements réels est le genre d'erreur qui bloque une validation.
- [ ] **Immatriculation** — même point de vigilance que sur `site-nat-civ` : à
      l'obtention du SIREN, remplacer la mention personne physique par « Hasaki Studio,
      micro-entreprise » + SIREN + adresse professionnelle, dans les mêmes fichiers sur
      les deux dépôts.
- [ ] **Thèmes du programme** listés sur la page d'accueil — repris de ceux de
      l'application sœur (Principes de la République, institutions, histoire, etc.),
      génériques et non spécifiques à la répartition exacte des 40 questions. À ajuster
      une fois le contenu réel de l'application connu.

## Éditeur, domicile, hébergeur

Identiques à `site-nat-civ` par choix explicite : Achraf AZOUZI (personne physique),
même adresse de réexpédition, même téléphone, Cloudflare comme hébergeur,
`contact@hasakistudio.fr`. Toute mise à jour de ces informations (immatriculation,
changement d'adresse) doit être répercutée **sur les deux dépôts**.

## Liens vers les magasins, le jour de la publication

`build/index.body.html` et `content/conseils.md` affichent « Bientôt disponible » et
pointent vers l'ancre `/#application`, faute d'application publiée. Le jour de la mise en
ligne, les remplacer par les adresses définitives, puis régénérer et redéployer :

    Google Play   https://play.google.com/store/apps/details?id=com.hasakistudio.examencivique
    App Store     https://apps.apple.com/app/id6804383682

Ne jamais publier une adresse `appstoreconnect.apple.com` : c'est le back-office privé,
pas la fiche publique.

