---
sourceHash: 3fe6055af094fd8e7406ff91d6def456705efa046e600819010957d5069d33a9
sourcePath: docs/guides/connect-an-ai-assistant.md
---

# Connecter un assistant IA à votre compte

Demandez à Claude ou à ChatGPT « combien de PETG il me reste ? » et obtenez la
réponse depuis votre propre inventaire. Le connecteur TigerTag permet à un
assistant IA de **lire** votre compte — rien de plus. C'est la même idée que le
serveur local de Tiger Studio, mais hébergée : ça marche depuis le web, depuis
un téléphone, et sans que Studio soit ouvert.

**L'adresse du connecteur :**

```
https://mcp.tigersystem.io/mcp
```

Il parle MCP (Model Context Protocol), le standard ouvert que les assistants
utilisent pour accéder à des outils externes. Tout assistant qui gère les
serveurs MCP distants avec OAuth peut l'utiliser.

## Ce que l'assistant peut lire

- vos bobines : marque, matière, couleurs, poids restant, températures, et où
  chacune est rangée ;
- vos racks, vos listes d'envies et l'historique de votre stock ;
- vos imprimantes — **codes d'accès compris** — et vos TigerScales ;
- ce que vos amis partagent avec vous (leurs bobines, racks et listes d'envies).

Il **ne peut rien modifier** : chaque outil est en lecture seule. Il lit en
votre nom, avec les mêmes règles d'accès que les apps, donc il ne voit jamais
un autre compte. Les secrets de votre compte (clé privée, e-mail) ne sont
jamais renvoyés.

## Claude (web, bureau, mobile)

1. **Réglages → Connecteurs → Ajouter un connecteur personnalisé.**
2. Nommez-le `TigerTag`, collez l'adresse ci-dessus, laissez vides les champs
   OAuth avancés, et ajoutez-le.
3. Cliquez **Connecter**. Vous arrivez sur tigersystem.io : connectez-vous,
   vérifiez que la page indique que les réponses partent vers claude.ai, et
   cliquez **Autoriser**.
4. Dans une conversation, activez le connecteur TigerTag depuis le menu des
   outils et posez votre question.

## ChatGPT

Les connecteurs personnalisés demandent le **mode développeur** (offre payante ;
sur les offres Business, c'est un administrateur qui l'active).

1. **Réglages → Apps et connecteurs → Paramètres avancés → Mode développeur**
   activé.
2. **Créez** un connecteur : nom `TigerTag`, l'adresse ci-dessus,
   authentification **OAuth**, sans identifiant ni secret client.
3. Connectez-vous sur tigersystem.io et cliquez **Autoriser**.
4. Dans une nouvelle conversation, activez le connecteur et posez votre
   question.

Les noms des menus changent souvent dans ces apps ; les étapes restent les
mêmes.

## Codex, Cursor et les autres apps de bureau

Ajoutez un serveur MCP distant de type **Streamable HTTP** (« diffusion HTTP »)
avec l'adresse ci-dessus et sans jeton. L'app ouvre votre navigateur pour la
connexion ; après **Autoriser**, le navigateur arrive sur une adresse
`127.0.0.1` — c'est l'app de votre ordinateur qui récupère la connexion. Une
page vide ou « impossible de se connecter » à cet endroit est normale : fermez
l'onglet.

## Couper l'accès d'un assistant

**tigersystem.io → votre compte → Données → Assistants connectés.** Chaque
assistant autorisé y figure, avec sa date de connexion et sa dernière
utilisation. **Couper l'accès** l'arrête immédiatement.

## Bon à savoir

- L'assistant peut répondre avec des données vieilles de deux minutes au plus :
  les lectures récentes sont gardées brièvement pour rester rapide et léger.
- L'état en direct des imprimantes (en ligne, en impression) n'est pas
  disponible depuis le cloud — seul Tiger Studio, sur le réseau de votre
  imprimante, le voit.
- Les notes, noms de couleurs et textes de listes sont vos mots (ou ceux d'un
  ami) ; l'assistant a pour consigne de les rapporter, jamais de les suivre
  comme des instructions.

---

**▲ [Index de la documentation](../../README.md)** · **Voir aussi :** [Tiger Studio](../products/tiger-studio.md), [Guides](./README.md)
