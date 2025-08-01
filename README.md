# DryFilaPobre
Secadora de filamentos caseira

## Instalação
Instale o Python e o ESPHome
https://esphome.io/guides/installing_esphome.html

Após a instalação faça o setup inicial, plugue o esp32 no computador segurando o botão BOOT, após plugar pode soltar o botão

Grave o arquivo inicial com o comando

```esphome run first_program.yaml```

Acesse o endereço IP informado após a instalação para confirmar o funcionamento


## Gravando o arquivo da estufa

Após é possivel gravar o arquivo da estufa via WIFI, podemos desconectar o cabo USB.
Edite o YAML inicial colocando o seguinte código:

```yaml
captive_portal:

# Enable web interface
web_server:
  port: 80
# Define the onboard LED output
output:
  - platform: gpio
    pin: 8  # GPIO2 is common for onboard LED on ESP32/ESP8266
    id: onboard_led
    inverted: true  # Most onboard LEDs are inverted (LOW = ON)
  
  - platform: slow_pwm
    pin: GPIO1
    id: heating_element
    period: 15s
  # Dummy cool output (can be same as heat or unused pin)
  - platform: template
    id: dummy_cool
    type: float
    write_action:
      - logger.log: "Cool output called"

climate:
  - platform: pid
    name: "Controlador PID"
    id: pid_estufa
    sensor: dht_temp
    default_target_temperature: 40°C
    heat_output: heating_element
    cool_output: dummy_cool
    visual:
      min_temperature: 10
      max_temperature: 80
      temperature_step:
        target_temperature: 1
        current_temperature: 0.1
    control_parameters:
      kp: 10
      ki: 1
      kd: 2
      output_averaging_samples: 5      
      derivative_averaging_samples: 5
    deadband_parameters:
      threshold_high: 2.0°C   
      threshold_low: -1.0°C

button:
  - platform: template
    name: "PID Autotune"
    on_press:
      - climate.pid.autotune: pid_estufa


sensor:
  - platform: dht
    model: DHT11
    pin: GPIO4
    temperature:
      name: "Temperatura"
      id: dht_temp
      filters:
        - sliding_window_moving_average:
            window_size: 3
            send_every: 3
    humidity:
      name: "Umidade"
      id: dht_hum
    update_interval: 10s
  
  - platform: pid
    name: "PID PWM"
    type: RESULT


# Create a switch to control it
switch:
  - platform: output
    name: "Onboard LED"
    output: onboard_led
    id: led_switch
```


Após a edição basta rodar

```esphome run first_program.yaml```

e escolher o dispositivo na rede.
