![Version](https://img.shields.io/github/v/release/lendy007/homeassistant-Seatconnect?include_prereleases)
![PyPi](https://img.shields.io/pypi/v/Seatconnect?label=latest%20pypi)
![Downloads](https://img.shields.io/github/downloads/lendy007/homeassistant-Seatconnect/total)

# Seat Connect - An home assistant plugin to add integration with your car

# v1.0.0

## This is fork of [robinostlund/homeassistant-volkswagencarnet] and [lendy007/homeassistant-skodaconnect] (https://github.com/robinostlund/homeassistant-volkswagencarnet) where I am trying to modify the code to support Seat Connect.

### What is NOT working / under development
- for auxiliary heating/ventilation - after enabling you need to wait about 2 minutes to get true status if it is really enabled or not
- trigger status refresh from car - for status changes where car doesn't report it automatically to server (for example car was unlocked on the garden and you just lock it) it still shows old status until car will upload new status or status is refreshed from Seat Connect App
- when vehicleMoving=yes device_tracker GPS stays on old values until parked
- Hass.io compatibility issues, should be fixed with 1.0.26 release (?)
- odometer
- fuel level, range, adblue level
- lock status, window status
- last trip info
- position - gps coordinates, vehicleMoving, parkingTime 
- auxiliary heating/ventilation control
- electric engine related information thanks to @Farfar
- electric climatisation and window_heater information thanks to @Farfar
- start/stop electric climatisation and window_heater thanks to @Farfar
- lock/unlock car thanks to @tanelvakker

### Install
Clone or copy the repository and copy the folder 'homeassistant-seatconnect/custom_component/Seatconnect' into '<config dir>/custom_components'
    
## Configure

Add a Seatconnect configuration block to your `<config dir>/configuration.yaml`:
```yaml
Seatconnect:
    username: <username for Seat connect>
    password: <password for Seat connect>
    spin: <S-PIN for Seat connect>
    combustion_engine_heating_duration: <allowed values 10,20,30,40,50,60 (minutes)>
    combustion_engine_climatisation_duration: <allowed values 10,20,30,40,50,60 (minutes)>
    scandinavian_miles: false
    imperial_units: false
    scan_interval:
        minutes: 5
    name:
        wvw1234567812356: 'Kodiaq'
    resources:
        - combustion_engine_heating         
        - combustion_climatisation
        - distance
        - position
        - service_inspection
        - oil_inspection
        - door_locked
        - trunk_locked
        - request_in_progress
        - fuel_level        
        - windows_closed        
        - adblue_level
        - climatisation_target_temperature
        - last_connected
        - combustion_range
        - trip_last_average_speed
        - trip_last_average_fuel_consumption
        - trip_last_duration
        - trip_last_length
        - parking_light
        - door_closed_left_front        
        - door_closed_left_back
        - door_closed_right_front
        - door_closed_right_back
        - trunk_closed
        - window_closed_left_front
        - window_closed_left_back
        - window_closed_right_front
        - window_closed_right_back
        - sunroof_closed
        - service_inspection_km
        - oil_inspection_km
        - outside_temperature
        - electric_climatisation
        - window_heater
        - charging
        - battery_level
        - charging_time_left
        - electric_range
        - combined_range
        - charge_max_ampere
        - climatisation_target_temperature
        - external_power
        - climatisation_without_external_power
        - charging_cable_connected
        - charging_cable_locked
        - trip_last_average_electric_consumption
        - hood_closed
        - combustion_engine_heatingventilation_status
        - vehicleMoving
        - parkingTime
        - energy_flow
        - requests_remaining
```

* **resources:** if not specified, it will create all supported entities

* **spin:** (optional) required for supporting combustion engine heating start/stop.

* **scan_interval:** (optional) specify in minutes how often to fetch status data from carnet. (default 5 min, minimum 1 min)

* **scandinavian_miles:** (optional) specify true if you want to change from km to mi on sensors

* **imperial_units:** (optional) specify true if you want imperial units for all sensors. Overrides scandinavian_miles

* **name:** (optional) map the vehicle identification number (VIN) to a friendly name of your car. This name is then used for naming all entities. See the configuration example. (by default, the VIN is used). VIN need to be entered lower case

## Automations

In this example we are sending notifications to an ios device

Save these automations in your automations file `<config dir>/automations.yaml`

### Get notification when your car is on a new place and show a map with start position and end position
```yaml
- id: notify_Seat_position_change
  description: Notify when position has been changed
  alias: Seat position changed notification
  trigger:
    - platform: state
      entity_id: device_tracker.kodiaq
  action:
    - service: notify.ios_my_ios_device
      data_template:
        title: "Kodiaq Position Changed"
        message: |
          🚗 Seat Car is now on a new place.
        data:
          url: /lovelace/car
          apns_headers:
            'apns-collapse-id': 'car_position_state_{{ trigger.entity_id.split(".")[1] }}'
          push:
            category: map
            thread-id: "HA Car Status"
          action_data:
            latitude: "{{trigger.from_state.attributes.latitude}}"
            longitude: "{{trigger.from_state.attributes.longitude}}"
            second_latitude: "{{trigger.to_state.attributes.latitude}}"
            second_longitude: "{{trigger.to_state.attributes.longitude}}"
            shows_traffic: true
```

### Announce when your car is unlocked but no one is home
```yaml
- id: 'notify_Seat_car_is_unlocked'
  alias: Seat is at home and unlocked
  trigger:
    - entity_id: binary_sensor.vw_carid_external_power
      platform: state
      to: 'on'
      for: 00:10:00
  condition:
    - condition: state
      entity_id: lock.kodiaq_door_locked
      state: unlocked
    - condition: state
      entity_id: device_tracker.kodiaq
      state: home
    - condition: time
      after: '07:00:00'
      before: '21:00:00'
  action:
    # Notification via push message to smartphone
    - service: notify.device
      data:
        message: "The car is unlocked!"
        target:
          - device/my_device
    # Notification via smart speaker (kitchen)
    - service: media_player.volume_set
      data:
        entity_id: media_player.kitchen
        volume_level: '0.6'
    - service: tts.google_translate_say
      data:
        entity_id: media_player.kitchen
        message: "My Lord, the car is unlocked. Please attend this this issue at your earliest inconvenience!"
```

## Enable debug logging
```yaml
logger:
    default: info
    logs:        
        custom_components.Seatconnect: debug
        custom_components.Seatconnect.climate: debug
        custom_components.Seatconnect.lock: debug
        custom_components.Seatconnect.device_tracker: debug
        custom_components.Seatconnect.switch: debug
        custom_components.Seatconnect.binary_sensor: debug
        custom_components.Seatconnect.sensor: debug
 ```

