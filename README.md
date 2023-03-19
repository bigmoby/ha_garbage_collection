# ha_garbage_collection

Project from: https://hassiohelp.eu/2019/03/17/raccolta-differenziata/

- Add `python_script:` into configuration.yaml
- add into configuration.yaml
  ```
  customize:
 
    package.node_anchors:
      customize: &customize
        Package:: Raccolta Differenziata
        Creato da:: Mattia (xxKira) e Fabio Bigmoby Mauro
        Creato per:: HassioHelp
        
    input_text.umido_hp:
      icon: mdi:food-apple
      hidden: false
    input_text.carta_hp:
      icon: mdi:file
      hidden: false
    input_text.vetro_hp:
      icon: mdi:glass-wine
      hidden: false
    input_text.plastica_hp:
      icon: mdi:bottle-wine 
      hidden: false
    input_text.verde_hp:
      icon: mdi:tree
      hidden: false
    input_text.secco_hp:
      icon: mdi:delete-empty
      hidden: false
    input_text.metallo_hp:
      icon: mdi:delete-variant
      hidden: false     
    input_text.pannolini_hp:
      icon: mdi:human-baby-changing-table
      hidden: false         
  
  customize_glob:
    "input_text.*_hp":
      <<: *customize
    "automation.*_hp":
      <<: *customize
    "input_datetime.*_hp":
      <<: *customize
    "input_boolean.*_hp":
      <<: *customize
    "script.*_hp":
      <<: *customize

    
input_datetime:
  orario_avviso_differenziata_hp:
    name: Orario avviso raccolta differenziata
    has_date: false
    has_time: true
    initial: "08:00"

#serve per far comparire/scomparire il menù scelta giorni in lovelace
input_boolean:
  differenziata_settings_hp:
  giorni_settings_hp:
  giorno_notifica_hp:
 
input_text:
  vuoto:
    name: vuoto
    initial: ' '
  secco_hp:
    name: secco
  metallo_hp:
    name: imballaggi metallici    
  umido_hp:
    name: umido
  carta_hp:
    name: carta
  plastica_hp:
    name: plastica
  vetro_hp:
    name: vetro
  verde_hp:
    name: verde
  pannolini_hp:
    name: pannolini    
  servizio_notifica_hp:
    name: notify.
    initial: 'telegram'
  titolo_notifica_hp:
    name: titolo notifica
    initial: '*Differenziata*'
  messaggio_notifica_hp:
    name: messaggio notifica
    initial: 'Domani ritirano:'
  entity_google_tts_hp:
    initial: 'google_home'
    name: 'tts.google_say'
  entity_alexa_tts_hp:
    initial: 'alexa, echo'
    name: 'alexa_tts'
  entity_script_tts_hp:
    initial: 'speech'
    name: 'script.'    

automation:
- alias: Sincronizzo Raccolta hp
  initial_state: 'on'
  trigger:
  - platform: homeassistant
    event: start
  - platform: state
    entity_id: input_text.verde_hp, input_text.umido_hp, input_text.vetro_hp, input_text.carta_hp,
      input_text.plastica_hp, input_text.secco_hp, input_text.metallo_hp, input_text.pannolini_hp
  - platform: template
    value_template: '{{ states("sensor.time") == "01:00" }}'
  action:
  - service: python_script.raccolta
    data:
      argomento: sincronizza
      set_settimana: 'True'
  id: e007486d01ca4236bf1837cf04c2bae7
- alias: Notifica Raccolta hp
  initial_state: 'on'
  trigger:
  - platform: template
    value_template: '{{ states("sensor.time") == (states.input_datetime.orario_avviso_differenziata_hp.attributes.timestamp
      | int | timestamp_custom("%H:%M", False)) }}'
  action:
  - service: script.notifica_raccolta_hp
  id: 58ebb2e666954ed6a046698140a513ad


- id: '1661111772253'
  alias: Sincronizza rifiuti
  description: ''
  trigger:
  - platform: state
    entity_id:
    - input_boolean.giorno_notifica_hp
  condition: []
  action:
  - service: script.sincronizza_raccolta_hp
    data: {}
  mode: single

script:
sincronizza_raccolta_hp:
  sequence:
  - service: python_script.raccolta
    data:
      argomento: sincronizza
      set_settimana: 'True'
notifica_raccolta_hp:
  sequence:
  - service: python_script.raccolta
    data_template:
      argomento: notifica
      set_settimana: 'True'
      servizio_notifica: '{{ states(''input_text.servizio_notifica_hp'') }}'
      titolo_notifica: '{{ states(''input_text.titolo_notifica_hp'') }}'
      messaggio_notifica: '{{ states(''input_text.messaggio_notifica_hp'') }}'
      entity_google_tts: '{{ states(''input_text.entity_google_tts_hp'') }}'
      entity_alexa_tts: '{{states(''input_text.entity_alexa_tts_hp'') }}'
      entity_script_tts: '{{ states(''input_text.entity_script_tts_hp'') }}'    
  ```

