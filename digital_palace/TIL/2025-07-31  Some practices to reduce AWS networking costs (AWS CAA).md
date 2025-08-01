(prix par GB)
- Trafic entrant sur une EC2 = gratuit
- Trafic entre 2 EC2 d'une même AZ = gratuit SI utilise IP privée
- Traffic entre 2 EC2 appartenant à 2 diff AZ de même région = $0.02 si utilise IP pub/elastique
- Traffic entre 2 EC2 appartenant à 2 diff AZ de même région = $0.01 si utilise IP privée (2x moins !!)
- Traffic entre 2 EC2 de différentes régions = $0.02

**Bonnes pratiques**
- Utiliser les adresses IP privées le + possible pour réduire les coûts **et** plus rapide
- Si HA pas primordiale -> rester dans la même AZ le + possible pour réduire les coûts
