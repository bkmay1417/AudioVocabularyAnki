# AudioVocabularyAnki

<div style="display: flex; align-items: center;"> 
    
<img src="https://img.shields.io/badge/STATUS-EN%20DESAROLLO-green" />
    
</div>

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

## Crear Tarjetas de Anki
El siguiente script crea un mazo de Anki con las palabras y los archivos de audio generados, usando el paquete genanki.
### Ejecución
- Asegúrate de que el archivo series.txt esté en el mismo directorio que los scripts.
- Ejecuta el script de procesamiento del archivo de texto para generar translations.txt.
- Ejecuta el script de generación de archivos de audio para crear los archivos MP3 en la carpeta audio_files.

```python
import pandas as pd
import genanki
import os

# Leer el archivo de Excel
df = pd.read_excel('flashcards.xlsx')

# Crear un modelo para las tarjetas con audio y estilo mejorado
model = genanki.Model(
    1607392319,
    'Elegant Model with Audio',
    fields=[
        {'name': 'Español'},
        {'name': 'Inglés'},
        {'name': 'Audio'},
    ],
    templates=[
        {
            'name': 'Card 1',
            'qfmt': '''
                <div class="card-container">
                    <div class="card">
                        <div class="content">
                            <h2>{{Español}}</h2>
                        </div>
                    </div>
                </div>
            ''',
            'afmt': '''
                <div class="card-container">
                    <div class="card">
                        <div class="content">
                            <h2>{{Inglés}}</h2>
                            <div class="audio-container">
                                {{Audio}}
                            </div>
                        </div>
                    </div>
                </div>
            ''',
        },
    ],
    css="""
    .card-container {
        display: flex;
        justify-content: center;
        align-items: center;
        height: 100vh;
        background-color: #222;
        padding: 20px;
    }
    .card {
        font-family: Arial, Helvetica, sans-serif;
        color: white;
        background-color: #333;
        border: 2px solid #555;
        border-radius: 8px;
        padding: 3vw;
        max-width: 600px;
        width: 80%;
        box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
        transition: opacity 0.5s ease-in-out;
        text-align: center;
    }
    .content h2 {
        font-size: 5vw;
        margin: 0;
    }
    .audio-container {
        margin-top: 2vw;
    }
    .audio-container audio {
        width: 100%;
        max-width: 300px;
        display: block;
        margin: 0 auto;
    }
    .card:hover {
        opacity: 0.9;
    }
    """,
)

# Crear el mazo
deck = genanki.Deck(
    2059400110,
    'Mi Mazo Elegante con Audio')

# Añadir las tarjetas al mazo
for index, row in df.iterrows():
    audio_file = f"{row['Palabra en inglés']}.mp3"
    audio_path = f"audio_files/{audio_file}"
    
    if os.path.exists(audio_path):
        note = genanki.Note(
            model=model,
            fields=[row['Traducción en español'], row['Palabra en inglés'], f"[sound:{audio_file}]"]
        )
        deck.add_note(note)

# Crear el paquete y añadir los archivos de audio
package = genanki.Package(deck)
package.media_files = [f"audio_files/{row['Palabra en inglés']}.mp3" for index, row in df.iterrows()]

# Guardar el mazo en un archivo .apkg
package.write_to_file('mi_mazo_elegante_con_audio.apkg')

print("El mazo de Anki ha sido creado exitosamente.")

```

## Notas
El archivo series.txt debe contener traducciones en formato de tabulaciones.
El script ignora las líneas que comienzan con #.
Asegúrate de que los nombres de los archivos de audio generados sean válidos en tu sistema operativo.
Ejecuta el script de creación de tarjetas de Anki para generar el mazo de Anki.

## Desarrolladores

| [<img src="https://avatars.githubusercontent.com/u/163685041?v=4" width=115><br><sub>Michael Martinez</sub>](https://github.com/bkmay1417) |
| :---: |

Copyright (c) 2024 [Michael Martinez] yam8991@gmail.com
