# Motor de Generación de Reportes PDF con ReportLab

El backend de CTT-FISEI incorpora un motor de renderizado y maquetación de documentos PDF vectoriales en memoria basado en la biblioteca **ReportLab**. Este motor se encarga de emitir dos tipos de documentos oficiales esenciales para la operación universitaria:
1. El **Comprobante Oficial RIDE SRI** (Representación Impresa del Documento Electrónico).
2. La **Orden de Pago Oficial UTA** (para ventanillas de recaudación y depósitos bancarios).

---

## 1. Comprobante Oficial RIDE SRI (`ride_generator.py`)

El generador `src/utils/ride_generator.py` construye el comprobante electrónico cumpliendo estrictamente con la ficha técnica y la normativa tributaria emitida por el Servicio de Rentas Internas (SRI) del Ecuador para instituciones de educación superior públicas.

### Características y Campos Institucionales del RIDE

- **RUC Institucional de la UTA**: `1860000880001` (Razón Social: *Universidad Técnica de Ambato*).
- **Clave de Acceso Tributaria SRI**: Cadena numérica calculada de **49 dígitos**, estructurada según la especificación técnica:
  - Fecha de emisión (`ddmmaaaa`).
  - Tipo de comprobante (`01` = Factura).
  - RUC institucional (`1860000880001`).
  - Tipo de ambiente (`2` = Producción).
  - Serie del establecimiento y punto de emisión (`001002`).
  - Secuencial del comprobante (`9` dígitos con ceros a la izquierda: `000000012`).
  - Código numérico aleatorio (`8` dígitos).
  - Dígito verificador ponderado módulo 11.
- **Desglose de Tarifas Académicas Oficiales FISEI**:
  - **Tarifa Estudiante UTA**: $20.00 USD (estudiantes de pregrado con matrícula activa).
  - **Tarifa Profesional / General**: $50.00 USD (profesionales, egresados o externos).
  - **Tarifa Ponente**: $80.00 USD (autores de artículos científicos aprobados por el comité).
- **Tratamiento Tributario del IVA**: De conformidad con la Ley de Régimen Tributario Interno del Ecuador, los servicios de educación y capacitación impartidos por universidades públicas están gravados con **Tarifa 0% de IVA**:
  - `SUBTOTAL 0%`: Equivalente al total pagado ($20.00 / $50.00 / $80.00).
  - `SUBTOTAL 15% (o tarifa vigente)`: $0.00.
  - `IVA (0%)`: $0.00.
  - `VALOR TOTAL`: Monto neto facturado.
- **Datos Fiscales del Cliente / Adquiriente**:
  - RUC o Cédula de identidad del comprador (del perfil de facturación `billing_identification`).
  - Razón social o nombres completos (`billing_name`).
  - Dirección física (`billing_address`), teléfono (`billing_phone`) y correo electrónico para envío de factura (`billing_email`).

### Código QR de Validación Tributaria

El comprobante incluye un código QR generado con la librería `qrcode` con nivel de corrección de errores medio (`ERROR_CORRECT_M`), coloreado con el azul institucional (`#00345e`), que codifica en formato JSON la clave de acceso, el RUC emisor, el número de factura y el monto total para verificación óptica inmediata.

---

## 2. Generador de Órdenes de Pago Institucionales UTA (`uta_order_generator.py`)

Para los participantes que eligen pagar en ventanillas de la Dirección Financiera en el Campus Huachi o mediante depósito/transferencia a la cuenta institucional de la UTA en el Banco del Pacífico, el sistema genera la **Orden de Pago UTA** (`src/utils/uta_order_generator.py`).

### Estructura del Documento

1. **Cabecera Institucional**:
   - Escudo oficial de la Universidad Técnica de Ambato y siglas FISEI / CTT.
   - Código único de orden: `UTA-FISEI-2026-{enrollment_id:06d}`.
   - Fecha de emisión y fecha límite de vencimiento (por defecto 5 días hábiles a partir de la emisión).
2. **Datos del Beneficiario y Cuenta Institucional**:
   - **Institución**: Universidad Técnica de Ambato.
   - **RUC**: `1860000880001`.
   - **Entidad Bancaria**: Banco del Pacífico.
   - **Tipo y Número de Cuenta**: Cuenta Corriente Nº `07310034`.
   - **Sublínea Contable**: Ingresos Autogestionados CTT - FISEI.
3. **Instrucciones de Conciliación**: Detalla el procedimiento paso a paso para que el estudiante cargue la fotografía o escaneo legible de la papeleta de depósito en la plataforma web a fin de que sea validada por el personal contable.

---

## 3. Clase `UncompressedCanvas` para Auditoría Binaria

Una particularidad técnica clave de ambos generadores es el uso de un lienzo (`canvas.Canvas`) personalizado:

```python
from reportlab.pdfgen import canvas

class UncompressedCanvas(canvas.Canvas):
    """
    Canvas personalizado con compresión deshabilitada para garantizar
    que las firmas y cadenas institucionales como el RUC 1860000880001
    sean inspeccionables directamente a nivel binario en pruebas de auditoría.
    """
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self._pageCompression = 0
```

### Justificación de Ingeniería

Al desactivar la compresión de flujo (`self._pageCompression = 0` y `doc.pageCompression = 0`), los flujos de texto del PDF no se comprimen mediante el algoritmo Deflate/zlib. Esto permite que:
- Las suites automatizadas de pruebas de integración y los auditores de seguridad puedan verificar directamente con aserciones binarias (`assert b"1860000880001" in res.content`) la presencia exacta de los datos fiscales sin necesidad de descomprimir o parsear el árbol de objetos PDF.
- Los lectores OCR de ventanilla e indexadores bancarios procesen los identificadores institucionales de forma inmediata y sin distorsiones tipográficas.

---

## 4. Endpoints de Descarga y Contrato HTTP

Ambos reportes se entregan a través del router de inscripciones (`enrollments_router.py`):

### Descarga del Comprobante RIDE
- **Ruta**: `GET /api/v1/enrollments/{enrollment_id}/ride/download`
- **Autenticación**: Token de portador (Admin o Participante titular de la inscripción).
- **Respuesta Exitosa**:
  - `Status Code`: `200 OK`
  - `Content-Type`: `application/pdf`
  - `Content-Disposition`: `attachment; filename="RIDE_UTA_000000012.pdf"`
  - `Cuerpo`: Flujo binario de bytes que inicia con el encabezado `%PDF-1.4`.

### Descarga de la Orden de Pago UTA
- **Ruta**: `GET /api/v1/enrollments/{enrollment_id}/payment-order/download`
- **Autenticación**: Token de portador.
- **Respuesta Exitosa**:
  - `Status Code`: `200 OK`
  - `Content-Type`: `application/pdf`
  - `Content-Disposition`: `attachment; filename="Orden_Pago_UTA_000012.pdf"`

En caso de que el participante intente descargar un RIDE de una inscripción que aún no ha sido pagada o facturada, el backend retorna un código `HTTP 400 Bad Request` indicando que el comprobante fiscal únicamente se encuentra disponible para matrículas con pago acreditado.
