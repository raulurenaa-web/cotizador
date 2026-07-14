# cotizador
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cotizador Glydea</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        * { box-sizing: border-box; }
        body { 
            font-family: Arial, sans-serif; 
            margin: 0; 
            padding: 15px; 
            background: #f0f2f5; 
        }
        .container { 
            max-width: 1300px; 
            margin: auto; 
            background: white; 
            padding: 20px; 
            border-radius: 12px; 
            box-shadow: 0 4px 20px rgba(0,0,0,0.1); 
        }
        h1 { color: #1e3a8a; text-align: center; margin-bottom: 8px; }
        .fecha { text-align: center; font-size: 1.05em; color: #444; margin-bottom: 20px; }
        
        .tabs { 
            display: flex; 
            justify-content: center; 
            gap: 12px; 
            margin: 25px 0; 
            flex-wrap: wrap; 
        }
        .tab { 
            padding: 12px 28px; 
            font-size: 1.05em; 
            border: none; 
            border-radius: 8px; 
            cursor: pointer; 
            background: #e2e8f0; 
        }
        .tab.active { background: #1e3a8a; color: white; }

        h2 { color: #1e3a8a; margin-top: 25px; }
        
        textarea { 
            width: 100%; 
            padding: 10px; 
            border: 1px solid #ccc; 
            border-radius: 6px; 
            margin-bottom: 12px; 
            font-family: inherit; 
        }
        
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        th, td { border: 1px solid #999; padding: 10px; text-align: left; }
        th { background: #1e3a8a; color: white; }
        .info-table td { background: #f8f9fa; }
        
        input[type="number"] { width: 100%; padding: 8px; border: 1px solid #ccc; border-radius: 4px; }
        
        /* Scroll en tabla para móviles */
        .table-wrapper {
            overflow-x: auto;
            -webkit-overflow-scrolling: touch;
            border-radius: 8px;
        }
        
        .total { 
            font-size: 1.5em; 
            font-weight: bold; 
            color: #1e3a8a; 
            text-align: right; 
            margin: 25px 0; 
            padding: 15px; 
            background: #f8fafc; 
            border-radius: 8px; 
        }
        
        .buttons {
            text-align: center;
            margin-top: 30px;
            display: flex;
            flex-wrap: wrap;
            gap: 12px;
            justify-content: center;
        }
        
        button { 
            padding: 14px 28px; 
            font-size: 1.05em; 
            border: none; 
            border-radius: 8px; 
            cursor: pointer; 
            min-height: 52px;
        }
        .btn-primary { background: #1e3a8a; color: white; }

        /* Responsive */
        @media (max-width: 768px) {
            body { padding: 10px; }
            .container { padding: 15px; }
            h1 { font-size: 1.6em; }
            th, td { padding: 8px 6px; font-size: 0.95em; }
            .tab { padding: 10px 20px; font-size: 1em; }
            button { flex: 1; min-width: 160px; }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🛠️ Cotizador Glydea</h1>
        <div class="fecha"><strong>Fecha:</strong> <span id="fechaActual"></span></div>
       
        <div class="tabs">
            <button class="tab active" onclick="switchTab(0)">COTIZADOR</button>
        </div>

        <h2>Datos del Cliente</h2>
        <textarea id="billTo" rows="5">Fábrica de Persianas y Complementos, S.A.
NIT 688207-2
Guatemala, Ciudad de Guatemala.
Km. 22.50 Carretera a El Salvador, Fraijanes
PC 01062</textarea>
        
        <textarea id="shipTo" rows="5">KOPE LOGISTICS, INC.
Doral, FL 33178
6550 NW 97th AVE Suite 220
Contact: Lady Aranzazu / (305) 570-2652
Email: info@kopelogistics.com</textarea>

        <h2>Información Técnica</h2>
        <table class="info-table">
            <thead>
                <tr>
                    <th>VIAS</th>
                    <th id="labelPart"># PARTICIONES / UNIONES</th>
                    <th>W (m)</th>
                    <th>H (m)</th>
                    <th>PESO (Kg)</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td><input type="number" id="vias" value="1" min="1" onchange="updateAll()"></td>
                    <td><input type="number" id="particiones" value="1" min="0" onchange="updateAll()"></td>
                    <td><input type="number" step="0.01" id="w" value="1" min="0" onchange="updateAll()"></td>
                    <td><input type="number" step="0.01" id="h" value="1" min="0" onchange="updateAll()"></td>
                    <td><input type="number" step="0.01" id="peso" value="1" min="0" onchange="updateAll()"></td>
                </tr>
            </tbody>
        </table>

        <h2>Productos</h2>
        <div class="table-wrapper">
            <table id="productsTable">
                <thead>
                    <tr>
                        <th>#</th>
                        <th>Item</th>
                        <th>Descripción</th>
                        <th>Cantidad</th>
                        <th>Unidad</th>
                        <th>Precio USD</th>
                        <th>Total USD</th>
                    </tr>
                </thead>
                <tbody id="tableBody"></tbody>
            </table>
        </div>

        <div class="total" id="grandTotal">Total: $0.00 USD</div>

        <div class="buttons">
            <button class="btn-primary" onclick="generatePDF()">📄 Generar PDF</button>
            <button onclick="resetAll()">🔄 Reiniciar Todo</button>
        </div>
    </div>

    <script>
        let currentTab = 0;
        let products = [];

        const baseData = [
            [
                {item:"1782084", desc:"GLYDEA 35/60 CINTA DENTADA", qty:16.2, unit:"U", price:1.79},
                {item:"1780947", desc:"GLYDEA 35/60 BRAZO DE SUPERPOSICION", qty:1, unit:"U", price:14.17},
                {item:"1781415", desc:"GLYDEA 35/60 CARRITO CON OJILLO GIRATORIO", qty:80, unit:"U", price:1.05},
                {item:"1781344", desc:"GLYDEA 35/60 CARRITO MAESTRO RIPPLE FOLD", qty:1, unit:"U", price:14.0718},
                {item:"1780945", desc:"GLYDEA 35/60 POLEA DE MOTOR", qty:2, unit:"U", price:7.77},
                {item:"1780907", desc:"GLYDEA 35/60 SOPORTE A TECHO", qty:12, unit:"U", price:1.591},
                {item:"1780899", desc:"GLYDEA 35/60 RIEL PARA CORTINERO", qty:10, unit:"U", price:2.89},
                {item:"1780898", desc:"GLYDEA 35/60 TAPA PARA POLEA BLANCA", qty:1, unit:"U", price:2.28375},
                {item:"1780895", desc:"GLYDEA 35/60 GANCHO PLASTICO", qty:1, unit:"U", price:0.40},
                {item:"1246118", desc:"GLYDEA 60e ULTRA RTS CON CLAVIJA", qty:0, unit:"U", price:360.37},
                {item:"1246116", desc:"GLYDEA 35e ULTRA RTS CON CLAVIJA", qty:1, unit:"U", price:294.67},
                {item:"1782316", desc:"GLYDEA 35/60 RIEL DE UNION INTERNA", qty:1, unit:"U", price:6.80}
            ],
            []
        ];

        const soporteRielMap = {
            1: {soporte: 2, riel: 2.5}, 2: {soporte: 3, riel: 2.5},
            3: {soporte: 5, riel: 5}, 4: {soporte: 6, riel: 5},
            5: {soporte: 7, riel: 5}, 6: {soporte: 9, riel: 7.5},
            7: {soporte: 10, riel: 7.5},8: {soporte: 12, riel: 10},
            9: {soporte: 13, riel: 10},10:{soporte: 14, riel: 10}
        };

        function calculateQuantities() {
            const vias = parseFloat(document.getElementById("vias").value) || 1;
            const particiones = parseFloat(document.getElementById("particiones").value) || 0;
            let w = Math.round(parseFloat(document.getElementById("w").value) || 0);
            const peso = parseFloat(document.getElementById("peso").value) || 0;

            products.forEach(item => {
                if (item.item === "1782084") item.qty = w * 2 + 0.2;
                if (item.item === "1781415") item.qty = w * 10;
                if (item.item === "1780895") item.qty = vias;
                if (item.item === "1781344") item.qty = vias;
                if (item.item === "1782316") item.qty = particiones;
                if (item.item === "1780907") item.qty = soporteRielMap[w] ? soporteRielMap[w].soporte : Math.ceil(w * 1.5);
                if (item.item === "1780899") item.qty = soporteRielMap[w] ? soporteRielMap[w].riel : w;
                if (item.item === "1246116") item.qty = (peso <= 35) ? 1 : 0;
                if (item.item === "1246118") item.qty = (peso > 35) ? 1 : 0;
            });
        }

        function renderTable() {
            calculateQuantities();
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = '';
            
            products.forEach((item, i) => {
                const total = (item.qty * item.price).toFixed(2);
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${i+1}</td>
                    <td>${item.item}</td>
                    <td>${item.desc}</td>
                    <td><input type="number" step="0.01" value="${parseFloat(item.qty).toFixed(1)}" onchange="products[${i}].qty = parseFloat(this.value)||0; renderTable()"></td>
                    <td>${item.unit}</td>
                    <td>$${item.price.toFixed(2)}</td>
                    <td>$${total}</td>
                `;
                tbody.appendChild(row);
            });
            updateTotal();
        }

        function updateTotal() {
            const total = products.reduce((sum, item) => sum + (item.qty * item.price), 0);
            document.getElementById('grandTotal').textContent = `Total: $${total.toFixed(2)} USD`;
        }

        function updateAll() {
            renderTable();
        }

        function switchTab(tab) {
            currentTab = tab;
            document.querySelectorAll('.tab').forEach((t, i) => t.classList.toggle('active', i === tab));
            products = JSON.parse(JSON.stringify(baseData[tab]));
            renderTable();
        }

        function generatePDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            
            doc.setFontSize(18);
            doc.text("Cotización Glydea", 105, 20, { align: "center" });
            
            doc.setFontSize(11);
            doc.text(`Fecha: ${document.getElementById("fechaActual").textContent}`, 20, 35);
            doc.text("Total: " + document.getElementById("grandTotal").textContent, 20, 50);
            
            doc.save("Cotizacion_Glydea.pdf");
            alert("✅ PDF generado y descargado correctamente");
        }

        function resetAll() {
            if (confirm("¿Reiniciar todo?")) location.reload();
        }

        window.onload = () => {
            document.getElementById("fechaActual").textContent = new Date().toLocaleDateString('es-ES', {
                weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'
            });
            products = JSON.parse(JSON.stringify(baseData[0]));
            renderTable();
        };
    </script>
</body>
</html>
