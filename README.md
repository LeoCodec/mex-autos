---

## 🏗️ **Arquitectura del Microservicio**

El proyecto **MexAutos** se basa en una arquitectura modular orientada a microservicios, donde el cliente (interfaz web desarrollada con **Vue 3 y Vite**) se comunica con una fuente de datos en formato **JSON**, simulando el comportamiento de una **API RESTful**.

En un escenario futuro, este modelo permite incorporar un **servidor Flask** (Python) para gestionar solicitudes HTTP (`GET`) y acceder a una base de datos **SQLite**, sin necesidad de modificar el contrato de comunicación actual.

<div align="center">

### 🧩 Diagrama de Arquitectura Completa
![Arquitectura del Microservicio – MexAutos](./src/assets/arquitectura_mexautos.png)

</div>

### 🔍 Descripción de la arquitectura:

| Componente | Descripción |
|-------------|-------------|
| 🧠 **Cliente (Frontend)** | Aplicación Vue (Vite + Netlify) que permite filtrar, mostrar y renderizar los autos en tiempo real. |
| ⚙️ **API Flask (futuro)** | Serviría como capa intermedia que expone endpoints `GET /api/autos` para obtener o filtrar vehículos desde un backend real. |
| 📂 **Fuente de datos (`data/autos.json`)** | Actúa como base de datos local; almacena más de 120 autos con sus atributos (marca, modelo, año, precio, color). |
| 💾 **SQLite (futuro)** | Permitiría persistencia estructurada sin modificar el contrato de la API, asegurando escalabilidad futura. |

El sistema sigue un **modelo cliente–servidor simplificado**, donde el flujo principal es:
1. El cliente envía una solicitud HTTP GET.  
2. La API o el frontend lee el archivo `cars.json`.  
3. Los componentes Vue (`FilterBar` y `CarCard`) procesan los filtros en memoria.  
4. Los resultados se renderizan dinámicamente en pantalla.  

> 🧩 Este modelo representa un enfoque educativo del concepto de **microservicios**, aplicable a sistemas reales donde la comunicación entre componentes se realiza mediante APIs REST y fuentes de datos independientes.

---

## 🌐 **Sitio web del proyecto**

El proyecto completo está desplegado en **Netlify**, empleando un entorno *serverless* para demostrar su funcionamiento real en la nube:  

🔗 **[MexAutos – Catálogo de Autos Americanos (2025)](https://mexautos.netlify.app)**  

*(El sitio se actualiza automáticamente con cada push al repositorio GitHub.)*

---
