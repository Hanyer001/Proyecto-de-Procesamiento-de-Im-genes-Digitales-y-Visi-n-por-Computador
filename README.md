# Proyecto Final: Monitoreo de EPP y Detección de Infracciones de Seguridad

**Facultad de Ingeniería - Universidad Alberto Hurtado**
**Asignatura: Procesamiento de Imágenes Digitales y Visión por Computador
**Equipo:** [Tus Nombres/Apellidos Aquí]

## Enlaces del Proyecto
* **Video 1 (Demostración de los 3 detectores):** [https://youtu.be/HCQj_n97s-8]
* **Video 2 (Justificación de despliegue - 3 min):** [https://youtu.be/XaC0UP39Vbw]

---

## 1. Contexto y Misión
Este proyecto desarrolla y evalúa un sistema de monitoreo de seguridad por video para una empresa constructora. El objetivo principal es vigilar el uso de equipo de protección personal (EPP) mediante las cámaras de la obra, enfocándose críticamente en detectar a los trabajadores que **no** llevan casco de seguridad ("NO-Hardhat"). 

Para lograrlo, se evaluaron tres paradigmas tecnológicos distintos sobre el dataset **CSS-Data** para determinar la mejor solución a desplegar en el entorno de producción. Debido a las limitaciones de memoria de la GPU T4 utilizada, el video de monitoreo fue muestreado a ~1 FPS para permitir la evaluación justa de los modelos de lenguaje visual (VLM) generativos, los cuales no pueden operar a 30 FPS en este hardware.

## 2. Los Tres Frameworks Evaluados
Se midieron los siguientes modelos sobre los mismos frames exactos del conjunto de prueba:

1. **Paradigma Cerrado (YOLOv8n):** Modelo afinado (fine-tuning) sobre las clases de EPP. Su salida entrega cajas con clase y nivel de confianza.
2. **Vocabulario Abierto (YOLO-World):** Modelo ejecutado mediante inferencia zero-shot configurando la búsqueda de clases mediante texto (`set_classes(["hard hat", "head", "person"])`).
3. **Modelos de Lenguaje Visual (VLM):**
   * **Florence-2 (Grounding):** Ejecutado mediante la tarea `<OPEN_VOCABULARY_DETECTION>` para obtener cajas delimitadoras a partir de texto.
   * **Moondream2 (Generativo):** Interrogado en lenguaje natural con la pregunta "¿Hay alguien en la imagen que no lleve casco de seguridad?" para obtener una respuesta binaria a nivel de evento.

---

## 3. Cuadro Comparativo de Métricas

La siguiente tabla resume el rendimiento de los tres paradigmas ejecutados bajo el mismo entorno de hardware (GPU T4):

| Paradigma | Modelo | Precisión Caja (mAP@0.5) | Precisión Evento ($F_{1}$) | Velocidad (Inferencia) | Flexibilidad / Robustez |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Cerrado** | YOLOv8n | [0.XX] | [0.XX] | ~18.6 ms (>30 FPS) | Nula. Ciego ante la clase no vista ("lentes de seguridad"). |
| **Abierto** | YOLO-World | [0.XX] | [0.XX] | ~77.6 ms | Alta (Zero-shot). Detecta nuevas clases sin reentrenar. |
| **VLM** | Moondream2 / Florence-2 | N/A | [0.XX] | [X.XX] s (<1-2 FPS) | Sensible al prompt. Cambia su respuesta al variar la redacción. |

*Nota sobre las métricas:* 
* El cálculo de la intersección sobre unión a nivel de caja se rige por $IoU=|A\cap B|/|A\cup B|$.
* El nivel-evento se calcula evaluando la pregunta binaria de infracción por frame utilizando $F_{1}=2~TP/(2~TP+FP+FN)$.

---

## 4. Análisis Crítico y Hallazgos

### El Hallazgo Pedagógico: La Negación
Durante el desarrollo del monitoreo, se evidenció que la pregunta más crítica en seguridad industrial es una negación ("¿hay alguien sin casco?"). Mientras que los modelos de vocabulario abierto y los VLM poseen un mayor nivel de "inteligencia" y razonamiento general, tropiezan severamente con la negación y los atributos. Por el contrario, el detector cerrado resuelve este problema de manera confiable únicamente porque se entrenó con una clase explícita `NO-Hardhat`.

### Licencias y Costos de Despliegue Comercial
Se debe tener en consideración el licenciamiento en caso de que este sistema se escale a un producto comercial:
* **YOLOv8 / Ultralytics:** AGPL-3.0
* **YOLO-World:** GPL-3.0
* **Florence-2:** MIT
* **Moondream2:** Apache-2.0
* **CSS-Data:** CC BY 4.0

### Sensibilidad de los VLM
El modelo VLM (Moondream2) demostró varianza en su robustez. Al variar la redacción del prompt buscando la misma infracción de seguridad (ej: "¿Todos los trabajadores están usando casco?" vs. "Identifica si alguna persona olvidó ponerse el casco"), el modelo arrojó inconsistencias en sus predicciones, validando que la métrica de nivel-evento ($F_{1}$) es la comparación justa para aislar las debilidades del VLM en una tarea de monitoreo estricta.

---

## 5. La Decisión Final: ¿Qué detector desplegar y por qué?

Basado en la evidencia recolectada, **nuestra recomendación es desplegar en producción el modelo de paradigma cerrado (YOLOv8n)**. 

Esta decisión se sustenta en tres pilares fundamentales obtenidos de nuestros propios números:

1. **Velocidad Crítica:** Una cámara de monitoreo de obra opera en tiempo real (típicamente a 30 FPS). YOLOv8n demostró un tiempo de inferencia de apenas ~18.6ms por frame en el hardware T4 evaluado. Los modelos VLM, que requieren muestrear el video a ~1 FPS para no colapsar la memoria, introducirían puntos ciegos temporales inaceptables en un entorno de alto riesgo.
2. **Efectividad ante la Negación:** En seguridad laboral, un falso negativo (no alertar sobre un trabajador sin casco) puede resultar en accidentes fatales o multas severas para la constructora. Al tener la clase `NO-Hardhat` entrenada explícitamente, YOLOv8 supera la incapacidad estructural que tienen YOLO-World y los VLM para entender eficientemente el concepto de negación de un atributo de seguridad.
