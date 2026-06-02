# Proyecto: Sistema de Filtrado y Nivel de Agua

## 1. Descripción General
Este proyecto trata de la instalación de un sistema de filtrado y correción de sarro (dos filtros) en la entrada del tanque de agua de la casa.  Luego le sigue un medidor de presión (para saber que presión esta entrando enla casa), luego un caudalimetro para finalmnete llegar a la entrada del tanque de agua donde hay un flotante nuevo. Este ssimeta ademas consta de un medidor ultrasonico para tener el nivel del tanque y luego un medidor de temperatrura de agua, otro de temperatura ambiente y ademas hay un snsor de liquidos que marca cuando rebalsa el tanque de agua porque el florante no corto. 

## 2. Hardware y Sensores
### 2.1 Hardware Fisico

Todo el hardware fisico se realizo con elemkentos de termofusión en cañerías de 3/4" de pulgada

| Componente | Modelo / Especificación | Pin (ESP32) |
| :--- | :--- | :--- |
| Llave de paso | ||
| Unión doble |||
| Filtro de Agua | 5 micrones ||
| Filtro de Sarro |||
| Codo 90°
| Union T
| Medidor de presión 
| Unión Roscada para caudalim
| Codo 90 °
| Llave de paso
| Codo 90°| Acceso al tanque de agua


### 2.2 Componentes Electrónicos
| Componente | Modelo / Especificación | Pin (ESP32) |
| :--- | :--- | :--- |
| Caudalimetro ||| FS401 - | GPIO 33 |
| Sensor de Nivel | Ultrasonido (ej: JSN-SR04T) | DXX |
| Sensor Temperatura Agua | DSB180 | GPIO4 |
| Filtrado | Sedimentos / Ion-Exchange | GPIO |
| Sensor Temperatura Ambiente
| Nivel de líquidos| | GPIO 6

## 3. Esquema de Conexiones
*   (Describe aquí brevemente cómo alimentas los sensores o si usas algún divisor de voltaje).

## 4. Configuración (ESPHome / YAML)
*   *Nota: Aquí puedes pegar los bloques de código más importantes de tu archivo yaml para no tener que buscarlo en el servidor de Home Assistant.*

## 5. Bitácora de Mantenimiento
* **[2026-06-02]**: Creación de la documentación inicial del sistema.
* **[Agregar fecha]**: (Ejemplo: Cambio de filtros de sedimentos).

## 6. Problemas conocidos / Pendientes
* [ ] (Ejemplo: Ajustar la calibración del ultrasonido porque el tanque tiene mucha humedad).