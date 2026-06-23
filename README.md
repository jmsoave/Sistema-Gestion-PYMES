# Sistema de Gestión Logística y Facturación Automatizada
### Micro-ERP para Pymes del sector logístico

Sistema de escritorio offline-first diseñado para automatizar el ciclo de vida
de pedidos, control de inventario y emisión de comprobantes fiscales (CAE) para
Pymes del sector logístico en Argentina.

Resuelve la fricción operativa entre el manejo de pedidos tradicional basado en
planillas Excel y la obligatoriedad de facturación electrónica exigida por ARCA
(ex-AFIP), operando de forma resiliente en entornos con conectividad inestable.

---

## Problema que resuelve

En el sector logístico tradicional, la carga manual de remitos y la facturación
individual generan cuellos de botella diarios que impactan directamente en la
capacidad operativa del negocio. Este sistema fue diseñado bajo tres premisas:

**Fricción cero** El sistema se adapta al flujo de trabajo existente del
cliente. Ingiere directamente las planillas Excel de choferes y vendedores sin
forzar una carga manual redundante.

**Operación local (offline-first)** La base de datos opera localmente,
garantizando que la consulta de stock, precios y armado de hojas de ruta
funcione a máxima velocidad sin depender de infraestructura en la nube.

**Trazabilidad absoluta** Registro histórico de movimientos de inventario
para auditorías de entrada y salida de mercadería.

---

## Stack tecnológico

| Capa | Tecnología | Justificación |
|---|---|---|
| Lenguaje | Python 3.x | Ecosistema maduro para automatización y procesamiento de datos |
| Interfaz gráfica | PySide6 | Arquitectura basada en eventos, licencia LGPL para uso comercial |
| Base de datos | SQLite | Portabilidad total, cero configuración de servidores locales |
| ORM | SQLAlchemy 2.0 | Abstracción de datos, prevención de inyecciones SQL, preparado para migración a PostgreSQL |
| Integración fiscal | PyAfipWs | Comunicación SOAP con los Web Services de ARCA |
| Procesamiento Excel | Pandas / OpenPyXL | Parseo y validación de archivos de entrada |

---

## Arquitectura y flujos core

El sistema no es un CRUD convencional. Es un orquestador de procesos logísticos
estructurado en tres pilares:

### 1. Ingesta y validación de pedidos

El operador carga la nota de pedido en formato `.xlsx`. El sistema ejecuta una
validación atómica cruzando los ítems solicitados contra el `stock_actual` en
base de datos.

- **Camino feliz:** se reserva el stock y se aprueba la hoja de ruta del camión.
- **Excepción:** si hay faltantes, el sistema genera automáticamente una orden
  de compra en estado `PENDIENTE` para el proveedor correspondiente.

### 2. Resiliencia fiscal e idempotencia

El mayor riesgo operativo en facturación electrónica es la duplicación de
comprobantes por caídas en los servidores de ARCA. La integración con WSFE
(Web Service de Factura Electrónica) está diseñada bajo el principio de
idempotencia:

- Las facturas se persisten en base de datos local en estado `PENDIENTE` antes
  de cualquier transmisión de red.
- Ante un timeout, el sistema no permite el reintento a ciegas. Ejecuta
  `FECompConsultar` para verificar el estado real en los servidores de ARCA,
  recuperando el CAE si la factura fue procesada exitosamente de forma remota
  pero se perdió el paquete de respuesta.
- Esto elimina el riesgo de duplicaciones impositivas críticas.

### 3. Generación de documentos operativos

A partir del resultado de la verificación de stock, el sistema genera
automáticamente los documentos necesarios para la operación diaria: hojas de
ruta del camión, órdenes de compra a proveedores y facturas electrónicas en PDF
con CAE incluido.

---

## Documentación de ingeniería

### Diagrama de flujo de datos (DFD)

Explosión de procesos desde el Nivel 0 (contexto) hasta el Nivel 1, mostrando
el viaje de la información entre actores externos y almacenes de datos.

![Diagrama DFD](Diagrama%20DFD.png)

### Diagrama entidad-relación (DER)

Diseño relacional de la base de datos. Destaca la normalización de entidades
(Clientes, Productos, Viajes) y las tablas intermedias para soportar un modelo
de datos escalable.

![Diagrama ER](Diagrama%20ER.png)

### Diagrama de secuencia (UML)

Interacción temporal entre los objetos del sistema durante el proceso de
facturación, con bloques de validación `alt faltantes / con stock`.

![Diagrama de Secuencia](Diagrama%20Secuencia.png)

### Diagrama de casos de uso

Fronteras del sistema y comportamientos esperados por los distintos roles de
usuario.

![Diagrama de Casos de Uso](Diagrama%20CU.png)

---

## Roadmap

El MVP cubre el ciclo completo de pedidos, stock y facturación local.
Las siguientes etapas están planificadas:

- **Migración de persistencia** — Transición a PostgreSQL para soportar
  concurrencia en red local (LAN) con múltiples operadores simultáneos.
- **Business intelligence** — Exposición de la base de datos para integración
  nativa con Power BI: dashboards de rendimiento por chofer y análisis de
  ventas por zona.
- **Portal de clientes** — Interfaz web liviana para que los clientes externos
  carguen sus propias notas de pedido, eliminando el paso de ingesta manual.
- **App móvil para repartidores** — Confirmación de entregas desde el celular
  del chofer, con actualización automática del estado del viaje.

---

## Contacto

**Juan Martino Soave**
Estudiante de Ingenieria en Informatica — UNLaM, Buenos Aires

Disponible para roles Junior en Analisis de Datos y Automatizacion de Procesos.

- LinkedIn: [linkedin.com/in/juan-martino-soave-4722682a1](https://linkedin.com/in/juan-martino-soave-4722682a1)
- GitHub: [github.com/jmsoave](https://github.com/jmsoave)
- Email: soavejuanmartino@gmail.com
