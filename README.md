# Análisis y procesamiento de señales de audio: Problema del cóctel
## Contextualización
El problema del cóctel, también conocido como el "cocktail party problem" [^1^], se relaciona con la capacidad de separar y entender múltiples fuentes de sonido que se superponen en una grabación.
La situación se presenta  en una fiesta con muchas personas hablando al mismo tiempo; el desafío es identificar y comprender la voz de una persona específica a pesar del ruido y las voces de fondo.

Este proyecto tiene como objetivo abordar el problema del cóctel de voces mediante la aplicación de técnicas de procesamiento de señales de audio. El proyecto se centra en tres grabaciones de audio, cada una conteniendo tres voces diferentes que dicen cosas distintas. 
Se realizó el trabajo con un enfoque en la separación de señales, utilizando técnicas avanzadas para dividir las voces individuales presentes en las grabaciones mezcladas, mediante el Análisis de Componentes Independientes (ICA).

También se realizó un análisis espectral y temporal, utilizando herramientas de transformada de Fourier y espectrogramas; proporcionado un panorama de las características de frecuencia y tiempo de las voces, permitiendo una comprensión más profunda de su contenido.

Del mismo modo, se calculó la relación señal-ruido (SNR) para evaluar la calidad de las grabaciones y la efectividad de las técnicas de procesamiento aplicadas, lo que ayuda a cuantificar la mejora en la separación de las señales y la reducción del ruido.

## Recopilación de las señales
Es fundamental tener en cuenta el método de grabación para las señales de audio que serán utilizadas en el proyecto. En primer lugar, se debe verificar que los tres sensores (micrófonos), estén grabando a la misma frecuencia. En este caso particular, se grabó con tres celulares marca samsung, ya que estos permiten elegir la calidad de grabación; se eligió 44.1kHz ya que brinda una calidad estándar y un equilibrio ideal entre tamaño de archivo y compatibilidad. 

![Configuración samsung](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/samsung.jpg?raw=true)
*Panel de ajustes de la grabadora de voz de samsung.(Tomada de uno de los dispositivos utilizados).*

Posteriormente se planificó la ubicación de los sensores con respecto a ellos mismos y a las fuentes de sonido (voces de las tres personas), esto resulta determinante ya que los resultados pueden verse afectados de acuerdo con factores como la acústica del espacio,
la intensidad de las voces, el ruido de fondo, entre otras. 
Idealmente, cada micrófono debe encontrarse a mínimo 1m de distancia, para reducir la interferencia entre las señales de cada sensor y asegurar que cada uno capture una mezcla diferente de las fuentes facilitando de algún modo el proceso de separación. En este caso, cada sensor se encontraba aproximadamente a 4.5m el uno del otro y a 0.5m de las fuentes de sonido, adicionalmente los sensores estaban orientados hacia las fuentes, alejándolos de los espacios donde no estaban las voces. De esta manera se evita la sobrecarga de los micrófonos, para no distorsionar la señal y se aseguraba la captura de la señal de manera más clara.

![Ubicación fuentes y sensores](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/ubicacion.jpg?raw=true)

*Ubicación de los micrófonos en el proceso de grabación.*

Inicialmente se hizo una grabación del ruido ambiente con cada uno de los micrófonos en su posición establecida, para proseguir con la grabación de las voces en esta misma posición. Lo anterior con el fin de poder evaluar la relación entre el ruido del espacio y las señales capturadas.

## Procesamiento inicial 
Una vez se obtuvieron los audios debieron pasar por una fase preliminar de eleccción y adecuación. Para empezar, la duración de las señales (tanto audios como ruido) no coincidia, por lo que se hizo uso de una aplicación online [^2^], que permite cortar los audios con  mayor precisión que las propias aplicaciones de grabación. 

![Cortador de audio](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/cortador.jpg?raw=true)
*Interfaz de la aplicación para cortar los audios (Tomado de: [^2^]).*

En consecuencia, la aplicación brinda un audio ajustado al tamaño deseado, cada audio y ruido fueron establecidos en la misam duración (1.38s). Sin embargo, la aplicación arroja los archivos en formato MP3, por lo que debieron ser convertidos a formato WAV para su correcto 
análisis en python. La conversión se hizo con otra página especializada. [^3^]

![Convertidor de audio](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/convertidor.jpg?raw=true)
*Interfaz de la aplicación para convertir los audios (Tomado de: [^3^]).*

## Carga de audio y ruido
Comenzando con el código, en la parte inicial se cargan los archivos de audio y ruido utilizando la biblioteca librosa. Las rutas a los archivos se definen en variables y se cargan en variables correspondientes.
Se debe asegurar que los audios a procesar se encuentren en formato .WAV y estén ubicados en una carpeta del computador.

```python
loc_audio1 = r'C:\Users\Valen\Downloads\V\Lab2Señales\Audios\Audio 1.wav'
loc_ruido1 = r'C:\Users\Valen\Downloads\V\Lab2Señales\Audios\Ruido 1.wav'

# Cargar audios y ruidos con librosa 
audio1, sr1 = librosa.load(loc_audio1, sr=None)
ruido1, sr1 = librosa.load(loc_ruido1, sr=None)
```
ibrosa.load carga los archivos de audio y ruido y devuelve el contenido del archivo de audio como un arreglo númerico. Asegurar la duración exacta de las señales -con el cortador de audios-, permite que los arreglos tengan la misma longitud y se puedan operar entre ellos.

## Funciones para el análisis y procesamiento 
### Graficar y reproducir
Se tiene una  función  llamada graficas se encarga de graficar las formas de onda del audio, el ruido y su combinación(para hallar SNR), así como reproducir estos archivos.
Se tiliza matplotlib para mostrar las formas de onda del audio y el ruido, mientras que para la reproducción de los audios se usa sounddevice. 
```python
def graficas(audio, ruido, sr, color, tituloA, tituloR, tituloC):
    
    # Graficar audio
    plt.figure(figsize=(16, 16))
    plt.subplot(3, 1, 1)
    plt.plot(np.linspace(0, len(audio) / sr, num=len(audio)), audio, color=color)
    plt.title(tituloA)
    plt.xlabel('Tiempo [s]')
    plt.ylabel('Amplitud')
    plt.grid(True)
    
    # Reproducir audio
    print(f'Se está reproduciendo {tituloA}...')
    sd.play(audio, sr)
    sd.wait()  

    # Graficar ruido
    plt.subplot(3, 1, 2)
    plt.plot(np.linspace(0, len(ruido) / sr, num=len(ruido)), ruido, color=color)
    plt.title(tituloR)
    plt.xlabel('Tiempo [s]')
    plt.ylabel('Amplitud')
    plt.grid(True)
    
    # Reproducir ruido
    print(f'Se está reproduciendo {tituloR}...')
    sd.play(ruido, sr)
    sd.wait()  
    
    # Audio con ruido
    # Asegurar que tanto el audio como el ruido tengan el mismo tamano
    if len(audio) > len(ruido):
        ruido = np.pad(ruido, (0, len(audio) - len(ruido)), 'constant')
    else:
        audio = np.pad(audio, (0, len(ruido) - len(audio)), 'constant')
        
    combinado = audio + ruido
    
    plt.subplot(3, 1, 3)
    plt.plot(np.linspace(0, len(combinado) / sr, num=len(combinado)), combinado, color=color)
    plt.title(tituloC)
    plt.xlabel('Tiempo [s]')
    plt.ylabel('Amplitud')
    plt.grid(True)
    
    return audio, ruido, combinado
```
#### Reproducción del audio 1: 

https://github.com/user-attachments/assets/c3807c1c-8337-438e-9706-80d9abd4e250

#### Reproducción del ruido 1: 

https://github.com/user-attachments/assets/33df3900-3ad9-48de-bca8-4a386b673378


 Se ajustaron las longitudes de las señales de audio y ruido, pues a pesar de haberlas recortado con precisión, algunas presentaban desfases respecto a las otras por milesimas de segundo, posteriormente se combinaron las dos señales, para observar la gráfica correspondiente.

![Gráficas 1](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/Graficas%20audio%201.png?raw=true)
*Gráfica de la señal 1, el ruido ambiente 1 y la suma de ambos realizada en python.*

![Gráficas 2](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/Graficas%20audio%202.png?raw=true)
*Gráfica de la señal 2, el ruido ambiente 2 y la suma de ambos realizada en python.*

![Gráficas 3](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/Graficas%20audio%203.png?raw=true)
*Gráfica de la señal 3, el ruido ambiente 3 y la suma de ambos realizada en python.*

### Calcular la relación señal-ruido
```python 
import math

def calcular_snr(audio, ruido):
    la = len(audio)
    lr = len(ruido)
    
    Pa = (sum(audio**2)) / la
    Pr = (sum(ruido**2)) / lr
    b = Pa / Pr
    return 10 * math.log10(b)
```
La relación señal-ruido se calcula como la razón entre la potencia de la señal de audio y la potencia del ruido. Esta nos permite evaluar la calidad de la señal con respecto a la cantidad de ruido que presenta. El SNR tiene valores ideales de acuerdo con las señales a
las que se lo este aplicando, sin embargo  debe tener un mínimo de +25dBm, para considerarse una señal "buena", puesto que los valores inferiores resultan en un mal desempeño.[^6^] Se cálcula como: 
* SNR=10⋅log_10(Pruido/Psenal)

La potencia se obtiene  de la siguiente manera: 

* P=(∑|X₁|^2 )/N

Se calcularon 3 SNR, uno para cada señal, tomando el ruido ambiente captado por el sensor que adquirió la misma señal.
1. SNR para el audio 1 y ruido 1 = 37,518
2. SNR para el audio 2 y ruido 2 = 42,018
3. SNR para el audio 3 y ruido 3 = 31,461

En general, los tres SNR son altos, lo que significa que la señal está  por encima del nivel de ruido. Esto implica una buena calidad del audio, ya que el ruido es poco perceptible en comparación con la señal.
El audio 2 tiene la mejor relación señal-ruido, seguido por el audio 1, y el audio 3 tiene la menor SNR, aunque sigue siendo un valor bueno.
Teniendo esto en cuenta, el audio 2 es la señal más limpia. 

### Análisis temporal y espectral 
Con el fin de comprender el comportamiento de las señales de audio en su totalidad se realizó un análisis temporal y espectral, que resulta indispensable para la calidad y procesamiento de la señal.    
El análisis temporal se enfoca en cómo las señales cambian con el tiempo, permitiendo entender su estructura básica, como los picos, valles, y cualquier cambio en la amplitud con el tiempo;  esto implica observar la forma de onda de la señal [^4^].
Por otro lado, el análisis espectral se centra en el contenido en frecuencia de la señal de audio. Utiliza técnicas como la Transformada de Fourier (FT) o la Transformada Rápida de Fourier (FFT) para descomponer la señal en sus componentes de frecuencia.
Se recomienda utilizar la transformada discreta de Fourier o la transformada rápida de Fourier para dicho análisis. [^5^]

* Transformada de Fourier Discreta (DFT): La DFT es una herramienta matemática que convierte una señal en el dominio del tiempo en una representación en el dominio de la frecuencia. Específicamente, descompone una señal en una serie de componentes de frecuencia, 
permitiendo analizar las frecuencias presentes en la señal y sus magnitudes. 

* Transformada Rápida de Fourier (FFT): La FFT es un algoritmo eficiente para calcular la DFT. La principal ventaja de la FFT sobre la DFT es su velocidad de cálculo. Mientras que la DFT es bastante compleja, lo que la hace mucho más adecuada para señales grandes.
La FFT aprovecha las propiedades simétricas y periódicas de la DFT para reducir significativamente el número de cálculos necesarios.

Para el proyecto se decidió utilizar la FFT debido a su velocidad, ya que permite analizar grandes cantidades de datos, además que permite un cálculo  más eficiente. Python contiene librerias específicas que permiten realizar las transformadas de Fourier de manera rápida y eficiente.
Se decidió utilizar librosa.stft, ya que es la misma libreria que permite cargar los audios. STFT es una función de la biblioteca que se encarga de calcular la Transformada de Fourier de Tiempo Corto (STFT) de una señal de audio.
Lo hace diviendo la señal  en segmentos de tiempo y mulplicandolos  por una ventana para minimizar discontinuidades en los bordes de cada sección. Luego, aplica la Transformada de Fourier a cada segmento de ventana conviertiendo cada una de ellas
en un espectro de frecuencias. La función Librosa.stft utiliza la FFT internamente para realizar estos cálculos de manera eficiente, por ejemplo, cada ventana de tiempo se somete a una DFT, realizada por medio de FFT para obtener las frecuencias.

A continuación se presenta el código que realiza un análisis temporal y espectral de señales de audio utilizando la STFT y visualiza tanto la forma de onda como el espectrograma; se hallaron también, las maginutudes, fases y frecuencias de la FFT, con el fin de poder interpretar de una manera más precisa el análisis espectral. 

```python 
def analisis_espectral(audio, sr, titulo):
    n_fft = 2048
    hop_length = 512

    S = librosa.stft(audio, n_fft=n_fft, hop_length=hop_length)
    S_db = librosa.amplitude_to_db(np.abs(S), ref=np.max)
    

    plt.figure(figsize=(12, 4))
    librosa.display.waveshow(audio, sr=sr, color='orange')
    plt.title(f'Forma de Onda del {titulo}')
    plt.xlabel('Tiempo [s]')
    plt.ylabel('Amplitud')
    plt.show()
    
    plt.figure(figsize=(12, 6))
    librosa.display.specshow(S_db, sr=sr, x_axis='time', y_axis='log', cmap='viridis')
    plt.colorbar(format='%+2.0f dB')
    plt.title(f'Espectrograma del {titulo}')
    plt.xlabel('Tiempo [s]')
    plt.ylabel('Frecuencia [Hz]')
    plt.show()

   # Obtener magnitudes y fases
    magnitudes = np.abs(S)
    fases = np.angle(S)
    
    # Obtener frecuencias y tiempos
    frecuencias = librosa.fft_frequencies(n_fft=n_fft)
    tiempos = librosa.frames_to_time(np.arange(S.shape[1]), sr=sr, hop_length=hop_length)
    
    # Estadísticas descriptivas
    print("Estadísticas de magnitudes:")
    print(f"Media: {np.mean(magnitudes)}")
    print(f"Desviación estándar: {np.std(magnitudes)}")
    print(f"Máximo: {np.max(magnitudes)}")
    print(f"Mínimo: {np.min(magnitudes)}")
    
    print("Estadísticas de frecuencias:")
    print(f"Media: {np.mean(frecuencias)}")
    print(f"Desviación estándar: {np.std(frecuencias)}")
    print(f"Máximo: {np.max(frecuencias)}")
    print(f"Mínimo: {np.min(frecuencias)}")
```
* .n_fft define el tamaño de la ventana para la STFT, y hop_length define el número de muestras entre ventanas consecutivas, afectando la resolución temporal del espectrograma.
* librosa.stft calcula la transformada de Fourier de corto tiempo de la señal, dividiendo la señal en segmentos de longitud n_fft y calculando la STFT de cada segmento.
* librosa.amplitude_to_db convierte la magnitud de la STFT a decibelios.
* librosa.display.waveshow se utiliza para visualizar la forma de onda de la señal de audio en el dominio del tiempo, mostrando cómo varía la amplitud a lo largo del tiempo.
* librosa.display.specshow se usa para visualizar el espectrograma. y_axis='log' presenta el eje de frecuencia en una escala logarítmica, lo que es útil para visualizar detalles en frecuencias bajas.

Se decidió presentar los datos de la STFT a través estadísticas descriptivas además del espectograma, ya que hacen que sea más fácil entender la distribución general de los datos sin tener que revisar cada valor individual.
Adicionalmente, brindan  los datos de manera clara y concisa, permitiendo identificar tendencias y características rápidamente.

#### Análisis de Resultados
#### Análisis espectral
Las magnitudes de la STFT representan la intensidad de las componentes de frecuencia en cada ventana de tiempo de la señal.
Las fases indican el desplazamiento de fase de las componentes de frecuencia,  pero no son tan directamente interpretables para análisis visual como las magnitudes.

Se aplico la función de análisis espectral y temporal  al audio 2, al ser la señal más limpia y ser la que se utilizará para el filtrado. Se obtuvo lo siguiente:

##### Reproducción del audio 2:

https://github.com/user-attachments/assets/4b96a16e-d2f4-4240-9d46-826d70ebfc71

##### Reproducción del ruido 2:

https://github.com/user-attachments/assets/9aa84ee9-dd27-4209-b59d-0ae0ed89dfed


1. Estadísticas descriptivas de las magnitudes:

* Media: La media de las magnitudes es 0.610, lo que indica que, en promedio, las magnitudes de la STFT son relativamente bajas. 
Sin embargo, la presencia de valores extremadamente altos  sugiere que hay picos significativos en ciertas frecuencias y tiempos.

* Desviación Estándar: La desviación estándar es 3.514, siendo bastante alta en comparación con la media. lo que indica que hay una gran variabilidad en las magnitudes. Esto es natural, si tenemos en cuenta las características de las señales donde las voces dominan por un corto período.

* Máximo : El valor máximo de 131.766 es mucho mayor que la media. Esto indica que hay componentes de frecuencia con una intensidad mucho mayor que otras, lo que señala eventos importantes en la señal.

* Mínimo : El valor mínimo de 0.000 muestra que hay frecuencias que no están presentes en la señal en ciertos momentos, y que las magnitudes pueden ser cero en esas áreas específicas.

2. Estadísticas descriptivas de las frecuencias:

* Media: La media de las frecuencias es 5512.5 Hz. Esto indica que, en promedio, las componentes de frecuencia están concentradas en la parte media del espectro de frecuencia analizado. Es decir, el contenido espectral de la señal tiene una tendencia a estar en frecuencias que están alrededor de 5500 Hz.

* Desviación Estándar: La desviación estándar de 3185.750 Hz, sugiere que hay una gran dispersión en las frecuencias presentes en la señal. La alta desviación estándar indica que la señal contiene una amplia gama de frecuencias, coincidiendo con lo mostrado anteriomente en las magnitudes.

* Máximo : El valor máximo de 11025 Hz indica que el rango de frecuencias analizado llega hasta un poco más de 11 kHz.. Este valor es relevante para entender el alcance de las frecuencias presentes en la señal y analizar las frceuencias de las fuentes. 

* Mínimo (0.0 Hz): El valor mínimo de 0.0 Hz representa el componente de frecuencia constante. 

3. Espectograma:

Un espectrograma es una representación visual que muestra cómo cambian las frecuencias de una señal a lo largo del tiempo. Consta de una imagen bidimensional que presenta la evolución de la frecuencia y la amplitud de la señal.
El eje horizontal (x) representa el tiempo, mientras que el eje vertical (y) representa la frecuencia.
El color o la intensidad en cada punto del espectrograma representa la magnitud de la señal en esa frecuencia y tiempo específico.

![Espectograma audio 2](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/espectograma%20audio%202Or.png?raw=true)

*Espectograma para el audio 2 realizado en python.*


En el eje X, el tiempo se extiende desde 0 hasta alrededor de 1.5 segundos, indicando la  duración del audio.
En el eje Y, se representan las frecuencias desde 0 Hz hasta aproximadamente 16,384 Hz. La mayor concentración de energía parece estar en las frecuencias entre 64 Hz y 512 Hz, lo que es típico de las frecuencias de las voces humanas.
En cuanto a la intensidad, la escala de colores va de azul oscuro, que representa las bajas intensidades (alrededor de -80 dB) a amarillo, que serían las altas intensidades, cerca de 0 dB.
Hay bandas claramente visibles entre 0.5 s y 1.2 s, que indican que la concentración de las frecuencias, lo que esta relacionado con las tres voces hablando simultáneamente.
Debido a las franjas amplias en el rango de 256 Hz a 512 Hz, se puede asumir que en estas areás se superpongan las voces.
Las variaciones a lo largo de las frecuencias confirman que las voces están pronunciando sonidos diferentes, ya que diferentes sonidos generan diferentes patrones de frecuencia.

#### Análisis temporal

La forma de onda refleja variaciones importantes en la energía temporal.

![Forma de onda audio 2](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/Forma%20de%20onda%20audio%202Or.png?raw=true)

*Forma de onda para el audio 2 realizado en python.*


En la forma de onda, la amplitud de la señal se representa en el eje vertical. Se evidencia que la amplitud varía considerablemente a lo largo del tiempo, alcanzando valores máximos cercanos a ±0.6 alrededor del intervalo entre 0.5 y 0.6 segundos. 
Esto indica que, en esos momentos, al menos una de las voces tiene una pronunciación más fuerte o un pico energético relevante.
En cuanto al eje horizontal, este representa el tiempo en segundos, extendiéndose desde 0 hasta 1.5 segundos, igual que en el espectrograma. Esto confirma que ambas representaciones cubren el mismo período de tiempo.

La región de mayor amplitud se situa de 0.5 a 0.8 s, lo que implica la interacción de  las voces simultáneamente. Este comportamiento coincide con la aparición de bandas de energía más pronunciadas en el espectrograma. 
Durante ese tiempo, hay una mayor cantidad de actividad  que se refleja tanto en el dominio del tiempo como en el dominio de la frecuencia.
De 1.2 s en adelante, cerca del  final de la señal, la amplitud se reduce notablemente, lo que indica que las voces están disminuyendo su intensidad, lo que es coherente con el espectrograma, donde las bandas de frecuencia pierden energía hacia el final del mismo.

### Análisis de componentes independientes (ICA)
El objetivo de esta sección del código es aplicar la técnica de ICA para separar las señales mezcladas. 
ICA es una técnica estadística que se utiliza para separar un conjunto de señales mixtas en componentes independientes. ICA se basa en la idea de que las señales observadas son combinaciones lineales de señales independientes que se originan en fuentes separadas. La idea es recuperar las señales originales a partir de las señales mezcladas analizadas.
Se desarrolla de la siguiente manera: 
1. Centralización: Se resta la media de cada señal para centrar los datos en torno a cero.
2. Normalización: Se ajusta la varianza de cada señal para que tengan varianza unitaria, facilitando la separación.
3. Descomposición: Se busca una matriz de separación W que permita obtener las señales independientes S a partir de las señales observadas X:
* S=W⋅X
En python existen varias funciones que implementan algoritmos para encontrar W, como FastICA (usado en este proyecto), que utiliza aproximaciones rápidas para calcular la matriz de separación.

```python 
# Crear una matriz con las señales
X = np.vstack([audio1, audio2, audio3])

# Centrar y normalizar las señales
X -= np.mean(X, axis=0)  # Centrar en torno a cero
X /= np.std(X, axis=0)   # Normalizar

# Aplicar ICA
ica = FastICA(n_components=3, random_state=0)  # Número de componentes igual al número de señales originales
S_ica = ica.fit_transform(X)  # Componentes independientes

# Seleccionar la columna 2 que corresponde a la señal separada 2
audio_separado = S_ica[:, 2]  
```
El resultado de la sepración fue el siguiente:

#### Reproducción audio 2 original:

https://github.com/user-attachments/assets/8c7c11c1-41b9-4edc-85b7-a03069cb8206


#### Reproducción voz separada de audio 2:

https://github.com/user-attachments/assets/6d56e08e-e02f-4030-ab3d-21930d011d4a

Se realizó el cálculo del SNR respecto a las dos señales y se obtuvo un valor de 13,789.
Este valor está en un nivel moderado donde el ruido es perceptible y afecta la claridad de la señal. Generalmente, un SNR inferior a 20 dB indica una señal donde el ruido comienza a ser notable, y a medida que se acerca a 10 dB, la interferencia de ruido se vuelve considerable. Se puede determinar que a pesar de que se hizo todo el proceso de separación de manera adecuada, no se consiguió extraer únicamente la voz, eliminando todo el ruido que causara interferencia. 
Esto puede ser debido a los métodos de separación y a las condiciones de grabación de los audios. En este caso, al realizar una primera grabación se determinó que la distribución de fuentes y sensores no era la indicada, ya que no fue posible
separar ninguna de las voces; por lo que se re-grabaron los audios teniendo en cuenta los parámetros indicados en la parte inicial de este archivo. Adicionalmente la frecuencia de muestreo también puede influir en el resultado, pues al tener una frecuencia pequeña -como la de este proyecto- es más complejo que los algoritmos puedan diferenciar las voces. 

Adicionalmente se aplicó el análisis espectral y temporal a la señal separada, obteniendo así su forma de onda y el espectograma para hacer una descripción más precisa de los resultados. 

![Forma de onda voz separada del audio 2](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/Forma%20de%20onda%20audio%202Se.png?raw=true)

*Forma de onda para una voz separada del audio 2 realizado en python.*

La amplitud de esta señal alcanza valores máximos cercanos a ±0.45, con oscilaciones iniciales  bastante regulares, con un incremento gradual de la amplitud hasta llegar a un punto máximo en torno a los 0.3 - 0.6 segundos. La amplitud muestra un patrón decreciente a partir de ese punto, reduciéndose gradualmente hacia el final de la señal.
La región de mayor amplitud se sitúa entre los 0.2 s y 0.7 s, con varios picos de energía prominentes.
Si hacemos una comparación con la forma de onda del audio original, el eje de tiempo se extiende hasta 1.5 segundos para ambas señales, lo cual es consistente con el análisis previo. Ambas señales cubren el mismo rango temporal, lo que permite hacer una comparación directa en términos de la dinámica de la señal.
En el caso incial la superposición de voces hace que el periodo de mayor actividad sea más amplio y abarque casi todo el rango hasta los 0.8 s. En cambio, en la señal separada, la concentración de energía está más limitada, reflejando la menor interacción en la señal con una sola voz.
La amplitud máxima de esta señal es un poco menor, lo que sugiere que esta voz específica es menos intensa, pues en el caso anterior se encontraban las tres voces mezcladas. 

![Espectograma  voz separada audio 2](https://github.com/lavaltt/Problema_del_coctel__procesamiento_de_senales/blob/main/espectograma%20audio%202Se.png?raw=true)

*Espectograma para una voz separada del audio 2 realizado en python.*

El eje de tiempo se extiende hasta alrededor de 1.5 segundos, lo que indica que la extracción no ha alterado la longitud de la grabación.
Las frecuencias más relevantes también se concentran principalmente en el rango de 64 Hz a 512 Hz, similar al espectrograma original, pero con algunas diferencias importantes en la distribución de la energía, pues 
se observa una reducción en la cantidad de energía distribuida en las frecuencias más altas. Es decir, las bandas por encima de los 512 Hz son menos evidentes, bandas que hacian referencia a ruidos adicionales, como lo son las demás voces.
Las franjas de energía en el rango de 128 Hz a 512 Hz son más definidas y menos superpuestas en comparación con el espectrograma original, lo que indica que la señal  es más "limpia" y presenta menos interferencias.

En términos de claridad, el espectrograma de la voz separada tiene bandas de energía más aisladas y definidas, lo que sugiere que la separación ha sido efectiva. Sin embargo, aún se aprecia la presencia de componentes en frecuencias más bajas y altas, lo que coincide con el bajo valor del SNR. 
Adicionalmente, la voz que ha sido separada ocupa la mayor parte de las frecuencias bajas a medias (entre 128 Hz y 512 Hz), lo que es típico de una voz masculina -tal como la que fue separada-.

Por último se creó una función que permitiera guardar el audio separado en una carpeta del computador.
```python 
def save_audio(file_path, audio, sr):
    sf.write(file_path, audio, sr)
    print(f"Audio guardado en {file_path}")
```
Con lo anterior se asegura tener el audio procesado guardado, para su correcta comparación con el audio original. 



[^1^]: Bee, M. A., & Micheyl, C. (2008). The cocktail party problem: What is it? How can it be solved? And why should animal behaviorists study it? Journal of Comparative Psychology (Washington, D.C.: 1983), 122(3), 235–251. https://doi.org/10.1037/0735-7036.122.3.235
[^2^]:Cortador online de Mp3 - Mp3cut.net.(https://mp3cut.net/es/)
[^3^]:Convertir MP3 a WAV (Online y Gratis) — Convertio. (s/f). Convertio.co.  https://convertio.co/es/mp3-wav/
[^4^]:Smith, S. W. (2011). The Scientist and Engineer's Guide to Digital Signal Processing. California Technical Publishing.
[^5^]: Proakis, J. G., & Manolakis, D. K. (2007). Digital Signal Processing: Principles, Algorithms, and Applications (4th ed.). Pearson.
[^6^]: (S/f). Watchguard.com.  https://www.watchguard.com/help/docs/fireware/12/es-419/Content/es-419/wireless/ap_wireless_signalstrength_c.html#:~:text=El%20SNR%20(relaci%C3%B3n%20se%C3%B1al%2Druido,y%20el%20nivel%20de%20ruido.&text=En%20general%2C%20debe%20tener%20un,un%20mal%20desempe%C3%B1o%20y%20velocidad.
[^7^]: (S/f). Vocal.com.  https://vocal.com/blind-signal-separation/independent-component-analysis/
[^8^]: (S/f). Medium.com. Recuperado el 5 de septiembre de 2024, de https://medium.com/@shreshta01/demystifying-the-cocktail-party-problem-57b8a1dfb6ad




