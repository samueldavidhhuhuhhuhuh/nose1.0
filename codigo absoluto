import network, time, urequests
from machine import Pin,ADC
from utime import sleep
from dht import DHT11
from umqtt.simple import MQTTClient
import ujson

# Inicializa el sensor DHT11.
dht11 = DHT11(Pin(15))  # Cambia el número del pin al que está conectado tu sensor.

# Inicializa el sensor de CO.
sensor_co = ADC(Pin(36))

# Inicializa el sensor MH para la luminosidad.
sensor_luz = ADC(Pin(35))

# Inicializa el sensor MH-RD para la humedad.
sensor_humedad = ADC(Pin(32))

# Inicializa el sensor FC-28 para la humedad del suelo.
sensor_suelo = ADC(Pin(39))

# Configura la resolución del ADC para los sensores de CO, luz, humedad y humedad del suelo.
sensor_co.atten(ADC.ATTN_11DB)
sensor_luz.atten(ADC.ATTN_11DB)
sensor_humedad.atten(ADC.ATTN_11DB)
sensor_suelo.atten(ADC.ATTN_11DB)


# MQTT Server Parameters
MQTT_CLIENT_ID = "covertura_123"
MQTT_BROKER    = "broker.hivemq.com"
#MQTT_BROKER    = "192.168.129.84" 

MQTT_USER      = ""
MQTT_PASSWORD  = ""
MQTT_TOPIC     = "covertura/vegetal/nodo1"

def leer_sensor_co():
    valor = float(sensor_co.read_u16())
    ppm = (valor - 0) * (2000 - 20) / (65535 - 0) + 20
    return ppm



def conectaWifi (red, password):
      global miRed
      miRed = network.WLAN(network.STA_IF)     
      if not miRed.isconnected():              #Si no está conectado…
          miRed.active(True)                   #activa la interface
          miRed.connect(red, password)         #Intenta conectar con la red
          print('Conectando a la red', red +"…")
          timeout = time.time ()
          while not miRed.isconnected():           #Mientras no se conecte..
              if (time.ticks_diff (time.time (), timeout) > 10):
                  return False
      return True


if conectaWifi ("Hugo", "Hupe6493"):

    print ("Conexión exitosa!")
    print('Datos de la red (IP/netmask/gw/DNS):', miRed.ifconfig())

    print("Conectando a  MQTT server... ",MQTT_BROKER,"...", end="")
    client = MQTTClient(MQTT_CLIENT_ID, MQTT_BROKER, user=MQTT_USER, password=MQTT_PASSWORD)
    client.connect()

    print("Conectado!")
    
    nuevo_dato = ""


    while True:
        
        # Realiza una medición del sensor DHT11.
        dht11.measure()
        
        # Obtiene la temperatura y la humedad.
        temp = dht11.temperature()  # Temperatura en grados Celsius.
        hum = dht11.humidity()  # Humedad en porcentaje.

        print('Temperatura: {}°C, Humedad: {}%'.format(temp, hum))

        # Lee el sensor de CO y muestra la lectura en ppm.
        ppm = leer_sensor_co()
        print('CO: {:.2f} ppm'.format(ppm))

        # Realiza una lectura del sensor MH para la luminosidad y conviértela a porcentaje.
        lectura_luz = sensor_luz.read()
        luminosidad = (4095 - lectura_luz) / 4095 * 100
        print('Luminosidad: {:.2f}%'.format(luminosidad))

        # Realiza una lectura del sensor MH-RD para la humedad y conviértela a porcentaje.
        lectura_humedad = sensor_humedad.read()
        humedad_lluvia = (4095 - lectura_humedad) / 4095 * 100
        print('Humedad: {:.2f}%'.format(humedad_lluvia))

        # Realiza una lectura del sensor FC-28 para la humedad del suelo y conviértela a porcentaje.
        lectura_suelo = sensor_suelo.read()
        humedad_suelo = (4095 - lectura_suelo) / 4095 * 100
        print('Humedad del suelo: {:.2f}%'.format(humedad_suelo))
        
        
        tem = temp
        lum = luminosidad
        lluvia = humedad_lluvia
        
        hum_s = humedad_suelo
    
        monoxido = ppm
        
        
        
        
        
        
       
        
        print("Revisando Condiciones ...... ")
        message = ujson.dumps({
        "Humedad_aire": hum,
        "Temperatura": tem,
        "lluvia": lluvia,
        "ppm": ppm,
        "Luz": lum,
        "Humedad_suelo":hum_s
        })

        if message != nuevo_dato:
            
            print("Reportando a  MQTT topic {}: {}".format(MQTT_TOPIC, message))
            client.publish(MQTT_TOPIC, message)
            nuevo_dato = message

        else:
            print("No hay cambios")
        time.sleep(3)

    
 
else:
       print ("Imposible conectar")
       miRed.active (False)
