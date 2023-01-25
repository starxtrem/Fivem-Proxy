# Fivem-Proxy Version Française
## Prérequis:
Vps ou Vm autre que ton serveur fivem si non aucun intérêts (mais sa fonctionne quand même) et avoir un nom de domaine

Linux Ubuntu ou Debian

Installez [Nginx](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-prebuilt-debian-packages) !

Une fois la chose fait vous allew devoir effectuer quelle-que configuration


##### Partie a rajouter dans le cfg de votre serveur

    set sv_forceIndirectListing true
    set sv_listingHostOverride sous-domain.test.com
    set sv_listingIpOverride "sous-domain.test.com"
    set sv_proxyIPRanges "IPProxy/32"
    set sv_endpoints "serveurfivem:30120"
    
    set adhesive_cdnKey "chainedecaracterequetuveux"
    fileserver_remove ".*"
    fileserver_add ".*" "https://sous-domain.test.com/files"
    fileserver_list
    
