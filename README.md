# Récupération des données de téléinformation dans home assistant avec esphome

Fonctionne avec un compteur linky avec le tic en mode historique ou basic.

<b>Quelques liens :</b>
- Module téléinfo dans esphome : [Esphome-linky](https://esphome.io/components/sensor/teleinfo.html) 
- Esphome-flasher : [Esphome-flasher](https://github.com/esphome/esphome-flasher/releases)

### Composants Electronique :

Liste des composants :

- 1x ftdi (1er flash)
- 1x esp01s (esp8266)
- 1x module [PitInfo](https://www.tindie.com/products/hallard/pitinfo/)
- 1x alimentation 5v
- 1x convertisseur 5v=>3.3v
- 1x boite de dérivation 
- fil, bornier, wago, etc..

Shéma de cablage :

Il faut relier le txd du pitinfo au rxd de l'esp8266  

![links](https://github.com/NicoDupont/esp_linky/blob/master/doc/esppit.png?raw=true)

Réalisation :

![links](https://github.com/NicoDupont/esp_linky/blob/master/img/IMG_0590.jpg?raw=true)

### HomeAssistant :

#### Esphome :

1. créer un nouveau noeud dans esphome.  
2. le nommer 'esp_linky' ou comme vous le souhaitez... 
3. copier le code du fichier esp_linky.yaml  
4. Adapter le code pour votre wifi
5. Vérifier si vous etes en mode historique ou basic et adapter le code si besoin.
6. vérifier le code  
7. télécharger le binaire et flasher votre esp8266 avec esphome flasher ou alors via le navigateur
8. Ensuite tout se fera en OTA


#### Dashboard Energy :

Si vous voulez suivre les coûts, il faut créer 2 input_number avec comme unité : €/kWh

Configurer le dashboard energy avec les sensors :

    sensor.linky_hc_kwh => avec son coût associé
    sensor.linky_hp_kwh => avec son coût associé

![links](https://github.com/NicoDupont/esp_linky/blob/master/img/input_number_prix.png?raw=true)

![links](https://github.com/NicoDupont/esp_linky/blob/master/img/dashboard_energy_1.png?raw=true)

#### configuration.yaml :

Dans le fichier configuration.yaml :

Ajouter le contenu suivant si vous souhaitez en sus du dashboard énergy des données agrégées dans lovelace. Ici par jour et par mois :

    utility_meter:
    daily_energie_hc:
        source: sensor.linky_hc_kwh
        cycle: daily
        tariffs:
        - Heure creuse
    daily_energie_hp:
        source: sensor.linky_hp_kwh
        cycle: daily
        tariffs:
        - Heure pleine
    monthly_energie_hc:
        source: sensor.linky_hc_kwh
        cycle: monthly
        tariffs:
        - Heure creuse
    monthly_energie_hp:
        source: sensor.linky_hp_kwh
        cycle: monthly
        tariffs:
        - Heure pleine


#### Graphique :

yaml pour lovelace :  

    type: vertical-stack
    cards:
    - type: markdown
        content: <b><center>Compteur Linky</center></b>
    - type: custom:mini-graph-card
        hours_to_show: 24
        points_per_hour: 60
        hour24: true
        decimals: null
        show:
        fill: fade
        extrema: false
        labels: true
        entities:
        - entity: sensor.linky_puissance_apparente
            name: Puissance (24h)
            show_state: true
        color_thresholds:
        - value: 1000
            color: '#009FC1'
        - value: 3000
            color: '#D0AB00'
        - value: 8000
            color: '#C10042'
    - type: custom:mini-graph-card
        entities:
        - entity: sensor.linky_puissance_apparente
            color: '#69a6e0'
            name: Puissance Moyenne (24h)
        aggregate_func: avg
        hour24: true
        decimals: null
        points_per_hour: 1
        lower_bound: 0
        show:
        graph: bar
        state: false
        extrema: true
    - type: custom:apexcharts-card
        graph_span: 15d
        span:
        end: day
        stacked: true
        header:
        show: true
        title: Consommation journalière (15j)
        show_states: true
        colorize_states: true
        series:
        - entity: sensor.daily_energie_hp_heure_pleine
            type: column
            color: '#1781d8'
            name: Heures pleines
            group_by:
            duration: 1d
            func: max
        - entity: sensor.daily_energie_hc_heure_creuse
            type: column
            color: '#94d5ef'
            name: Heures creuses
            group_by:
            duration: 1d
            func: max
    - type: custom:apexcharts-card
        graph_span: 13month
        span:
        end: month
        stacked: true
        header:
        show: true
        title: Consommation Mensuelle (13 Mois)
        show_states: true
        colorize_states: true
        series:
        - entity: sensor.monthly_energie_hp_heure_pleine
            type: column
            color: '#1781d8'
            name: Heures pleines
            group_by:
            duration: 1month
            func: max
        - entity: sensor.monthly_energie_hc_heure_creuse
            type: column
            color: '#94d5ef'
            name: Heures creuses
            group_by:
            duration: 1month
            func: max


![links](https://github.com/NicoDupont/esp_linky/blob/master/img/dashboard_ha_1.png?raw=true)

![links](https://github.com/NicoDupont/esp_linky/blob/master/img/dashboard_ha_2.png?raw=true)


### Sources, inspiration puis adaptation :

[https://github.com/xmow49/teleinfo-esphome-homeassistant/](https://github.com/xmow49/teleinfo-esphome-homeassistant/)  
[https://github.com/schmurtzm/Teleinfo-TIC-with-ESPhome](https://github.com/schmurtzm/Teleinfo-TIC-with-ESPhome)  
[https://342apps.net/configurer-home-assistant-pour-le-teleinfokit/](https://342apps.net/configurer-home-assistant-pour-le-teleinfokit/)  
[https://www.tindie.com/products/hallard/wemos-teleinfo/](https://www.tindie.com/products/hallard/wemos-teleinfo/)  
