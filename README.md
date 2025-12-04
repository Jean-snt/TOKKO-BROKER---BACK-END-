🏡 Tokko Broker — Back-End
Plataforma de Gestión Inmobiliaria (Arquitectura Hexagonal)

Este repositorio contiene la implementación del Back-End de Tokko Broker, una plataforma de gestión inmobiliaria desarrollada utilizando:

Python (Django)

PostgreSQL + PostGIS

Redis (procesamiento asíncrono)

Arquitectura Hexagonal (Ports & Adapters)

Multitenencia por Sucursal

El objetivo principal es ofrecer una API RESTful robusta, segura y escalable, con estricto aislamiento de datos entre sucursales.

🏗️ 1. Arquitectura: Hexagonal (Ports & Adapters)

La Arquitectura Hexagonal garantiza que la lógica de negocio (Dominio) permanezca aislada de frameworks, infraestructura y APIs externas.

📦 Estructura Principal del Proyecto
Capa	Carpeta	Descripción
Dominio	src/core/domain	Reglas de negocio puras. Contiene Entidades (Propiedad, Usuario), Objetos de Valor y Excepciones. Sin Django.
Aplicación	src/core/application	Casos de Uso (Use Cases). Define Interfaces (Ports). Orquesta flujos como IniciarSesionUC, CargarPropiedadUC.
Infraestructura	src/infrastructure	Adaptadores: DB (ORM + repositorios), API (controladores), Queue (Celery/Django-Q), Integraciones externas.
🔁 Principio Fundamental

Las dependencias siempre apuntan hacia adentro:
Infraestructura → Aplicación → Dominio

🔒 2. Seguridad y Multitenencia

Tokko Broker implementa un modelo estricto de aislamiento de datos por sucursal.

2.1. Aislamiento por Sucursal

Cada registro crítico contiene un campo obligatorio sucursal_id.

Se implementa un Query Scope Global en los Managers/Repositorios para añadir automáticamente:

WHERE sucursal_id = <ID extraído del token JWT>


Aplicado a modelos como:

Propiedad

Contacto

Oportunidad

2.2. Flujo de Autenticación

El endpoint de login genera un JWT que incluye sucursal_id.

Un Middleware extrae el sucursal_id del token.

Los repositorios y casos de uso lo utilizan para filtrar datos.

🎯 3. Plan de Desarrollo por Módulos (Equipo BK)

El proyecto se divide en módulos funcionales desarrollados en varios Sprints.

🧩 Módulo I: Seguridad, Core y Permisos

Equipo: BK1, BK2, BK3

Dev	Enfoque	Responsabilidades
BK 1 (Líder)	Estructura y Persistencia	Configuración inicial, entidades puras (Usuario, Sucursal), repositorios base.
BK 2	Autenticación	Caso de uso IniciarSesionUC, JWT con sucursal_id, POST /api/auth/login, hashing.
BK 3	Autorización y Multitenencia	Query Scope Global, middleware de extracción de sucursal_id, permisos y grupos.
🏘️ Módulo II: Gestión de Inventario

Equipo: BK4, BK1, BK2

Dev	Enfoque	Responsabilidades
BK 4	CRUD & Multimedia	CrearUsuarioUC, CargarPropiedadUC, subida de multimedia (/media), adaptador S3.
BK 1	Inventario Avanzado	Entidades avanzadas, repositorios completos, PostGIS para búsquedas geográficas.
BK 2	Búsqueda	GET /api/properties con filtros avanzados, optimización de queries.
🔄 Módulo III: Colas, CRM e Integraciones

Equipo: BK3, BK2, BK4

Dev	Enfoque	Responsabilidades
BK 3	Infraestructura de Colas	Configuración Redis/Celery/Django-Q, tabla Publicacion, UC PublicarPropiedadUC.
BK 2	CRM	Entidades y repositorios de Contacto y Oportunidad, GET /api/opportunities.
BK 4	Integraciones Externas	Adaptador de portal externo (Mock), email/WhatsApp, generador de PDF de propiedad.
🧪 4. Guía de Ejecución y Pruebas
4.1. Inyección de Dependencias

Se recomienda:

Contenedor IoC simple

Patrones Factory dentro del Adaptador Web

Objetivo: instanciar Use Cases siempre con los repositorios correctos.

4.2. Pruebas Unitarias

Capa de Dominio y Aplicación deben tener 100% de cobertura.

No dependen de Django → pruebas rápidas y confiables.

4.3. Pruebas de Integración

Repositorios DB (db/repositories.py)

Adaptadores Externos (external/)

Validan:

Mapeo Entidades ⇄ ORM

Comunicación con servicios simulados (Mocks)

🛠️ 5. Configuración del Entorno de Desarrollo
5.1. Clonar Repositorio
git clone https://aws.amazon.com/es/what-is/repo/ tokko-broker-backend
cd tokko-broker-backend

5.2. Crear Entorno Virtual
python -m venv venv
source venv/bin/activate      # Linux/Mac
venv\Scripts\activate         # Windows
pip install -r requirements.txt

5.3. Configuración de Base de Datos

Asegúrate de tener:

PostgreSQL

Extensión PostGIS habilitada

5.4. Variables de Entorno

Copia y renombra:

cp example.env .env


Completa:

SECRET_KEY

DATABASE_URL

REDIS_URL

STORAGE_ADAPTER (S3 o local)

5.5. Migraciones
python manage.py makemigrations
python manage.py migrate

5.6. Iniciar Servidor
python manage.py runserver


API disponible en:

http://127.0.0.1:8000/api/v1/

🚀 Inicio del Desarrollo (Sprint 1)

El equipo BK 1, BK 2 y BK 3 debe comenzar por:

✔ Seguridad
✔ Core del sistema
✔ Flujo de autenticación y multitenencia

Antes de construir cualquier módulo avanzado.
