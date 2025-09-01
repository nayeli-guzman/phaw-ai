# 📚 Documentación Completa de APIs - Plataforma de Inteligencia Oncológica

## 🔗 Endpoints Base

- **Servidor FHIR:** `https://cancer-care-fhir-api.onrender.com`
- **Stream HL7:** `https://cancer-care-hl7-stream.onrender.com`  
- **API FDA:** `https://cancer-care-fda-api.onrender.com`
- **Clinical AI:** `https://cancer-care-ai-api.onrender.com`

---

# 🔐 API FHIR (Datos de Pacientes)

## Autenticación OAuth2

**Endpoint:** `POST /oauth/token`

```bash
curl -X POST https://cancer-care-fhir-api.onrender.com/oauth/token \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=test_client&client_secret=test_secret"
```

**Respuesta exitosa:**
```json
{
  "access_token": "uuid-token-aqui",
  "token_type": "Bearer", 
  "expires_in": 3600,
  "refresh_token": "refresh-token-opcional"
}
```

**Errores comunes:**
```json
{
  "error": "invalid_grant",
  "error_description": "El tipo de grant proporcionado no es válido"
}
```

## Obtener Pacientes

**Endpoint:** `GET /fhir/Patient`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  https://cancer-care-fhir-api.onrender.com/fhir/Patient?_count=10
```

**Parámetros de consulta:**
- `_count` - Número de pacientes a retornar (default: 10)

**Respuesta exitosa:**
```json
{
  "resourceType": "Bundle",
  "type": "searchset", 
  "total": 5,
  "entry": [
    {
      "resource": {
        "resourceType": "Patient",
        "id": "paciente-1",
        "identifier": [
          {
            "system": "http://hospital.saludinteligente.ejemplo/pacientes",
            "value": "MRN123456"
          }
        ],
        "name": [
          {
            "family": "García",
            "given": ["María"]
          }
        ],
        "gender": "female",
        "birthDate": "1985-03-15",
        "address": [
          {
            "line": ["Calle 123 #45"],
            "city": "México",
            "state": "CDMX",
            "postalCode": "12345"
          }
        ],
        "telecom": [
          {
            "system": "phone",
            "value": "55-1234-5678"
          }
        ]
      }
    }
  ],
  "link": [
    {
      "relation": "next",
      "url": "/fhir/Patient?_getpages=uuid&_count=10"
    }
  ]
}
```

**Errores esperados:**
```json
{
  "resourceType": "OperationOutcome",
  "issue": [
    {
      "severity": "error",
      "code": "processing",
      "diagnostics": "Ocurrió un error interno del servidor"
    }
  ]
}
```

## Obtener Observaciones de Laboratorio

**Endpoint:** `GET /fhir/Observation`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://cancer-care-fhir-api.onrender.com/fhir/Observation?subject=Patient/paciente-1"
```

**Parámetros de consulta:**
- `subject` - Referencia al paciente (ej: Patient/paciente-1)
- `code` - Código LOINC del laboratorio
- `date` - Fecha de la observación

**Respuesta:**
```json
{
  "resourceType": "Bundle",
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "Observation",
        "id": "obs-1",
        "status": "final",
        "category": [
          {
            "coding": [
              {
                "system": "http://terminology.hl7.org/CodeSystem/observation-category",
                "code": "laboratory",
                "display": "Laboratorio"
              }
            ]
          }
        ],
        "code": {
          "coding": [
            {
              "system": "http://loinc.org",
              "code": "718-7",
              "display": "Hemoglobina"
            }
          ]
        },
        "subject": {
          "reference": "Patient/paciente-1"
        },
        "effectiveDateTime": "2024-01-15T10:30:00Z",
        "valueQuantity": {
          "value": 12.5,
          "unit": "g/dL",
          "system": "http://unitsofmeasure.org"
        }
      }
    }
  ]
}
```

## Obtener Solicitudes de Medicamentos

**Endpoint:** `GET /fhir/MedicationRequest`

```bash
curl -H "Authorization: Bearer YOUR_TOKEN" \
  "https://cancer-care-fhir-api.onrender.com/fhir/MedicationRequest?subject=Patient/paciente-1"
```

**Respuesta:**
```json
{
  "resourceType": "Bundle",
  "type": "searchset",
  "entry": [
    {
      "resource": {
        "resourceType": "MedicationRequest",
        "id": "med-req-1",
        "status": "active",
        "intent": "order",
        "medicationCodeableConcept": {
          "coding": [
            {
              "system": "http://www.nlm.nih.gov/research/umls/rxnorm",
              "code": "RX123456",
              "display": "Paracetamol"
            }
          ],
          "text": "Paracetamol"
        },
        "subject": {
          "reference": "Patient/paciente-1"
        },
        "authoredOn": "2024-01-15",
        "dosageInstruction": [
          {
            "text": "Tomar 1 vez al día",
            "timing": {
              "repeat": {
                "frequency": 1,
                "period": 1,
                "periodUnit": "d"
              }
            }
          }
        ]
      }
    }
  ]
}
```

## Health Check

**Endpoint:** `GET /health`

```bash
curl https://cancer-care-fhir-api.onrender.com/health
```

**Respuesta:**
```json
{
  "status": "operational",
  "timestamp": "2024-01-15T10:30:00Z",
  "service": "FHIR Server",
  "version": "R4"
}
```

---

# 📊 API Stream HL7 (Datos de Laboratorio)

## Obtener Mensajes HL7 (REST)

**Endpoint:** `GET /hl7/messages`

```bash
curl https://cancer-care-hl7-stream.onrender.com/hl7/messages
```

**Respuesta:**
```json
{
  "messages": [
    {
      "id": "msg_123456",
      "timestamp": "2024-01-15T10:30:00Z",
      "message": "MSH|^~\\&|LAB|HOSPITAL|RECEPTOR|RECEPTOR|20240115103000||ORU^R01|MSG123|P|2.5.1\rPID|1||P123456^^^MR||GARCÍA^MARÍA||19850315|F\rOBR|1|ORD123|RES123|CBC^BIOMETRIA^L|||20240115103000\rOBX|1|NM|WBC^GLOBULOS BLANCOS^L||7.5|10*3/uL|4.0-11.0||||F|||20240115103000",
      "source": "SISTEMA_LAB",
      "status": "procesado"
    }
  ],
  "total": 1,
  "timestamp": "2024-01-15T10:30:00Z",
  "status": "ok"
}
```

## Estadísticas del Stream

**Endpoint:** `GET /hl7/stats`

```bash
curl https://cancer-care-hl7-stream.onrender.com/hl7/stats
```

**Respuesta:**
```json
{
  "queue_size": 45,
  "timestamp": "2024-01-15T10:30:00Z",
  "processing_rate": "125 msg/min",
  "error_rate": "15%",
  "last_message": "2024-01-15T10:29:45Z",
  "system_status": "operacional"
}
```

## Información del Stream

**Endpoint:** `GET /hl7/stream/info`

```bash
curl https://cancer-care-hl7-stream.onrender.com/hl7/stream/info
```

**Respuesta:**
```json
{
  "websocket_url": "ws://localhost:8765",
  "status": "activo",
  "supported_versions": ["2.3", "2.5", "2.5.1"],
  "message_types": ["ORU^R01", "ADT^A01", "ORM^O01"],
  "encoding": ["UTF-8", "ISO-8859-1"],
  "note": "El stream puede incluir mensajes malformados para propósitos de prueba"
}
```

## Formato de Mensajes HL7

Los mensajes siguen el estándar HL7 v2.x con estos segmentos principales:

### Estructura Básica:
```
MSH|^~\&|LAB|HOSPITAL|RECEPTOR|RECEPTOR|timestamp||ORU^R01|msg_id|P|version
PID|1||patient_id^^^MR||APELLIDO^NOMBRE||birth_date|gender
OBR|1|order_num|result_num|test_code^test_name^L|||order_time
OBX|1|NM|test_code^test_name^L||value|units|range||||F|||result_time
```

---

# 💊 API FDA (Información de Medicamentos)

## Buscar Medicamentos

**Endpoint:** `GET /drug/drugsfda.json`

```bash
curl "https://cancer-care-fda-api.onrender.com/drug/drugsfda.json?search=aspirina&limit=5"
```

**Parámetros:**
- `search` - Nombre del medicamento (requerido, coincidencia exacta)
- `limit` - Número máximo de resultados (default: 10, max: 100)
- `detailed` - Incluir información detallada (opcional)

**Respuesta exitosa:**
```json
{
  "meta": {
    "disclaimer": "No se base en openFDA para tomar decisiones médicas.",
    "terms": "https://open.fda.gov/terms/",
    "license": "https://open.fda.gov/license/",
    "last_updated": "2024-01-15",
    "results": {
      "skip": 0,
      "limit": 5,
      "total": 1
    }
  },
  "results": [
    {
      "application_number": "NDA123456",
      "sponsor_name": "Laboratorios Pisa",
      "openfda": {
        "application_number": ["NDA123456"],
        "brand_name": ["Aspirina"],
        "generic_name": ["aspirina"],
        "manufacturer_name": ["Pisa"],
        "product_ndc": ["12345-678-90"],
        "product_type": ["MEDICAMENTO DE PRESCRIPCIÓN HUMANA"],
        "route": ["ORAL"],
        "substance_name": ["ASPIRINA"],
        "rxcui": ["123456"],
        "spl_id": ["spl-123456"],
        "spl_set_id": ["set-123456"],
        "package_ndc": ["12345-678-90"],
        "unii": ["UNII123456"]
      }
    }
  ]
}
```

**Respuesta detallada (con parámetro `detailed`):**
```json
{
  "results": [
    {
      "application_number": "NDA123456",
      "sponsor_name": "Laboratorios Pisa",
      "openfda": { /* igual que arriba */ },
      "products": [
        {
          "product_number": "001",
          "reference_drug": "Sí",
          "brand_name": "Aspirina",
          "active_ingredients": [
            {
              "name": "ASPIRINA",
              "strength": "500MG"
            }
          ],
          "reference_standard": "Sí",
          "dosage_form": "TABLETA",
          "route": "ORAL",
          "marketing_status": "Prescripción"
        }
      ]
    }
  ]
}
```

## Obtener Etiquetado e Interacciones

**Endpoint:** `GET /drug/label.json`

```bash
curl "https://cancer-care-fda-api.onrender.com/drug/label.json?search=aspirina"
```

**Respuesta (formato nuevo):**
```json
{
  "meta": {
    "disclaimer": "No se base en openFDA para tomar decisiones médicas.",
    "results": {
      "skip": 0,
      "limit": 1,
      "total": 1
    }
  },
  "results": [
    {
      "spl_product_data_elements": ["ASPIRINA"],
      "openfda": {
        "application_number": ["NDA123456"],
        "brand_name": ["Aspirina"],
        "generic_name": ["aspirina"],
        "manufacturer_name": ["Laboratorios Modernos"],
        "product_ndc": ["12345-678-90"],
        "product_type": ["MEDICAMENTO DE PRESCRIPCIÓN HUMANA"],
        "route": ["ORAL"],
        "substance_name": ["ASPIRINA"]
      },
      "warnings_and_cautions": [
        "Advertencia: aspirina puede causar riesgo aumentado de sangrado"
      ],
      "drug_interactions": [
        {
          "drug_name": "warfarina",
          "interaction_significance": "major",
          "interaction_description": "riesgo aumentado de sangrado"
        },
        {
          "drug_name": "metotrexato",
          "interaction_significance": "moderate", 
          "interaction_description": "toxicidad"
        }
      ],
      "contraindications": [
        "úlcera péptica activa",
        "trastornos de coagulación"
      ],
      "indications_and_usage": [
        "Indicado para tratamiento cuando aspirina es apropiado"
      ],
      "dosage_and_administration": [
        {
          "dosage_form": "tableta",
          "route_of_administration": "oral",
          "dosage_strength": "500mg"
        }
      ]
    }
  ]
}
```

**Respuesta (formato legacy - cambia según hora del día):**
```json
{
  "results": [
    {
      "set_id": "legacy-123456",
      "id": "label-123456",
      "effective_time": "20240101",
      "version": "1",
      "openfda": {
        "brand_name": ["Aspirina"],
        "generic_name": ["aspirina"],
        "manufacturer_name": ["Laboratorios Legacy"],
        "product_ndc": ["12345-678-90"]
      },
      "drug_interactions": [
        "Puede interactuar con warfarina",
        "Puede interactuar con metotrexato"
      ],
      "contraindications": [
        "No usar con warfarina",
        "No usar con metotrexato"
      ],
      "warnings": ["Estos son datos en formato legacy"],
      "indications_and_usage": ["Aspirina está indicado para varias condiciones"],
      "dosage_and_administration": ["Tomar aspirina según las indicaciones del médico"]
    }
  ]
}
```

## Datos de Retiros y Cumplimiento

**Endpoint:** `GET /drug/enforcement.json`

```bash
curl "https://cancer-care-fda-api.onrender.com/drug/enforcement.json?search=aspirina"
```

**Respuesta:**
```json
{
  "meta": {
    "disclaimer": "No se base en openFDA para tomar decisiones médicas.",
    "results": {
      "skip": 0,
      "limit": 100,
      "total": 2
    }
  },
  "results": [
    {
      "recall_number": "F-1234-2024",
      "reason_for_recall": "Contaminación potencial en lotes de aspirina",
      "status": "En curso",
      "distribution_pattern": "Nacional",
      "product_description": "Aspirina tabletas",
      "code_info": "Lote #123456",
      "product_quantity": "50000 unidades",
      "recall_initiation_date": "2024-01-15",
      "state": "CDMX",
      "event_id": "123456",
      "product_type": "Medicamentos",
      "termination_date": "",
      "more_code_info": "Información adicional del lote: 78901",
      "recalling_firm": "Laboratorios ABC",
      "classification": "Clase II",
      "openfda": {
        "application_number": ["ANDA123456"],
        "brand_name": ["Aspirina"],
        "generic_name": ["aspirina"],
        "manufacturer_name": ["Laboratorios ABC"],
        "product_ndc": ["12345-678-90"],
        "product_type": ["MEDICAMENTO DE PRESCRIPCIÓN HUMANA"]
      }
    }
  ]
}
```

## Endpoint Específico de Interacciones

**Endpoint:** `GET /drug/interactions.json`

```bash
curl "https://cancer-care-fda-api.onrender.com/drug/interactions.json?search=aspirina"
```

**Respuesta:**
```json
{
  "drug_name": "aspirina",
  "interactions": [
    {
      "interacting_drug": "warfarina",
      "severity": "major",
      "effect": "riesgo aumentado de sangrado",
      "recommendation": "Monitorear cuando se use aspirina con warfarina",
      "evidence_level": "alta",
      "source": "Base de datos simulada FDA"
    },
    {
      "interacting_drug": "metotrexato",
      "severity": "moderate",
      "effect": "toxicidad",
      "recommendation": "Monitorear cuando se use aspirina con metotrexato",
      "evidence_level": "moderada",
      "source": "Base de datos simulada FDA"
    }
  ],
  "contraindications": [
    "úlcera péptica activa",
    "trastornos de coagulación"
  ],
  "total_interactions": 2,
  "last_updated": "2024-01-15",
  "disclaimer": "Datos simulados - no usar para decisiones clínicas reales"
}
```

## Información de la API

**Endpoint:** `GET /api/info`

```bash
curl https://cancer-care-fda-api.onrender.com/api/info
```

**Respuesta:**
```json
{
  "service": "API FDA Simulada",
  "version": "1.0.0",
  "endpoints": [
    "/drug/drugsfda.json?search=<nombre_medicamento>",
    "/drug/label.json?search=<nombre_medicamento>",
    "/drug/enforcement.json?search=<nombre_medicamento>",
    "/drug/interactions.json?search=<nombre_medicamento>"
  ],
  "known_drugs": [
    "aspirina", "warfarina", "metformina", 
    "lisinopril", "simvastatina", "paracetamol", 
    "ibuprofeno", "omeprazol"
  ],
  "rate_limit": "10 solicitudes por minuto",
  "note": "Esta API tiene problemas intencionales para propósitos de prueba",
  "issues": [
    "Límites de tasa más estrictos que lo documentado",
    "Requiere coincidencia exacta de nombres (sin búsqueda difusa)",
    "Errores 503 frecuentes",
    "Caché que retorna datos obsoletos",
    "Formato de respuesta que cambia según parámetros"
  ]
}
```

---

# 🧠 API Clinical AI Assistant (LLMs + RAG)

## Análisis de Síntomas con LLM

**Endpoint:** `POST /ai/analyze-symptoms`

```bash
curl -X POST https://cancer-care-ai-api.onrender.com/ai/analyze-symptoms \
  -H "Content-Type: application/json" \
  -d '{
    "symptoms": "fatiga extrema, náuseas, entumecimiento en manos",
    "medical_history": "cáncer de mama, quimioterapia AC",
    "current_medications": ["doxorubicina", "ciclofosfamida", "ondansetron"]
  }'
```

**Campos requeridos:**
- `symptoms` - Descripción de síntomas del paciente (requerido)
- `medical_history` - Historial médico relevante (opcional)
- `current_medications` - Lista de medicamentos actuales (opcional)

**Respuesta exitosa:**
```json
{
  "analysis": "Basado en los síntomas presentados, es importante considerar que la quimioterapia puede causar múltiples efectos secundarios. Los síntomas descritos podrían estar relacionados con neutropenia o neuropatía periférica. Recomiendo: 1) Evaluación inmediata de conteo sanguíneo, 2) Evaluación neurológica, 3) Ajuste de dosis si es necesario. Nivel de urgencia: 7/10.",
  "confidence": 0.87,
  "tokens_used": 245,
  "model": "gpt-4-turbo",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

**Errores comunes:**
```json
{
  "error": "LLM_API_ERROR",
  "message": "Error de conexión con el modelo de lenguaje",
  "retry_after": 60
}
```

```json
{
  "error": "RATE_LIMIT_EXCEEDED",
  "message": "Has excedido el límite de tokens por minuto",
  "retry_after": 60
}
```

```json
{
  "error": "CONTEXT_TOO_LONG",
  "message": "El prompt excede el límite de tokens del modelo",
  "max_tokens": 2000
}
```

## Búsqueda RAG en Base de Conocimiento

**Endpoint:** `POST /ai/knowledge-search`

```bash
curl -X POST https://cancer-care-ai-api.onrender.com/ai/knowledge-search \
  -H "Content-Type: application/json" \
  -d '{
    "query": "efectos secundarios quimioterapia neuropatía",
    "max_results": 3
  }'
```

**Parámetros:**
- `query` - Consulta de búsqueda en lenguaje natural (requerido)
- `max_results` - Máximo número de resultados (default: 3, max: 10)

**Respuesta exitosa:**
```json
{
  "query": "efectos secundarios quimioterapia neuropatía",
  "results": [
    {
      "id": "doc_004",
      "title": "Neuropatía Inducida por Quimioterapia",
      "content": "La neuropatía periférica afecta hasta 70% de pacientes con taxanos. Síntomas incluyen entumecimiento y dolor en manos/pies.",
      "relevance_score": 0.92,
      "source": "Neurology in Cancer Care 2024"
    },
    {
      "id": "doc_001",
      "title": "Protocolos de Quimioterapia en Cáncer de Mama",
      "content": "Los protocolos AC (Adriamicina + Ciclofosfamida) son estándar para cáncer de mama HER2 negativo. Efectos secundarios incluyen neutropenia, fatiga y neuropatía.",
      "relevance_score": 0.87,
      "source": "Journal of Clinical Oncology 2024"
    },
    {
      "id": "doc_irrelevant_123",
      "title": "Recetas de Cocina Mediterránea",
      "content": "Este documento no tiene relación con medicina pero aparece por error en los resultados de búsqueda.",
      "relevance_score": 0.15,
      "source": "Irrelevant Database"
    }
  ],
  "total_found": 3,
  "search_time_ms": 456
}
```

**Error de búsqueda:**
```json
{
  "error": "SEARCH_ERROR",
  "message": "Error en la búsqueda de la base de conocimiento"
}
```

## Análisis de Interacciones Medicamentosas

**Endpoint:** `POST /ai/drug-interactions`

```bash
curl -X POST https://cancer-care-ai-api.onrender.com/ai/drug-interactions \
  -H "Content-Type: application/json" \
  -d '{
    "current_medications": ["warfarina", "omeprazol"],
    "new_medication": "aspirina"
  }'
```

**Campos requeridos:**
- `current_medications` - Lista de medicamentos actuales (requerido)
- `new_medication` - Nuevo medicamento a evaluar (requerido)

**Respuesta exitosa:**
```json
{
  "interaction_analysis": "Análisis de interacciones medicamentosas:\n\nSe identificó interacción MAYOR entre warfarina y aspirina. Ambos medicamentos afectan la coagulación sanguínea y su uso conjunto puede aumentar significativamente el riesgo de sangrado.\n\nRecomendaciones:\n- Monitoreo estricto de INR\n- Evaluación de riesgo/beneficio\n- Considerar alternativas a la aspirina\n- Educación al paciente sobre signos de sangrado",
  "current_medications": ["warfarina", "omeprazol"],
  "new_medication": "aspirina",
  "risk_level": "high",
  "confidence": 0.94,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Generación de Planes de Tratamiento

**Endpoint:** `POST /ai/treatment-plan`

```bash
curl -X POST https://cancer-care-ai-api.onrender.com/ai/treatment-plan \
  -H "Content-Type: application/json" \
  -d '{
    "patient_info": {
      "age": 45,
      "gender": "female",
      "diagnosis": "cáncer de mama",
      "stage": "IIA",
      "comorbidities": "diabetes tipo 2"
    }
  }'
```

**Campos requeridos en patient_info:**
- `age` - Edad del paciente (requerido)
- `gender` - Género del paciente (requerido)
- `diagnosis` - Diagnóstico oncológico (requerido)
- `stage` - Estadio del cáncer (requerido)
- `comorbidities` - Comorbilidades del paciente (opcional)

**Respuesta exitosa:**
```json
{
  "treatment_plan": "Plan de tratamiento oncológico sugerido:\n\nFASE 1 (Meses 1-3):\n- Quimioterapia neoadyuvante: AC x 4 ciclos\n- Evaluación de respuesta con imagenología\n- Monitoreo de toxicidades\n- Control de diabetes durante tratamiento\n\nFASE 2 (Meses 4-6):\n- Evaluación para cirugía\n- Terapia adyuvante según respuesta\n- Cuidados de soporte\n\nSEGUIMIENTO:\n- Consultas cada 3 meses año 1\n- Imagenología según protocolo institucional\n- Monitoreo glucémico estrecho\n\nNOTA: Este plan debe ser individualizado por equipo multidisciplinario considerando la diabetes como comorbilidad importante.",
  "patient_info": {
    "age": 45,
    "gender": "female",
    "diagnosis": "cáncer de mama",
    "stage": "IIA",
    "comorbidities": "diabetes tipo 2"
  },
  "rag_sources": [
    "Protocolos de Quimioterapia en Cáncer de Mama",
    "Manejo de Toxicidades en Quimioterapia"
  ],
  "confidence": 0.89,
  "model": "gpt-4-turbo",
  "timestamp": "2024-01-15T10:30:00Z"
}
```

## Modelos Disponibles

**Endpoint:** `GET /ai/models`

```bash
curl https://cancer-care-ai-api.onrender.com/ai/models
```

**Respuesta:**
```json
{
  "models": [
    {
      "id": "gpt-4-turbo",
      "name": "GPT-4 Turbo",
      "context_length": 4096,
      "status": "available"
    },
    {
      "id": "gpt-3.5-turbo-medical",
      "name": "GPT-3.5 Turbo Medical",
      "context_length": 2048,
      "status": "maintenance"
    },
    {
      "id": "claude-medical",
      "name": "Claude Medical",
      "context_length": 8192,
      "status": "rate_limited"
    }
  ],
  "default_model": "gpt-4-turbo"
}
```

## Estadísticas de Uso

**Endpoint:** `GET /ai/usage`

```bash
curl https://cancer-care-ai-api.onrender.com/ai/usage
```

**Respuesta:**
```json
{
  "tokens_used_today": 89456,
  "tokens_limit": 200000,
  "requests_today": 1234,
  "requests_limit": 2000,
  "rate_limit_per_minute": 20,
  "current_usage": 8,
  "timestamp": "2024-01-15T10:30:00Z"
}
```

---

# 🏥 Health Checks de Todas las APIs

## FHIR Health Check
```bash
curl https://cancer-care-fhir-api.onrender.com/health
```

## HL7 Health Check
```bash
curl https://cancer-care-hl7-stream.onrender.com/health
```

## FDA Health Check
```bash
curl https://cancer-care-fda-api.onrender.com/health
```

## AI Health Check
```bash
curl https://cancer-care-ai-api.onrender.com/health
```

---

**⚠️ RECORDATORIO IMPORTANTE:** 

Estas APIs simulan problemas reales de sistemas de healthcare. Los errores, inconsistencias y limitaciones son **intencionales** y forman parte del desafío técnico. 

**La meta es construir un sistema robusto que funcione a pesar de estas limitaciones, no "arreglar" las APIs.**

**¡Buena suerte integrando estos sistemas complejos! 🚀**