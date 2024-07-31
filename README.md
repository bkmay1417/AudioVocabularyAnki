# AudioVocabularyAnki

Este proyecto procesa un archivo de texto con traducciones, extrae las traducciones, las guarda en un archivo de texto y luego genera archivos de audio para cada traducción utilizando la API de Google Text-to-Speech (gTTS).

## Requisitos

- Python 3.x
- Pandas
- gTTS

Puedes instalar los paquetes necesarios ejecutando:
```sh
pip install pandas gtts
```

## Archivos
- series.txt: Archivo de texto que contiene las traducciones.
- translations.txt: Archivo generado que contiene solo la columna de traducciones.
- audio_files/: Carpeta generada que contiene los archivos de audio en formato MP3.
- 
## Uso
Procesar el Archivo de Texto
El script procesa el archivo series.txt y extrae las traducciones, guardándolas en translations.txt.

```python
import pandas as pd
import re

# Ruta al archivo TXT
file_path = 'series.txt'

def clean_line(line):
    """
    Limpiar y dividir una línea en campos.
    """
    # Reemplaza múltiples tabulaciones con una sola tabulación
    line = re.sub(r'\t+', '\t', line.strip())
    # Dividir por tabulación
    fields = line.split('\t')
    # Limpiar espacios adicionales
    fields = [field.strip() for field in fields]
    return fields

def process_file(file_path):
    """
    Procesar el archivo y devolver un DataFrame.
    """
    with open(file_path, 'r', encoding='utf-8') as file:
        lines = file.readlines()
    
    # Limpiar líneas
    cleaned_lines = [clean_line(line) for line in lines if not line.startswith("#")]
    
    # Procesar datos
    data = []
    for fields in cleaned_lines:
        # Añadir campos vacíos si hay menos de 3 columnas
        while len(fields) < 3:
            fields.append('')
        
        # Solo conservar las primeras 3 columnas
        data.append(fields[:3])
    
    # Crear DataFrame
    df = pd.DataFrame(data, columns=['English', 'Translation', 'Notes'])
    
    return df

# Procesar el archivo y obtener el DataFrame
try:
    df = process_file(file_path)
    
    # Extraer la columna 'Translation'
    translations = df['Translation']
    
    # Guardar la columna en un archivo de texto
    output_file = 'translations.txt'
    with open(output_file, 'w', encoding='utf-8') as file:
        for translation in translations:
            file.write(translation + '\n')
    
    print(f"Columna 'Translation' guardada en '{output_file}'")
except Exception as e:
    print(f"Error al procesar el archivo: {e}")

```

## Generar Archivos de Audio
El siguiente script lee translations.txt y genera un archivo de audio para cada traducción en la carpeta audio_files.

```python
from gtts import gTTS
import os
import re

def clean_filename(filename):
    # Reemplaza caracteres no válidos con guiones bajos
    return re.sub(r'[<>:"/\\|?*\x00-\x1F]', '_', filename)

# Lee el archivo translations.txt
with open('translations.txt', 'r') as file:
    words = file.readlines()

# Crea una carpeta para los audios
if not os.path.exists('audio_files'):
    os.makedirs('audio_files')

for word in words:
    word = word.strip()
    if word:  # Si la línea no está vacía
        # Limpia el nombre del archivo
        safe_word = clean_filename(word)
        # Genera el archivo de audio
        tts = gTTS(text=word, lang='en')
        filename = os.path.join('audio_files', f'{safe_word}.mp3')
        tts.save(filename)
        print(f'Archivo de audio generado para: {word}')


```
## Ejecución
- Asegúrate de que el archivo series.txt esté en el mismo directorio que los scripts.
- Ejecuta el script de procesamiento del archivo de texto para generar translations.txt.
- Ejecuta el script de generación de archivos de audio para crear los archivos MP3 en la carpeta audio_files.

## Notas
El archivo series.txt debe contener traducciones en formato de tabulaciones.
El script ignora las líneas que comienzan con #.
Asegúrate de que los nombres de los archivos de audio generados sean válidos en tu sistema operativo.

## Desarrolladores

| [<img src="https://avatars.githubusercontent.com/u/163685041?v=4" width=115><br><sub>Michael Martinez</sub>](https://github.com/bkmay1417) |
| :---: |

Copyright (c) 2024 [Michael Martinez] yam8991@gmail.com
