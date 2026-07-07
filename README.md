# cotizador
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Cotizador Glydea</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background: #f0f2f5; }
        .container { max-width: 1300px; margin: auto; background: white; padding: 25px; border-radius: 10px; box-shadow: 0 4px 15px rgba(0,0,0,0.1); }
        h1 { color: #1e3a8a; text-align: center; }
        .fecha { text-align: center; font-size: 1.1em; margin-bottom: 20px; }
        .tabs { display: flex; justify-content: center; gap: 15px; margin: 20px 0; }
        .tab { padding: 12px 30px; font-size: 1.1em; border: none; border-radius: 8px; cursor: pointer; }
        .tab.active { background: #1e3a8a; color: white; }
        table { width: 100%; border-collapse: collapse; margin: 20px 0; }
        th, td { border: 1px solid #999; padding: 10px; text-align: left; }
        th { background: #1e3a8a; color: white; }
        input { width: 100%; padding: 6px; box-sizing: border-box; }
        .total { font-size: 1.4em; font-weight: bold; color: #1e3a8a; text-align: right; margin: 20px 0; }
        button { padding: 12px 25px; margin: 8px; font-size: 1em; border: none; border-radius: 6px; cursor: pointer; }
        .btn-primary { background: #1e3a8a; color: white; }
        .btn-secondary { background: #6c757d; color: white; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🛠️ Cotizador Glydea</h1>
        <div class="fecha"><strong>Fecha:</strong> <span id="fechaActual"></span></div>

        <div class="tabs">
            <button class="tab active" onclick="switchTab(0)">FRANCES</button>
            <button class="tab" onclick="switchTab(1)">RIPPLE FOLD 100%</button>
        </div>

        <h2>Datos del Cliente</h2>
        <textarea id="billTo" rows="4" style="width:100%; margin-bottom:10px;">Fábrica de Persianas y Complementos, S.A.
NIT 688207-2

Km. 22.50 Carretera a El Salvador, Fraijanes
PC 01062</textarea>

        <textarea id="shipTo" rows="4" style="width:100%;">KOPE LOGISTICS, INC.
Doral, FL 33178
6550 NW 97th AVE Suite 220
Contact: Lady Aranzazu / (305) 570-2652
Email: info@kopelogistics.com</textarea>

        <h2>Productos</h2>
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

        <div class="total" id="grandTotal">Total: $0.00 USD</div>

        <div style="text-align:center; margin-top:30px;">
            <button class="btn-primary" onclick="generatePDF()">📄 Generar PDF</button>
            <button class="btn-secondary" onclick="resetAll()">🔄 Reiniciar Todo</button>
        </div>
    </div>

    <script>
        let currentTab = 0;

        const data = [
            // FRANCES
            [
                {item:"1782084", desc:"GLYDEA 35/60 CINTA DENTADA", qty:16.2, unit:"U", price:1.7876},
                {item:"1780947", desc:"GLYDEA 35/60 BRAZO DE SUPERPOSICION", qty:1, unit:"U", price:14.1658},
                {item:"1781415", desc:"GLYDEA 35/60 CARRITO CON OJILLO GIRATORIO", qty:80, unit:"U", price:1.04545},
                {item:"1781344", desc:"GLYDEA 35/60 CARRITO MAESTRO RIPPLE FOLD", qty:1, unit:"U", price:14.0718},
                {item:"1780945", desc:"GLYDEA 35/60 POLEA DE MOTOR", qty:2, unit:"U", price:7.7677},
                {item:"1780907", desc:"GLYDEA 35/60 SOPORTE A TECHO", qty:12, unit:"U", price:1.5891},
                {item:"1780899", desc:"GLYDEA 35/60 RIEL PARA CORTINERO", qty:10, unit:"U", price:2.8869},
                {item:"1780898", desc:"GLYDEA 35/60 TAPA PARA POLEA BLANCA", qty:1, unit:"U", price:2.28375},
                {item:"1780895", desc:"GLYDEA 35/60 GANCHO PLASTICO", qty:1, unit:"U", price:0.3973},
                {item:"1246116", desc:"GLYDEA 35e ULTRA RTS CON CLAVIJA", qty:1, unit:"U", price:294.6705},
                {item:"1782316", desc:"GLYDEA 35/60 RIEL DE UNION INTERNA", qty:1, unit:"U", price:6.7954}
            ],
            // RIPPLE FOLD
            [
                {item:"1782084", desc:"GLYDEA 35/60 CINTA DENTADA", qty:20.2, unit:"U", price:1.7876},
                {item:"1780902", desc:"GLYDEA 35/60 CARRITO PARA CORTINERO RPF", qty:100, unit:"U", price:1.2232},
                {item:"1781344", desc:"GLYDEA 35/60 CARRITO MAESTRO RIPPLE FOLD", qty:2, unit:"U", price:14.0718},
                {item:"1780945", desc:"GLYDEA 35/60 POLEA DE MOTOR", qty:2, unit:"U", price:7.7677},
                {item:"1780907", desc:"GLYDEA 35/60 SOPORTE A TECHO", qty:14, unit:"U", price:1.5891},
                {item:"1780899", desc:"GLYDEA 35/60 RIEL PARA CORTINERO", qty:10, unit:"M", price:9.4718},
                {item:"1780905", desc:"GLYDEA 35/60 GANCHO PLASTICO RIPPLE", qty:2, unit:"U", price:0.5227},
                {item:"1246116", desc:"GLYDEA 35e ULTRA RTS CON CLAVIJA", qty:1, unit:"U", price:294.6705},
                {item:"1782316", desc:"GLYDEA 35/60 RIEL DE UNION INTERNA", qty:1, unit:"U", price:6.7954},
                {item:"1782301", desc:"GLYDEA 35/60 CINTA PARA CORTINERO 4 - 1/4", qty:24, unit:"M", price:2.3941}
            ]
        ];

        function updateDate() {
            const today = new Date();
            document.getElementById("fechaActual").textContent = today.toLocaleDateString('es-ES', { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
        }

        function renderTable() {
            const tbody = document.getElementById('tableBody');
            tbody.innerHTML = '';
            const items = data[currentTab];

            items.forEach((item, i) => {
                const total = (item.qty * item.price).toFixed(2);
                const row = document.createElement('tr');
                row.innerHTML = `
                    <td>${i+1}</td>
                    <td><input type="text" value="${item.item}" onchange="data[${currentTab}][${i}].item = this.value; renderTable()"></td>
                    <td><input type="text" value="${item.desc}" onchange="data[${currentTab}][${i}].desc = this.value; renderTable()"></td>
                    <td><input type="number" step="0.01" value="${item.qty}" onchange="data[${currentTab}][${i}].qty = parseFloat(this.value)||0; renderTable()"></td>
                    <td><input type="text" value="${item.unit}" onchange="data[${currentTab}][${i}].unit = this.value; renderTable()"></td>
                    <td><input type="number" step="0.0001" value="${item.price}" onchange="data[${currentTab}][${i}].price = parseFloat(this.value)||0; renderTable()"></td>
                    <td>$${total}</td>
                `;
                tbody.appendChild(row);
            });
            updateTotal();
        }

        function updateTotal() {
            const total = data[currentTab].reduce((sum, item) => sum + (item.qty * item.price), 0);
            document.getElementById('grandTotal').textContent = `Total: $${total.toFixed(2)} USD`;
        }

        function switchTab(tab) {
            currentTab = tab;
            document.querySelectorAll('.tab').forEach((t, i) => t.classList.toggle('active', i === tab));
            renderTable();
        }

        function generatePDF() {
            const { jsPDF } = window.jspdf;
            const doc = new jsPDF();
            const fecha = document.getElementById('fechaActual').textContent;

            doc.setFontSize(18);
            doc.text("COTIZACIÓN GLYDEA", 105, 20, { align: "center" });
            doc.setFontSize(11);
            doc.text(fecha, 105, 30, { align: "center" });

            doc.text("Bill To:", 20, 50);
            doc.text(document.getElementById('billTo').value, 20, 55);

            doc.text("Ship To:", 20, 90);
            doc.text(document.getElementById('shipTo').value, 20, 95);

            let y = 130;
            doc.text("Productos:", 20, y);
            y += 10;

            data[currentTab].forEach(item => {
                if (item.qty > 0) {
                    const subtotal = (item.qty * item.price).toFixed(2);
                    doc.text(`${item.item} | ${item.desc.substring(0,45)}... | ${item.qty} ${item.unit} | $${subtotal}`, 20, y);
                    y += 8;
                }
            });

            const grandTotal = data[currentTab].reduce((sum, item) => sum + item.qty * item.price, 0).toFixed(2);
            doc.setFontSize(14);
            doc.text(`TOTAL GENERAL: $${grandTotal} USD`, 20, y + 15);

            doc.save(`Cotizacion_Glydea_${currentTab === 0 ? 'FRANCES' : 'RIPPLE'}_${new Date().toISOString().slice(0,10)}.pdf`);
        }

        function resetAll() {
            if (confirm("¿Reiniciar todas las cantidades y datos?")) {
                location.reload();
            }
        }

        // Iniciar app
        window.onload = () => {
            updateDate();
            renderTable();
        };
    </script>
</body>
</html>