# Trabajo Práctico Integrador - Grupo G03
###### **UTN FRLP - Comunicación de Datos S31**
Este repositorio contiene la implementación de una aplicación web interactiva diseñada para simular y comprender los conceptos fundamentales de la **digitalización de imágenes**. A través de procesos de muestreo y cuantización, el software demuestra cómo se transforman señales bidimensionales analógicas (simuladas) en datos digitales aptos para su almacenamiento y transmisión. Nuestro grupo se compone como:

| Integrante               |
| ------------------------ |
| Eleno, Lautaro           |
| Llanos Boscato, Jerónimo |
| Sander, Martín           |
## Problema planteado

En el marco de la teoría de la información y la comunicación de datos, la digitalización es el proceso de convertir una señal analógica continua en una representación digital discreta. El desafío planteado por la cátedra para la **Propuesta 3 (Digitalización de Imágenes)** consiste en desarrollar una herramienta que tome una imagen y simule de manera didáctica las siguientes transformaciones:
- **Carga de imágenes en alta resolución**.
    
- **Proceso de Muestreo:** Reducción y alteración de las dimensiones espaciales de la imagen en píxeles (ej: $100\times100$, $500\times500$, $1000\times1000$).
    
- **Proceso de Cuantización:** Reducción de la profundidad de bits por canal de color (ej: escala de grises, color de 8 bits o color real de 24 bits).
    
- **Visualización y Comparación:** Contraste directo entre la señal original vs. la digitalizada.
    
- **Compresión:** Aplicar algoritmos de compresión para evaluar la reducción del peso del archivo resultante.
## Soluciones propuestas

La aplicación web construida en un único archivo modular (`index.html`) resuelve de forma interactiva y visual los requerimientos del enunciado:
- **Carga Dinámica Multiformato:** Permite subir archivos PNG, JPG y WEBP mediante arrastrar y soltar (Drag & Drop) o exploración local.
    
- **Muestreo configurable:** El usuario puede seleccionar diferentes resoluciones de salida (como $400\times400$, $800\times800$, $1024\times1024$ o $1920\times1080$ píxeles) para observar el fenómeno de pixelado y pérdida de detalle espacial.
    
- **Simulación de Compresión y Cuantización:** Mediante la API de Canvas, la imagen se procesa y comprime en formato JPEG con una pérdida controlada de calidad (70%), emulando la reducción de bits por pixel y la compresión con pérdida.
    
- **Análisis Cuantitativo en Tiempo Real:** Calcula de forma automatizada:
    
    - El tamaño del archivo original en Megabytes (MB).
        
    - El tamaño estimado del archivo digitalizado/comprimido en MB.
        
    - El **porcentaje (%) exacto de reducción** de datos obtenido tras la digitalización.
        
- **Exportación y Descarga:** Permite guardar localmente la imagen resultante ya digitalizada para verificar el resultado fuera del navegador.
## Funcionamiento Técnico

A continuación explicamos el desarrollo de la aplicación, que aprovecha las capacidades nativas del navegador mediante **React** (para la reactividad y manejo del estado) y la **HTML5 Canvas API** (para el procesamiento gráfico en tiempo real).

1. Captura del archivo y muestreo inicial

Cuando el usuario sube un archivo, el método `loadImage(file)` captura el objeto `File`:

- Registra su tamaño en bytes (`file.size`) que conformará el `originalSize`.
    
- Genera una URL en memoria usando `URL.createObjectURL(file)` para renderizar la imagen original de manera instantánea sin subirla a ningún servidor.

2. Procesamiento de la Imagen (Muestreo y Cuantización)

Al presionar el botón **"Comprimir Imagen"**, se ejecuta la función `handleCompress()`. La lógica interna realiza los siguientes pasos:

```js
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
const img = new Image();

img.onload = () => {
  // 1. Re-muestreo (Mapeo a la resolución seleccionada)
  canvas.width = selectedResolution.width;
  canvas.height = selectedResolution.height;
  
  // 2. Dibujado en el lienzo (Interpolación bilineal nativa)
  ctx.drawImage(img, 0, 0, canvas.width, canvas.height);

  // 3. Cuantización y Compresión (Conversión a JPEG con factor de calidad 0.7)
  const dataUrl = canvas.toDataURL('image/jpeg', 0.7);
  const base64str = dataUrl.split(',')[1];
  const decodedLength = atob(base64str).length; // Tamaño real en bytes

  setCompressedImage(dataUrl);
  setCompressedSize(decodedLength);
};
img.src = originalImage;
```

- **Muestreo:** Al asignar `canvas.width` y `canvas.height` con las propiedades de `selectedResolution`, se fuerza a la imagen de entrada a redimensionarse a una grilla discreta de píxeles.
    
- **Cuantización y Compresión:** `canvas.toDataURL('image/jpeg', 0.7)` exporta el mapa de bits del canvas a un formato JPEG. El parámetro `0.7` (70% de calidad) reduce la profundidad cromática efectiva y remueve frecuencias espaciales altas redundantes (compresión por transformada de coseno discreta cuantizada).
    
- **Cálculo del Peso Resultante:** Al ser una cadena en formato DataURL (Base64), el tamaño del archivo se incrementaría artificialmente por la codificación. Para obtener el peso físico real, se decodifica el segmento Base64 usando `atob()` para medir su longitud real en bytes discretos.

3. Visualización y métricas de eficiencia

El estado de React se actualiza reactivamente, recalculando las observaciones matemáticas expuestas en el footer:

- **Tasa de reducción:** Se calcula mediante la fórmula:

$$\text{Reducción} = \left( 1 - \frac{\text{Tamaño Comprimido}}{\text{Tamaño Original}} \right) \times 100$$
- **Formatos:** Transforma los bytes puros a formato legible en Megabytes (MB) dividiendo de manera sucesiva por $1024$.

## Tecnologías utilizadas

Para garantizar un entorno portable y de fácil ejecución por la cátedra sin requerir procesos complejos de instalación o compilación, se optó por:

- **HTML5 & CSS3:** Diseño responsivo con CSS Grid y Variables CSS para una interfaz moderna "Dark Mode".
    
- **React 18 (Vía CDN):** Gestión del flujo de datos, estados reactivos y renderizado del DOM de forma declarativa.
    
- **Babel Standalone (Vía CDN):** Compilación en tiempo de ejecución del código JSX en el navegador.
    
- **Canvas API:** Procesador nativo de la GPU del navegador para manipulación de píxeles.
