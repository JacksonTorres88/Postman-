Pruebas de API con Postman + Newman + GitHub Actions

Colección de pruebas de la API de Rick & Morty creada en Postman, ejecutada automáticamente con Newman (CLI) y GitHub Actions.
Incluye casos positivos (200 + validación de JSON + tiempo) y negativos (404).

💡 Objetivo: poder correr todo manualmente en tu compu y que GitHub lo ejecute en cada push/PR, dejando un reporte HTML como evidencia.

📁 Estructura del repositorio
.
├─ tests/
│  ├─ rickandmorty_collection.json        # Colección exportada de Postman
│  └─ rickandmorty_environment.json       # Environment exportado de Postman
└─ .github/
   └─ workflows/
      └─ api-tests.yml                    # Workflow de GitHub Actions


(Si cambiás los nombres, acordate de actualizar las rutas en el YAML).

🧰 Requisitos (local)

Node.js
 18+ (recomendado 20)

Newman y reporter HTML extra

npm install -g newman newman-reporter-htmlextra

▶️ Ejecutar localmente

Parate en la carpeta donde están los JSON y corré:

newman run tests/rickandmorty_collection.json \
  -e tests/rickandmorty_environment.json \
  -r cli,htmlextra \
  --reporter-htmlextra-export ./ReporteRegresion.html


En Windows (PowerShell/CMD) es igual.
Se genera ReporteRegresion.html; abrilo en el navegador.

(Opcional) CSV
npm install -g newman-reporter-csv
newman run tests/rickandmorty_collection.json -e tests/rickandmorty_environment.json \
  -r csv --reporter-csv-export ./ReporteRegresion.csv

(Opcional) Script .bat (doble clic)

Crea regresion.bat:

@echo off
cd %~dp0
newman run tests\rickandmorty_collection.json -e tests\rickandmorty_environment.json -r htmlextra --reporter-htmlextra-export ReporteRegresion.html
start "" "ReporteRegresion.html"
pause

🤖 CI/CD con GitHub Actions (ya configurado)

El workflow está en /.github/workflows/api-tests.yml.

¿Cuándo corre?

En cada push a main

En cada pull request

Manualmente desde Actions → API Tests (Newman) → Run workflow

¿Qué hace?

Instala Node, Newman y el reporter.

Ejecuta la colección con el environment.

Genera un reporte HTML.

Sube el HTML como Artifact.

¿Dónde veo el reporte?

Pestaña Actions → elegí el run → abajo en Artifacts descarga newman-report → adentro está newman-report.html.

⚠️ Si un test falla, el job queda en rojo (comportamiento esperado). El HTML igual se adjunta.

🔁 Actualizar colección / environment

En Postman: Export la colección (v2.1) y el environment.

Subilos reemplazando:

tests/rickandmorty_collection.json

tests/rickandmorty_environment.json

Hacé commit → GitHub Actions corre solo.

🔐 Variables y secretos (si tu API los necesita)

Guardá tokens/keys en Settings → Secrets and variables → Actions.

En el environment usá {{API_TOKEN}} y en un Pre-request Script podés mapear:

pm.environment.set('API_TOKEN', pm.variables.get('API_TOKEN'));


En el YAML, inyectá variables si hace falta:

env:
  API_TOKEN: ${{ secrets.API_TOKEN }}

🧪 Ejecutar sólo un folder (opcional)

Si tu colección tiene carpetas y querés correr una en particular:

newman run tests/rickandmorty_collection.json \
  -e tests/rickandmorty_environment.json \
  --folder "Characters Numerico positivo" \
  -r htmlextra --reporter-htmlextra-export ./ReporteRegresion.html

🛠️ Problemas comunes

could not find "htmlextra" reporter
Instalá el reporter: npm install -g newman-reporter-htmlextra.

ENOENT: no such file or directory
Revisá que las rutas/archivos del YAML existan:
tests/rickandmorty_collection.json y tests/rickandmorty_environment.json.

Job “rojo” en Actions
Significa que falló un test de Postman (por ejemplo, esperaba 200 y vino 404).
Abrí el Artifact newman-report.html para ver cuál fue.

(Si querés que el job no falle aunque haya errores —solo para demo— podés añadir continue-on-error: true al paso de ejecución en el YAML).

📝 Notas

Postman = crear y probar manual.

Newman = ejecutar automático (local, scripts, CI/CD).

El reporte HTML es tu evidencia para compartir.
