# Code review ciblée (sécurité & perf)

Analyse le code référencé et propose :
- risques sécurité (XSS, injections, secrets) & mitigations
- points de performance (allocations, I/O, N+1, complexité)
- suggestions concrètes et diff minimal par fichier

Contexte à examiner : #BasketService.cs #OrderController #Utilities

Format de sortie attendu :
- Tableau récap (Fichier | Problème | Gravité | Suggestion)
- Puis un plan d’implémentation en étapes
