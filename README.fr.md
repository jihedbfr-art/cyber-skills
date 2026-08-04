<p align="center">
  <img src="assets/jihedailabs-logo.svg" alt="JihedAiLabs" width="150" height="150">
</p>

<h1 align="center">cyber-skills</h1>

<p align="center"><b>Un projet JihedAiLabs</b> — une bibliothèque de skills sécurité regroupées en 26 domaines.</p>

<p align="center"><a href="README.md">English version</a></p>

---

Chaque skill occupe son propre dossier avec un seul `SKILL.md`, lisible de deux façons : comme une checklist qu'un ingénieur déroule, et comme une procédure qu'un assistant de code agentique charge et exécute.

## Pourquoi ce repo

La plupart des ressources sécurité sur GitHub sont des listes de liens ou des listes d'outils. C'est utile, mais l'ordre des opérations reste à votre charge : par quoi commencer, ce qu'un résultat signifie vraiment, quoi faire ensuite.

Ici, c'est la procédure qui est stockée, pas seulement le pointeur. Une skill répond à trois questions : dans quel cas je la sors, qu'est-ce que je lance exactement, et comment je lis la sortie.

## Les 26 domaines

| # | Domaine | Objet |
|---|---|---|
| 01 | [OSINT & Reconnaissance](skills/01-osint-and-reconnaissance) | Découverte de surface d'attaque via sources publiques |
| 02 | [Sécurité réseau](skills/02-network-security) | Segmentation, scan, analyse de trafic |
| 03 | [Sécurité des applications web](skills/03-web-application-security) | Failles de classe OWASP |
| 04 | [Sécurité des API](skills/04-api-security) | REST, GraphQL, gRPC, auth, rate limiting |
| 05 | [Sécurité mobile](skills/05-mobile-security) | Évaluation d'applications Android et iOS |
| 06 | [Sécurité cloud](skills/06-cloud-security) | Posture et mauvaises configurations AWS, Azure, GCP |
| 07 | [Conteneurs & Kubernetes](skills/07-container-and-kubernetes-security) | Images, runtime, durcissement de cluster |
| 08 | [DevSecOps & CI/CD](skills/08-devsecops-and-cicd-security) | Intégrité de pipeline, secrets, gates |
| 09 | [Chaîne d'approvisionnement logicielle](skills/09-software-supply-chain-security) | Dépendances, SBOM, signature d'artefacts |
| 10 | [Revue de code sécurité](skills/10-secure-code-review) | Lire du code à la recherche de patterns vulnérables |
| 11 | [Gestion des identités et des accès](skills/11-identity-and-access-management) | OAuth2, OIDC, SAML, gestion de session |
| 12 | [Active Directory & Windows](skills/12-active-directory-and-windows-security) | Chemins d'attaque de domaine et durcissement |
| 13 | [Linux & Unix](skills/13-linux-and-unix-security) | Durcissement, frontières de privilèges, audit |
| 14 | [Cryptographie & PKI](skills/14-cryptography-and-pki) | Usage correct des primitives, certificats, gestion de clés |
| 15 | [Gestion des vulnérabilités](skills/15-vulnerability-management) | Scan, tri, priorisation, SLA |
| 16 | [Red team & émulation d'adversaire](skills/16-red-teaming-and-adversary-emulation) | Test offensif autorisé aligné ATT&CK |
| 17 | [Défense contre l'ingénierie sociale](skills/17-social-engineering-defence) | Analyse de phishing, sensibilisation, contrôles |
| 18 | [Detection engineering](skills/18-detection-engineering) | Écrire et tester des règles de détection |
| 19 | [SOC & SIEM](skills/19-security-operations-and-siem) | Pipelines de logs, tri d'alertes, astreinte |
| 20 | [Threat hunting](skills/20-threat-hunting) | Recherche par hypothèse dans la télémétrie |
| 21 | [Threat intelligence](skills/21-threat-intelligence) | Collecte, enrichissement, suivi d'acteurs |
| 22 | [Réponse à incident](skills/22-incident-response) | Confinement, éradication, remise en service |
| 23 | [Investigation numérique](skills/23-digital-forensics) | Artefacts disque, mémoire, cloud et mobile |
| 24 | [Analyse de malware & reverse](skills/24-malware-analysis-and-reverse-engineering) | Analyse statique et dynamique |
| 25 | [Sécurité IA & LLM](skills/25-ai-and-llm-security) | Injection de prompt, chaîne d'appro modèles, sûreté des agents |
| 26 | [GRC, risque & conformité](skills/26-grc-risk-and-compliance) | ISO 27001, SOC 2, NIS2, registres de risque |

## Utiliser une skill

**En tant qu'humain.** Ouvrez le `SKILL.md`, allez au cheatsheet, lancez les commandes. La procédure au-dessus explique pourquoi l'ordre compte.

**En tant que skill d'agent.** L'arborescence suit la convention `SKILL.md` reconnue par plusieurs assistants de code agentiques : un dossier par skill, un bloc frontmatter YAML avec `name` et `description`, les instructions dans le corps. Copiez les dossiers voulus dans le répertoire de skills de votre assistant :

```bash
cp -r skills/03-web-application-security/* ~/.config/agent-skills/
```

Le champ `description` du frontmatter sert au matching de l'assistant : il est donc rédigé comme une phrase de déclenchement, pas comme un titre.

## Format d'une skill

Tous les `SKILL.md` partagent le même squelette, documenté dans [docs/skill-format.md](docs/skill-format.md). Il est appliqué souplement : une section qui ne s'applique pas à une skill est supprimée plutôt que remplie de vide.

```
---
name, domain, description, difficulty, tags, tools
---
## Purpose
## When to use it
## Procedure      <- étapes ordonnées, la partie qu'un agent exécute
## Cheatsheet     <- commandes et options, la partie qu'un humain parcourt
## Reading the output
## Pitfalls
## References
```

## Périmètre et autorisation

Cette bibliothèque couvre le travail défensif et le test offensif autorisé : missions avec accord écrit, CTF, environnements de lab, votre propre infrastructure. Plusieurs domaines décrivent des techniques d'attaque, parce qu'on ne détecte ni ne corrige ce qu'on n'a jamais vu s'exécuter.

Ce qu'elle ne contient volontairement pas : des payloads d'exploitation prêts à l'emploi visant des logiciels non corrigés, du code de malware, des techniques dont la seule finalité est d'échapper aux défenseurs, ou quoi que ce soit dirigé contre des systèmes qui ne vous appartiennent pas. Les skills offensives sont écrites du point de vue du testeur et se terminent toujours par la remédiation correspondante.

Lancer tout ceci contre une infrastructure que vous n'êtes pas autorisé à tester est illégal dans la plupart des juridictions. La responsabilité vous revient, pas au repo.

## État d'avancement

Les domaines se remplissent par vagues, pas d'un bloc. Une skill entre quand elle a servi, pas quand un dossier réclame un fichier. Les `TODO` dans les README de domaine sont réels.

## Contribuer

Voir [CONTRIBUTING.md](CONTRIBUTING.md). En résumé : une skill par pull request, frontmatter valide, et pas de théorie sans une commande qui la démontre.

## Licence

MIT pour le contenu de ce dépôt. Les outils tiers cités conservent leurs propres licences.

---

<p align="center">
  <img src="assets/jihedailabs-logo.svg" alt="JihedAiLabs" width="80" height="80"><br>
  <sub>Conçu et maintenu sous <b>JihedAiLabs</b>.</sub>
</p>
