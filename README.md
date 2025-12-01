# Evidencias trabajo MGADS usando YOLOv8

En este markdown se recopilan las evidencias del trabajo con YOLO realizado en el curso de Ciencia de Datos de MGADS.

## Integrantes

- Juan Eduardo Quintero Palacio, <jquintero743@unab.edu.co>
- Daniel Hernando León Pimiento, <dleon148@unab.edu.co>

## Objetivo

Implementar un sistema de detección automática de placas vehiculares usando YOLOv8 con técnicas de transfer learning y de reconocimiento de caracteres usando EasyOCR.

## Tecnologías utilizadas

- Python 3.11 (3.13 no permitía instalar algunas librerías)
- YOLOv8 (ultralytics)
- OpenCV
- FastAPI
- Uvicorn
- EasyOCR
- Torch
- Visual Studio Code
- Git y GitHub

### Actividades realizadas

#### Creación y configuración de instancia EC2

Primero creamos la instancia EC2 en AWS.

![Creación de instancia EC2](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/creacion_ec2.png)

Habilitamos el puerto 8080 en el Security Group para poder acceder a la API FastAPI que se implementará más adelante.

![Habilitación de puerto 8080](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/habilitacion_puerto_8080.png)

Verificamos la IP, el estado y el DNS de la instancia.

![Verificación de instancia EC2](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/verificacion_ec2.png)

#### Conexión a la instancia EC2 vía SSH

Por costumbre de Juan Quintero, se decidió usar un .pem para la conexión SSH. Este PEM se generó al momento de crear la instancia EC2 y se llama `mgads-yolov8.pem`.

Tuvimos que cambiar los permisos del archivo `.pem` para que solo el usuario tenga permisos de lectura.

![Intento fallido por permisos](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/primer_intento_fallido.png)

```bash
chmod 400 mgads-yolov8.pem
```

![Conexión exitosa vía SSH](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/conexion_exitosa_ssh.png)

#### Instalación de dependencias en la instancia EC2

Escribiremos los comandos usados para instalar las dependencias necesarias para el proyecto, ya que varias capturas de pantalla son muy largas y son solo comandos en terminal.

```bash
# En la instancia EC2
sudo apt update
sudo apt install -y libgl1 libglib2.0-0
sudo apt install -y python3-pip python3-venv
mkdir proyecto
cd proyecto
python3 -m venv .venv
source .venv/bin/activate
pip install --upgrade pip
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
pip install ultralytics fastapi uvicorn easyocr opencv-python-headless pillow numpy python-multipart
yolo check
# Creating new Ultralytics Settings v0.0.6 file ✅ 
# View Ultralytics Settings with 'yolo settings' or at '/tmp/Ultralytics/settings.json'
# Update Settings with 'yolo settings key=value', i.e. 'yolo settings runs_dir=path/to/dir'. For help see https://docs.ultralytics.com/quickstart/#ultralytics-settings.
# Ultralytics 8.3.233 🚀 Python-3.12.3 torch-2.9.1+cpu CPU (Intel Xeon Platinum 8259CL CPU @ 2.50GHz)
# Setup complete ✅ (2 CPUs, 1.9 GB RAM, 4.6/10.6 GB disk)
```

#### Subir archivos a la instancia EC2

Usamos `scp` para subir los archivos del proyecto a la instancia EC2.

```bash
# Maquina local
scp -i ~/Downloads/mgads-yolov8.pem modelo/best.pt snippet/app.py ubuntu@35.171.244.166:/home/ubuntu/proyecto
```

Verificamos que los archivos estén en la instancia EC2.

```bash
# En la instancia EC2
ls -la
vim app.py
```

![Verificación de archivos en EC2](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/›.png)

#### Ejecución de la API FastAPI en la instancia EC2

Ejecutamos la API usando FastAPI.

```bash
# En la instancia EC2
python app.py
```

![Ejecución de API FastAPI](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/ejecucion_api_fastapi.png)

#### Pruebas de la API FastAPI

Entramos al endpoint de documentación: <http://35.171.244.166:8080/docs>

![Documentación FastAPI](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/documentacion_fastapi.png)

Probamos la API subiendo una imagen de prueba.

![Imagen de prueba](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/carroprueba.png)

![Prueba de API FastAPI](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/prueba_fastapi.png)

![Respuesta de la API FastAPI](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/respuesta_fastapi.png)

#### Ejecutar aplicación móvil localmente

Verificamos la versión de Node.js y npm instalada.

```bash
node -v
# v25.1.0
npm -v
# 11.6.2
```

Instalamos Expo CLI y EAS CLI.

```bash
npm install expo-cli eas-cli --verbose
```

Verificamos las versiones instaladas.

```bash
npx expo --version
# 0.10.17
npx eas --version
# eas-cli/16.28.0 darwin-arm64 node-v25.1.0
```

Instalamos el resto de dependencias del proyecto móvil.

```bash
npx create-expo-app DetectorPlacas --template expo-template-blank-typescript
cd DetectorPlacas
npx expo install expo-camera expo-speech expo-image-manipulator
npx expo install expo-router expo-splash-screen expo-build-properties
npm install axios --legacy-peer-deps
npx expo install react-dom react-native-web
npx expo install expo-splash-screen
npx expo install expo-constants expo-linking react-native-safe-area-context react-native-screens

rm app.json
cp ../snippet/app.json .
rm index.tsx
cp ../snippet/index.tsx index.tsx
```

Tuvimos que realizar unos ajustes en el código para que la aplicación móvil funcione correctamente ya que las versiones de React del tutorial y la nuestra eran diferentes.

Ejecutamos `npx expo start --web` y abrimos `http://localhost:8081/`

![Aplicación móvil funcionando](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/aplicacion_movil_funcionando.png)

![Respuesta de la API en la aplicación móvil](https://github.com/juedquipa/mgads-yolov8/imagenes/evidencias/respuesta_api_aplicacion_movil.png)

### Conclusiones

El proyecto fue exitoso ya que se logró implementar un sistema de detección automática de placas vehiculares usando YOLOv8 y EasyOCR, desplegado en una instancia EC2 y consumido por una aplicación móvil desarrollada con Expo. Se aprendieron varias tecnologías y herramientas nuevas, y se enfrentaron desafíos relacionados con la configuración del entorno y la compatibilidad de versiones, los cuales fueron superados con éxito.
