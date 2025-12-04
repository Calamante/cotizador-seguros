Cotizador de Seguros para el Hogar (React + Vite)

Este es un cotizador de seguros de hogar totalmente funcional desarrollado con React y estilizado con Bootstrap 5. Permite a los usuarios ingresar datos de su propiedad, seleccionar coberturas y obtener un cálculo de costo anual en tiempo real.

 Características Principales:

*Formulario Multi-Paso: Proceso de cotización guiado en 3 pasos (Datos, Riesgos, Coberturas).

*Cálculo en Tiempo Real: Algoritmo que calcula el costo anual basado en M² y factores de riesgo (Ubicación, Tipo de Propiedad, Antigüedad).

*Gestión de Pólizas: Permite guardar cotizaciones, y cambiar el estado a "Contratada" o "Cancelada".

*Persistencia Local: Utiliza el localStorage del navegador para guardar y recuperar cotizaciones previamente generadas.

*Diseño Responsivo: Estilizado con Bootstrap para una visualización óptima en dispositivos móviles y de escritorio.

 Tecnologías Utilizadas

 * React (usando Hooks como useState, useEffect, useCallback).

 *Vite

*Estilos: Bootstrap 5 (a través de CDN)

*Iconografía: Lucide React


 ----Instalación y Ejecución-----

Sigue estos pasos para obtener una copia local del proyecto y ponerlo en marcha.

Requisitos

Node.js (versión LTS recomendada)

npm (incluido con Node.js)

Pasos

Clona el Repositorio:

git clone https://github.com/Calamante/cotizador-seguros.git
cd cotizador-seguros


Instala las Dependencias:

npm install


Instala Librerías Adicionales (Iconos):

npm install lucide-react


Ejecuta la Aplicación en Modo Desarrollo:

npm run dev
