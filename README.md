🏡 Tokko Broker: Plataforma de Gestión Inmobiliaria (Back-End)

Este repositorio contiene la implementación del Back-End para la plataforma de gestión inmobiliaria Tokko Broker, desarrollado bajo la Arquitectura Hexagonal (Ports and Adapters) utilizando Python (Django), PostgreSQL (PostGIS) y Redis para el procesamiento asíncrono.

El objetivo principal es proveer una API RESTful robusta, segura y escalable, con estricto aislamiento de datos para cada Sucursal (Multitenencia).

🏗️ 1. Arquitectura: Hexagonal (Ports and Adapters)

Hemos adoptado la Arquitectura Hexagonal para asegurar que la lógica de negocio (Dominio) sea independiente de la tecnología (Django ORM, APIs externas). Esto facilita las pruebas unitarias y garantiza un bajo acoplamiento.

Estructura de Carpetas Clave:

Capa

Carpeta Principal

Rol y Enfoque del Código

I. Dominio

src/core/domain

Reglas de Negocio Puras. Contiene Entidades (Propiedad, Usuario), Objetos de Valor, y Excepciones. No hay código de Django aquí.

II. Aplicación

src/core/application

Casos de Uso (Use Cases - UC). Define las Interfaces (Ports) y orquesta el flujo de la aplicación (IniciarSesionUC, CargarPropiedadUC).

III. Infraestructura

src/infrastructure

Adaptadores (Adapters). Conecta el Core con el mundo exterior: db/ (Django ORM, Repositorios), api/ (Vistas/Controladores), queue/ (Workers de Celery/Django-Q), external/ (Integraciones API).

Principio Fundamental: Las dependencias siempre apuntan hacia el interior: Infraestructura → Aplicación → Dominio.

🔒 2. Seguridad y Multitenencia (Regla de Oro)

El sistema opera bajo un esquema de Multitenencia por Sucursal. La seguridad es la máxima prioridad.

2.1. Aislamiento de Datos por Sucursal

Identificador (Tenant ID): sucursal_id (obligatorio en tablas críticas).

Mecanismo: El desarrollador BK 3 es responsable de implementar un Query Scope Global en los Managers de Django (en la capa de Infraestructura/DB) que automáticamente añade la condición WHERE sucursal_id = [ID del Token] a todas las consultas de lectura y escritura en los modelos sensibles (Propiedad, Contacto, Oportunidad).

2.2. Flujo de Autenticación

El token JWT retornado tras el login DEBE incluir el sucursal_id en su payload.

Un Middleware de seguridad debe extraer este sucursal_id y ponerlo a disposición del contexto de la petición para que los Repositorios y Use Cases puedan utilizarlo.

🎯 3. Plan de Desarrollo y Módulos del Equipo BK (4 Desarrolladores)

El desarrollo está organizado en módulos funcionales y asignado de acuerdo al Plan de Trabajo (Sprints 1, 2 y 3).

Módulo I: Seguridad, Core y Permisos (BK 1, BK 2, BK 3)

Este módulo es la base y requiere la máxima coordinación para asegurar la coherencia arquitectónica.

Desarrollador

Enfoque Principal

Responsabilidades Clave

BK 1 (Líder)

Estructura y Persistencia

Configuración inicial, Definición de Entidades Puras (Usuario, Sucursal), Implementación de Repositorios base (Adaptadores DB).

BK 2

Autenticación

IniciarSesionUC (Generación de JWT con sucursal_id), POST /api/auth/login, Implementación de Password Hashing.

BK 3

Autorización y Multitenencia

Implementación del Query Scope Global (Aislamiento de Sucursales), Middleware de Extracción de sucursal_id, Implementación de Permiso y GrupoPermisos.

Módulo II: Gestión de Inventario (BK 4, BK 1, BK 2)

Este módulo se centra en las entidades principales de Tokko Broker.

Desarrollador

Enfoque Principal

Responsabilidades Clave

BK 4

CRUD Básico & Multimedia

CrearUsuarioUC, CargarPropiedadUC (primera implementación CRUD), POST /api/properties/{id}/media (Adaptador de Storage S3).

BK 1

Inventario Core Avanzado

Entidades Puras (Propiedad, Emprendimiento), Repositorios completos, Configuración de PostGIS para búsquedas geográficas.

BK 2

Búsqueda y Filtrado

GET /api/properties con filtros avanzados (precio, dormitorios, cercanía geográfica). Optimización de consultas.

Módulo III: Colas, CRM e Integraciones (BK 3, BK 2, BK 4)

Este módulo requiere la implementación de Puertos y Adaptadores de salida para la comunicación asíncrona.

Desarrollador

Enfoque Principal

Responsabilidades Clave

BK 3

Infraestructura de Colas

Configuración de Redis/Celery/Django-Q (Workers), Implementación de la tabla Publicacion (Jobs) [cite: 7.7], Creación del Trigger PublicarPropiedadUC (que encola el Job).

BK 2

CRM

Entidades y Repositorios de Contacto y Oportunidad, Use Cases de gestión de Leads y GET /api/opportunities.

BK 4

Adaptadores de Salida

Adaptador de Integración (Mock de Portal Externo), Adaptador de Servicio de Email/WhatsApp, Lógica para Generar PDF de Ficha (Preparación para notificaciones).

4. Guía de Ejecución y Pruebas

4.1. Inyección de Dependencias

Se recomienda usar un contenedor de Inversión de Control (IoC) simple o un patrón de Fábrica dentro del Adaptador Web para instanciar los Use Cases, asegurando que siempre se inyecten los Repositorios de Django (Adaptadores de Infraestructura).

4.2. Pruebas Unitarias

Foco: La capa de Dominio y Aplicación debe tener una cobertura del 100% de pruebas unitarias.

Ventaja Hexagonal: Al ser código Python puro, estas pruebas no necesitan Django ni la base de datos (PostgreSQL), lo que las hace rápidas y confiables.

4.3. Pruebas de Integración

Foco: Repositorios (db/repositories.py) y Adaptadores Externos (external/).

Estas pruebas deben validar el mapeo correcto entre las Entidades Puras de Dominio y los Modelos de Django ORM, y la comunicación con servicios simulados (Mocks).

5. Configuración del Entorno de Desarrollo

Clonar Repositorio:

git clone [https://aws.amazon.com/es/what-is/repo/](https://aws.amazon.com/es/what-is/repo/) tokko-broker-backend
cd tokko-broker-backend


Configurar Entorno Virtual:

python -m venv venv
source venv/bin/activate  # o venv\Scripts\activate en Windows
pip install -r requirements.txt


Base de Datos (PostgreSQL + PostGIS):
Asegúrate de tener un servidor PostgreSQL con la extensión PostGIS habilitada.

Variables de Entorno:
Copia example.env a .env y rellena las variables, incluyendo:

SECRET_KEY de Django.

Credenciales de DATABASE_URL (PostgreSQL).

Credenciales de REDIS_URL (para Colas).

STORAGE_ADAPTER (S3 o Local).

Ejecutar Migraciones:

python manage.py makemigrations
python manage.py migrate


Iniciar Servidor:

python manage.py runserver


El API estará disponible en http://127.0.0.1:8000/api/v1/.

¡Manos a la obra! El equipo debe comenzar por el Sprint 1 (Seguridad y Core), con BK 1, 2 y 3 sentando las bases de la arquitectura y el flujo de seguridad antes de implementar cualquier funcionalidad de negocio compleja.
