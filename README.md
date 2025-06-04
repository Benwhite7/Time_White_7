# 🌤️ Aplicación del Clima

Aplicacion interactiva para hacer consultas del clima, brinda datos como temperatura en diversas ciudades.

## 🚀 Características

- Búsqueda de ciudades por nombre (con sugerencias).
- Visualización del clima actual (temperatura, humedad, viento).
- Detección automática por geolocalización.
- Ciudades favoritas guardadas en `localStorage`.
- Fondos animados dinámicos según el clima (día/noche, lluvia, niebla, etc.).
- Compatible con dispositivos móviles.

## 🛠️ Tecnologías utilizadas

- [Vue 3](https://vuejs.org/)
- [Vite](https://vitejs.dev/)
- [OpenWeatherMap API](https://openweathermap.org/)
- CSS personalizado + animaciones

## 📦 Instalación

1. Clona este repositorio:
   git clone https://github.com/Benwhite7/Time_White_7.git →
   cd Time_White_7
2. Instala las dependencias:
   npm install
3. Agrega tu API Key en un archivo .env:
   VITE_API_KEY=tu_clave_de_openweathermap 
4. Inicia el servidor de desarrollo:
   npm run dev

## 🔍 Cómo usar
   Escribe el nombre de una ciudad y presiona Enter o haz clic en "Buscar".
   Puedes hacer clic en los íconos de corazón para guardar o eliminar favoritos.
   La interfaz se adapta según el clima y la hora del lugar buscado.   

## 📁 Estructura del proyecto
src/assets/              → JSON de ciudades
src/components/          → Componentes Vue como SearchSection, RainCanvas, etc.
src/App.vue              → Componente raíz
src/main.js              → Punto de entrada
src/styles.css           → Estilos globales
src/assets/styles/       → Estilos para componentes

## 📄 Licencia
Este proyecto está licenciado bajo la MIT License. 
Desarrollado por BenWhite7 ⚡
