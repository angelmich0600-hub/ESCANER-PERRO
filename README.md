<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📷 Generador PDF de Identificación (Doble Cara) - Final</title>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.js"></script>
    
    <style>
        /* (Mismos estilos CSS de la versión anterior) */
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f4f7f6;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        .container {
            background-color: #ffffff;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            max-width: 900px;
            width: 100%;
        }

        h1 {
            color: #2c3e50;
            text-align: center;
            margin-bottom: 5px;
        }

        p.subtitle {
            text-align: center;
            color: #7f8c8d;
            margin-bottom: 30px;
            border-bottom: 2px solid #ecf0f1;
            padding-bottom: 15px;
        }

        .input-group {
            display: flex;
            justify-content: space-around;
            gap: 20px;
            margin-bottom: 30px;
        }

        .input-box {
            flex: 1;
            padding: 15px;
            border: 2px dashed #3498db;
            border-radius: 8px;
            text-align: center;
            background-color: #ecf0f1;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        .input-box:hover {
            background-color: #e0e6e9;
            border-color: #2980b9;
        }

        input[type="file"] {
            display: none;
        }

        .preview-area {
            display: flex;
            justify-content: center;
            gap: 20px;
            min-height: 200px;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 20px;
        }

        .preview-area img {
            max-width: 45%;
            height: auto;
            border: 1px solid #ddd;
            object-fit: contain;
        }
        
        .placeholder {
            color: #95a5a6;
            display: flex;
            align-items: center;
            justify-content: center;
            font-style: italic;
        }

        #generate-pdf-btn {
            display: block;
            width: 100%;
            padding: 15px;
            background-color: #27ae60;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 1.2em;
            cursor: pointer;
            transition: background-color 0.3s;
        }

        #generate-pdf-btn:hover {
            background-color: #2ecc71;
        }
        
        #generate-pdf-btn:disabled {
            background-color: #95a5a6;
            cursor: not-allowed;
        }

        /* --- Estilos del Modal de Recorte --- */
        #crop-modal {
            display: none;
            position: fixed;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background: rgba(0, 0, 0, 0.8);
            z-index: 9999;
            justify-content: center;
            align-items: center;
        }
        
        .modal-content {
            background: white;
            padding: 20px;
            width: 90%;
            max-width: 600px;
            border-radius: 10px;
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
        }

        #image-to-crop {
            display: block;
            max-width: 100%;
        }
        
        .modal-buttons button {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            margin-top: 15px;
        }

        #crop-confirm-btn {
            background: #2ecc71;
            color: white;
        }

        #crop-cancel-btn {
            background: #e74c3c;
            color: white;
        }
        
        /* ESTILO NUEVO PARA EL BOTÓN DE ROTACIÓN */
        #rotate-btn {
            padding: 10px 20px;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
            background: #3498db;
            color: white;
            margin-top: 15px;
            display: block; /* Para que ocupe todo el ancho o centrarlo */
            width: 100%;
        }

    </style>
</head>
<body>

    <div class="container">
        <h1>Generador de PDF para Identificación (Doble Cara)</h1>
        <p class="subtitle">Sube, recorta al tamaño de la INE y unifica ambas caras en una sola hoja PDF tamaño A4.</p>

        <div class="input-group">
            <label for="file-frente" class="input-box">
                📸 **Frente** (Toca para tomar foto)
                <input type="file" id="file-frente" accept="image/*" capture="environment" onchange="handleFileInput(event, 'frente')">
            </label>
            
            <label for="file-reverso" class="input-box">
                📸 **Reverso** (Toca para tomar foto)
                <input type="file" id="file-reverso" accept="image/*" capture="environment" onchange="handleFileInput(event, 'reverso')">
            </label>
        </div>

        <h2>Vista Previa (PDF Final)</h2>
        <div class="preview-area">
            <img id="preview-frente" src="" alt="Frente - Recortado" style="display:none;">
            <img id="preview-reverso" src="" alt="Reverso - Recortado" style="display:none;">
            <div id="placeholder" class="placeholder" style="display:flex;">Sube y recorta ambas imágenes.</div>
        </div>

        <button id="generate-pdf-btn" disabled>
            ⬇️ Generar y Compartir / Descargar PDF
        </button>
    </div>

    <div id="crop-modal">
        <div class="modal-content">
            <h2>✂️ Recortar Imagen</h2>
            <p>Ajusta el área para que coincida con tu credencial. El ratio es fijo (tarjeta estándar).</p>
            <div style="max-height: 400px; overflow: hidden; margin-bottom: 15px;">
                <img id="image-to-crop">
            </div>
            
            <button id="rotate-btn">🔄 Rotar 90°</button>
            <div class="modal-buttons">
                <button id="crop-confirm-btn">Confirmar Recorte</button>
                <button id="crop-cancel-btn">Cancelar</button>
            </div>
        </div>
    </div>


    <script>
        // Variables globales
        let imgDataFrente = null;
        let imgDataReverso = null;
        let currentCropper = null;
        let currentSide = null;
        let rotated = 0; // Variable para rastrear la rotación acumulada
        
        const { jsPDF } = window.jspdf;

        // Referencias del DOM
        const D = {
            cropModal: document.getElementById('crop-modal'),
            imageToCrop: document.getElementById('image-to-crop'),
            cropConfirmBtn: document.getElementById('crop-confirm-btn'),
            cropCancelBtn: document.getElementById('crop-cancel-btn'),
            generatePdfBtn: document.getElementById('generate-pdf-btn'),
            placeholder: document.getElementById('placeholder'),
            previewFrente: document.getElementById('preview-frente'),
            previewReverso: document.getElementById('preview-reverso'),
            // NUEVA REFERENCIA
            rotateBtn: document.getElementById('rotate-btn') 
        };

        // --- MANEJADORES DE EVENTOS ---
        D.cropConfirmBtn.onclick = confirmCrop;
        D.cropCancelBtn.onclick = cancelCrop;
        D.generatePdfBtn.onclick = generatePDF;
        // MANEJADOR DE EVENTO PARA EL NUEVO BOTÓN
        D.rotateBtn.onclick = () => { rotateImage(90) }; 


        /**
         * Maneja la subida del archivo y comienza el proceso de recorte.
         */
        function handleFileInput(event, side) {
            const file = event.target.files[0];

            if (file) {
                if (!['image/jpeg', 'image/png'].includes(file.type)) {
                    alert('Formato de archivo no soportado. Por favor, sube una imagen JPEG o PNG.');
                    event.target.value = '';
                    return;
                }

                const reader = new FileReader();
                reader.onload = (e) => {
                    currentSide = side;
                    rotated = 0; // Reinicia la rotación para cada nueva imagen
                    
                    if (currentCropper) currentCropper.destroy();
                    D.imageToCrop.src = e.target.result;
                    
                    D.cropModal.style.display = 'flex';
                    
                    // Inicializa Cropper.js
                    setTimeout(() => {
                         currentCropper = new Cropper(D.imageToCrop, {
                            aspectRatio: 1.586, // Ratio estándar de tarjeta de crédito/INE
                            viewMode: 1, 
                            dragMode: 'move',
                            autoCropArea: 0.9,
                        });
                    }, 50);
                };
                reader.readAsDataURL(file);
            }
        }
        
        /**
         * Rotar la imagen 90 grados usando Cropper.js
         */
        function rotateImage(degree) {
            if (currentCropper) {
                rotated += degree;
                currentCropper.rotateTo(rotated);
            }
        }


        /**
         * Confirma el recorte y actualiza las variables de datos y la vista previa.
         */
        function confirmCrop() {
            if (!currentCropper) return;

            // Obtiene la imagen recortada en formato base64 con un ancho de 500px para buena calidad
            const base64Data = currentCropper.getCroppedCanvas({
                imageSmoothingQuality: 'high',
                width: 500, 
            }).toDataURL('image/jpeg', 0.9);

            // Actualiza la variable de datos y la vista previa
            const previewElement = (currentSide === 'frente') ? D.previewFrente : D.previewReverso;
            previewElement.src = base64Data;
            previewElement.style.display = 'block';

            if (currentSide === 'frente') {
                imgDataFrente = base64Data;
            } else {
                imgDataReverso = base64Data;
            }

            // Cierra el modal y limpia el cropper
            closeCropModal();
            checkAndEnableButton();
        }

        /**
         * Cancela la operación de recorte y cierra el modal.
         */
        function cancelCrop() {
            closeCropModal();
        }

        function closeCropModal() {
            if (currentCropper) {
                currentCropper.destroy();
                currentCropper = null;
            }
            D.cropModal.style.display = 'none';
        }

        /**
         * Verifica el estado de las imágenes y habilita/deshabilita el botón.
         */
        function checkAndEnableButton() {
            if (imgDataFrente && imgDataReverso) {
                D.generatePdfBtn.disabled = false;
                D.placeholder.style.display = 'none';
            } else {
                D.generatePdfBtn.disabled = true;
                D.placeholder.style.display = 'flex';
            }
        }
        
        // --- FUNCIÓN PRINCIPAL DE GENERACIÓN Y COMPARTICIÓN ---

        /**
         * Genera el PDF, luego intenta compartirlo o lo descarga.
         */
        function generatePDF() {
            if (D.generatePdfBtn.disabled) return;
            
            const doc = new jsPDF('portrait', 'mm', 'a4');
            const margin = 15; 
            const spacing = 5; 
            const cardWidth = 85.6; // Ancho estándar INE en mm
            const cardHeight = 54; // Alto estándar INE en mm
            const fileName = 'Identificacion_Doble_Cara.pdf';

            try {
                // Colocar el FRENTE
                doc.addImage(imgDataFrente, 'JPEG', margin, margin, cardWidth, cardHeight);

                // Colocar el REVERSO
                const xReverso = margin + cardWidth + spacing; 
                const yReverso = margin; 
                doc.addImage(imgDataReverso, 'JPEG', xReverso, yReverso, cardWidth, cardHeight);
                
                // 1. Convertir el PDF generado a un Blob (necesario para compartir)
                const pdfBlob = doc.output('blob');

                // 2. Intentar compartir si el navegador lo soporta
                const fileToShare = new File([pdfBlob], fileName, { type: 'application/pdf' });
                
                if (navigator.canShare && navigator.canShare({ files: [fileToShare] })) {
                    
                    sharePDF(fileToShare);
                    
                } else {
                    // 3. Si no se puede compartir, forzar la descarga
                    doc.save(fileName);
                    alert("Tu dispositivo/navegador no soporta la opción de compartir archivos. El PDF ha sido descargado automáticamente.");
                }
                
            } catch (error) {
                alert("Ocurrió un error al generar el PDF. Revisa las imágenes.");
                console.error("Error al generar PDF:", error);
            }
        }
        
        /**
         * Llama a la Web Share API para abrir el menú nativo del móvil.
         */
        async function sharePDF(pdfFile) {
            try {
                await navigator.share({
                    files: [pdfFile],
                    title: 'INE CLIENTE',
                    text: 'Suerte en la venta 🌟.',
                });
                
                console.log('PDF compartido con éxito.');

            } catch (error) {
                if (error.name === 'AbortError') {
                     console.log('Compartir cancelado por el usuario.');
                } else {
                     console.error('Error al intentar compartir:', error);
                     alert('No se pudo compartir. El PDF será descargado.');
                     // Fallback: si falla el compartir (pero no por cancelación), se descarga
                     const doc = new jsPDF('portrait', 'mm', 'a4');
                     doc.addImage(imgDataFrente, 'JPEG', 15, 15, 85.6, 54);
                     doc.addImage(imgDataReverso, 'JPEG', 15 + 85.6 + 5, 15, 85.6, 54);
                     doc.save('Identificacion_Doble_Cara.pdf');
                }
            }
        }
        
        // Inicializa el estado del botón
        checkAndEnableButton();
    </script>

</body>
</html>
