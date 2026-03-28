# Challenge 9 - Análisis de Actividad de Producto

Este proyecto tiene como objetivo analizar un dataset de actividad de productos (`product_activity.csv`) para extraer métricas confiables, limpiar inconsistencias y facilitar la toma de decisiones basada en datos reales. Sigue un enfoque de procesamiento de datos simple y eficiente, cumpliendo con los requerimientos mínimos del desafío.

## Propósito del Proyecto
El sistema permite:
- **Medir calidad de datos**: Identificar nulos, duplicados y errores lógicos en las fechas.
- **Normalizar información**: Estandarizar tipos de planes, categorías y dispositivos mediante diccionarios canónicos.
- **Calcular métricas de negocio**: Evaluar el engagement (votos), distribución de posts por país y concentración de actividad por usuario.
- **Separar errores**: Crear un reporte de "quarantine" para registros inconsistentes sin eliminarlos del flujo de auditoría.

## Estructura y Arquitectura
El proyecto sigue un flujo lineal de datos dentro de un Notebook de Jupyter:

```text
📁 Proyecto
├── 📁 docs/
│   ├── product_activity.csv       # Dataset original (entrada)
│   └── challenge.txt               # Especificaciones del reto
├── challenge_9.ipynb              # Notebook principal con todo el código
├── clean_product_activity.csv     # Dataset limpio (salida)
├── quarantine_product_activity.csv # Registros con errores (salida)
├── metrics_summary.csv            # Tabla resumen de KPIs (salida)
└── README.md                      # Documentación del proyecto
```

**Arquitectura:**
1. **Ingesta**: Lectura de CSV desde `docs/`.
2. **Procesamiento**: Limpieza, mapeo de diccionarios y recálculo de campos derivados.
3. **Validación**: Filtrado hacia `df_core` (limpio) o `df_quarantine` (errores).
4. **Análisis**: Generación de estadísticas descriptivas y conclusiones de producto.
5. **Persistencia**: Exportación de resultados a archivos CSV.

## Requisitos y Ejecución

### 1. Configuración del Entorno Virtual (Recomendado)
Para mantener las dependencias aisladas, crea y activa un entorno virtual en la raíz del proyecto:

- **Windows:**
  ```bash
  python -m venv .venv
  .\.venv\Scripts\activate
  ```
- **macOS/Linux:**
  ```bash
  python3 -m venv .venv
  source .venv/bin/activate
  ```

### 2. Instalación de Librerías
Una vez activado el entorno, instala las librerías necesarias:

```bash
pip install pandas numpy
```

### 3. Cómo ejecutar el proyecto
1. Asegúrate de tener el archivo `product_activity.csv` dentro de la carpeta `docs/`.
2. Abre el archivo `challenge_9.ipynb` en VS Code o cualquier entorno compatible con Jupyter con el kernel de tu entorno virtual.
3. Ejecuta todas las celdas de forma secuencial (de arriba hacia abajo).
4. Al finalizar, se generarán automáticamente los tres archivos de salida (.csv) en la raíz del proyecto.

---
**Desarrollado por: Edgar Vega - Da21nny - Software Developer**
