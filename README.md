# Prueba técnica GERPRO - Django API

API Django que calcula:
- Desembolsos de crédito constructor
- Aportes de capital

con base en movimientos (ingresos y costos por periodo) y parámetros del crédito.

Incluye:
- Lógica en Python (services.py)
- Modelos Django (parte 2)
- Endpoint REST listo para usar
- Dockerfile y Procfile para despliegue

## Requisitos
- Python 3.11+
- pip
- (Opcional) Docker

## Instalación local

```bash
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python manage.py migrate
python manage.py runserver
```

Endpoint:
- POST http://localhost:8000/api/financiar/

## Ejemplo de uso

Body de ejemplo:

```json
{
  "movimientos": [
    {"subetapa":"Torre 1","valor":1000,"periodo":1,"concepto":"ingresos"},
    {"subetapa":"Torre 1","valor":1000,"periodo":2,"concepto":"ingresos"}
    // ... resto del JSON
  ],
  "cupo_credito": 7000,
  "max_pct_mensual": 0.08,
  "periodo_inicio_credito": 9,
  "periodo_fin_credito": 23,
  "tasa_interes_anual": 0.12
}
```

Notas de la lógica:
- El crédito solo cubre FCO negativo (ingresos - costos). No cubre intereses ni pago de capital.
- Intereses se generan en el mismo mes sobre el saldo al cierre y se pagan al mes siguiente.
- El capital se paga en los últimos 2 periodos con ingresos (mitad y mitad).
- Tope de desembolso: `min(necesidad operativa, cupo_restante, cupo * % mensual)`.

La respuesta incluye:
- `desembolsos`
- `aportes`
- `breakdown` con FCO, FCN, intereses, pagos de capital, saldos.

## Docker

```bash
docker build -t gerpro-api .
docker run -p 8000:8000 gerpro-api
```

## Despliegue

- Render:
  - New Web Service -> Conecta el repo
  - Runtime: Docker
  - Env vars: DJANGO_SECRET_KEY, DEBUG=false, ALLOWED_HOSTS=*
  - Start command: (usa el CMD del Dockerfile)

- Railway:
  - New Project from GitHub -> Repo
  - Autodetecta Dockerfile
  - Env vars como arriba

- Heroku:
  - Conecta el repo
  - Procfile listo
  - Set env vars
  - Tras el primer deploy, corre migraciones:
    - `heroku run python manage.py migrate`

## Modelos (parte 2)
- Project, Subetapa, Movement (ingresos/costos por periodo)
- Credit (cupo, % mensual, periodo inicio/fin, tasa anual)
- CreditDisbursement, CapitalContribution, InterestPayment, PrincipalPayment