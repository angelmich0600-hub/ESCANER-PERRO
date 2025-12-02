
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>📷 PDF de Credencial (Doble Cara)</title>

    <!-- jsPDF -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>

    <!-- CropperJS -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.css" />
    <script src="https://cdnjs.cloudflare.com/ajax/libs/cropperjs/1.6.1/cropper.min.js"></script>

    <style>
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
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            max-width: 900px;
            width: 100%;
        }

        h1 {
            color: #2c3e50;
            text-align: center;
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
            border: 2px dashed #bdc3c7;
            border-radius: 8px;
            background-color: #ecf0f1;
            text-align: center;
            cursor: pointer;
        }

        .preview-area {
            display: flex;
            justify-content: center;
            gap: 20px;
            min-height: 200px;
            border: 1px solid #ddd;
            padding: 15px;
            border-radius: 8px;
        }

        .preview-area img {
            max-width: 45%;
            border: 1px solid #ddd;
        }

        #generate-pdf-btn {
            width: 100%;
            padding: 15px;
            background-color: #3498db;
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 1.2em;
            cursor: pointer;
        }

        #generate-pdf-btn:disabled {
            background-color: #95a5a6;
        }

        /* Modal */
        #crop-modal {
            display: none;
            position: fixed;
            top:0; left:0;
            width:100%; height:100%;
            background:rgba(0,0,0,0.8);
            z-index:9999;
            justify-content:center;
            align-items:center;
        }

        #crop-box {
            background:white;
            padding:20px;
            width:90%;
            max-width:600px;
            border-radius:10px;
        }
    </style>
</head>

<body>

    <div class="container">
        <h1>Generador de PDF para Identificación</h1>
        <p class="subtitle">Sube el frente y reverso de tu credencial</p>

        <div class="input-group">
            <label class="input-box">
                📤 Subir Frente
                <input type="file" id="file-frente" accept="image/*">
            </label>

            <label class="input-box">
                📥 Subir Reverso
                <input type="file" id="file-reverso" accept="image/*">
            </label>
        </div>

        <h2>Vista Previa</h2>
        <div class="preview-area">
            <img id="preview-frente" style="display:none;">
            <img id="preview-reverso" style="display:none;">
        </div>

        <button id="generate-pdf-btn" disabled>⬇️ Generar PDF</button>
    </div>

    <!-- Modal de recorte -->
    <div id="crop-modal">
        <div id="crop-box">
            <h2>✂️ Recortar Imagen</h2>
            <div style="max-height:400px; overflow:hidden;">
                <img id="image-to-crop" style="max-width:100%;">
            </div>

            <button id="crop-confirm-btn" style="background:#2ecc71;color:white;padding:10px;">Confirmar</button>
            <button id="crop-cancel-btn" style="background:#e74c3c;color:white;padding:10px;">Cancelar</button>
        </div>
    </div>

<script>
let imgDataFrente = null;
let imgDataReverso = null;
let currentSide = null;
let cropper = null;

const { jsPDF } = window.jspdf;

const modal = document.getElementById("crop-modal");
const imgElement = document.getElementById("image-to-crop");

document.getElementById("file-frente").onchange = () => loadAndCrop("frente");
document.getElementById("file-reverso").onchange = () => loadAndCrop("reverso");

function loadAndCrop(side) {
    const input = document.getElementById("file-" + side);
    const file = input.files[0];
    if (!file) return;

    currentSide = side;

    const reader = new FileReader();
    reader.onload = e => {
        imgElement.src = e.target.result;
        modal.style.display = "flex";

        if (cropper) cropper.destroy();

        setTimeout(() => {
            cropper = new Cropper(imgElement, {
                aspectRatio: 1.586,
                viewMode: 1,
                autoCropArea: 0.9
            });
        }, 100);
    };

    reader.readAsDataURL(file);
}

document.getElementById("crop-confirm-btn").onclick = () => {
    cropper.getCroppedCanvas().toBlob(blob => {
        const reader = new FileReader();
        reader.onloadend = () => {
            const base64 = reader.result;

            const preview = document.getElementById("preview-" + currentSide);
            preview.src = base64;
            preview.style.display = "block";

            if (currentSide === "frente") imgDataFrente = base64;
            else imgDataReverso = base64;

            checkReady();
        };
        reader.readAsDataURL(blob);
    });

    modal.style.display = "none";
    cropper.destroy();
};

document.getElementById("crop-cancel-btn").onclick = () => {
    modal.style.display = "none";
    if (cropper) cropper.destroy();
};

function checkReady() {
    const btn = document.getElementById("generate-pdf-btn");
    btn.disabled = !(imgDataFrente && imgDataReverso);
}

document.getElementById("generate-pdf-btn").onclick = () => {
    const doc = new jsPDF("portrait", "mm", "a4");

    const margin = 10;
    const width = 90;
    const height = 60;

    doc.addImage(imgDataFrente, "JPEG", margin, margin, width, height);
    doc.addImage(imgDataReverso, "JPEG", margin + width + 10, margin, width, height);

    doc.save("credencial.pdf");
};
</script>

</body>
</html>
