
{% assign gatewayConfiguration = '
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/conn1.png,  
        title: Open a Browser to the administration web of Cell 1024 using the URL: https://[IP_DEVICE] and go to the "<b>Cloud</b>" tab.
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/conn2.png,
        title: Activate the Cloud control and configure all the parameters to connect the device to the specific ThingsBoard platform via MQTT.
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/conn3.png,
        title: Click "<b>Save configuration</b>" button.
'
%}  

{% include images-gallery.liquid showListImageTitles="true" imageCollection=gatewayConfiguration %}

| Configuration parameters | Description |
|-|-|
|**Cloud Platform**| Select ThingsBoard. |
|**MQTT broker URL**| URL to de Broker of the server we want to integrate to. |
|**MQTT broker port**| Number of the port used by the server. |
|**TLS**| Select true if the server use the Transport Layer Security protocol. |
|**Connection Type**| Select 'Access Token' option. We will use an Access token previously created in ThingsBoard. |
|**Access Token**| Indicate the Access token previously copied in ThingsBoard. |

{% capture provisioningIsComing %}
**Note**

Currently, the IoT EXXN Gateways use the 'Access Token' integration method.  
EXXN team is working on a Pre-Provisioning integration method that will eliminate the need to copy this Access token on the device.  

{% endcapture %}
{% include templates/info-banner.md content=provisioningIsComing %}

To verify that the device is connected correctly to ThingsBoard, go to the **Device groups** menu -> **All** devices, select your device.  
In the **device details** select **client attributes** tab and check if the client attributes have been communicated to the device.  

{% assign checkConnection = '
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/exxn-client-attributes-device-1.png,
        title: If you did everything is correct, we will see client attributes like the <i>serial_number</i>, <i>last_reboot</i>, <i>device_model</i>, etc.
'
%}

{% include images-gallery.liquid showListImageTitles="true" imageCollection=checkConnection %}

The EXXN IoT Gateway will connect to ThingsBoard using the MQTT API.  
We have previously covered how to configure the device to connect to ThingsBoard.  
Now, we will show the steps to configure the device in ThingsBoard in order to monitor data and manage the device.  

In order to configure the datalogger options of the EXXN IoT Gateway, we should create a new JSON "**Shared Attribute**" for the Device with the key "**config**".  

We will use the following JSON:

```json
{
  "control_config": {
    "debug_level": 10
  },
  "datalogger_config": {
    "csv_path": "/opt/celling/datalogger/data/",
    "redis_list_max_len": 16384,
    "sensor_list": [
      {
        "id": "system_monitor",
        "description": "CPU usage (%), disk free (MB), memory available (MB)",
        "model": "sys_mon",
        "storage_device": "/dev/mmcblk0p2",
        "measures": [
          "disk_free",
          "mem_available",
          "cpu_usage"
        ],
        "enabled": true,
        "period": 60,
        "csv": true,
        "mqtt": true
      },
      {
        "id": "temp_sc0",
        "description": "Temperature of the small cell #0",
        "model": "ds18b20",
        "name": "sc0",
        "address": "28-01212e9c95be",
        "measures": [
          "temperature"
        ],
        "enabled": true,
        "period": 15,
        "csv": true,
        "mqtt": true
      },
      {
        "id": "temp_sc1",
        "description": "Temperature of the small cell #1",
        "model": "ds18b20",
        "name": "sc1",
        "address": "28-01212e96afff",
        "measures": [
          "temperature"
        ],
        "enabled": false,
        "period": 15,
        "csv": true,
        "mqtt": true
      },
      {
        "id": "fan_rpm",
        "description": "Fan tachometer monitor",
        "model": "tachometer",
        "name": "fan",
        "gpio": 22,
        "sampling_window": 5000,
        "measures": [
          "rpm"
        ],
        "enabled": true,
        "csv": true,
        "mqtt": true
      },
      {
        "id": "temp_hum",
        "description": "Temperature (celsius degrees), humidity (%)",
        "model": "cwt_th01s",
        "name": "th",
        "config_params": {
          "port": "/dev/ttyS1",
          "baudrate": 9600,
          "read_timeout": 0.5,
          "slave_id": 2
        },
        "measures": [
          "temperature",
          "humidity"
        ],
        "enabled": true,
        "period": 15,
        "csv": true,
        "mqtt": true
      },
      {
        "id": "gpio_monitor",
        "description": "GPIOs configuration and monitoring",
        "model": "gpio_mon",
        "name": "gpio",
        "measures": [
          "state"
        ],
        "mapping": {
          "LOCK": {
            "pin": 1,
            "gpio": 4,
            "direction": "out",
            "boot_value": 0
          },
          "LED": {
            "pin": 3,
            "gpio": 2,
            "direction": "out",
            "boot_value": 0
          },
          "FAN": {
            "pin": 5,
            "gpio": 34,
            "direction": "out",
            "boot_value": 0
          },
          "BUZZER": {
            "pin": 1,
            "gpio": 43,
            "direction": "out",
            "boot_value": 0
          }
        },
        "enabled": true,
        "period": 15,
        "csv": true,
        "mqtt": true
      },
      {
        "id": "energy_meter",
        "description": "Voltage (V), Current (A), Active Power (KWh), Power Factor",
        "model": "ddm18sd",
        "config_params": {
          "port": "/dev/ttyS1",
          "baudrate": 9600,
          "read_timeout": 1,
          "slave_id": 0
        },
        "measures": [
          "voltage",
          "current",
          "active_power",
          "power_factor"
        ],
        "enabled": true,
        "period": 15,
        "csv": true,
        "mqtt": true
      }
    ]
  }
}
```
{: .copy-code}

All the information to configure the device correctly through this JSON File can be found in the EXXN IoT Gateway Manual.

{% assign configureTheGateway = "
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/exxn-shared-attributes-device-1.png,
        title: Go to device'" | append: 's <b>attributes</b> tab in the device details. Add a new "<b>Shared attribute</b>" with the key "<b>config</b>" of type <b>JSON</b>.
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/ennx-config-json.png,
        title: Expand the content of the attribute to full screen for ease of writing it. Paste the contents of the device configuration file into the attribute value.
    ===
        image: /images/devices-library/ready-to-go-devices/iot-gateway-cell-1024/exxn-shared-attributes-device-2.png,
        title: Click "<b>Add</b>" attribute.
'
%}

{% include images-gallery.liquid showListImageTitles="true" imageCollection=configureTheGateway %}

