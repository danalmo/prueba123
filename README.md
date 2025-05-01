<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <title>Panel de Monitoreo de Fraudes</title>
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@4.6.2/dist/css/bootstrap.min.css">
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        body {
            background-color: #f8f9fa;
        }

        h2 {
            margin-top: 20px;
            font-weight: bold;
            text-align: center;
        }

        .table thead th {
            background-color: #dc3545;
            color: white;
            text-align: center;
        }

        .card {
            margin-top: 30px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
        }

        canvas {
            max-height: 300px;
        }
    </style>
</head>
<body>
<div class="container">
    <h2>Panel de Monitoreo de Fraudes</h2>

    <!-- Tabla de fraudes -->
    <div class="table-responsive mt-4">
        <table class="table table-bordered table-hover">
            <thead>
            <tr>
                <th>Número Tarjeta</th>
                <th>Fecha Transacción</th>
                <th>ID Transacción</th>
                <th>Categoría</th>
                <th>Comercio</th>
                <th>Monto (€)</th>
                <th>Distancia</th>
                <th>Edad</th>
            </tr>
            </thead>
            <tbody id="fraud_table_body">
            <!-- Se llenará dinámicamente -->
            </tbody>
        </table>
    </div>

    <!-- Gráfico de fraude por categoría -->
    <div class="card">
        <div class="card-body">
            <h5 class="card-title text-center">Cantidad de fraudes por categoría</h5>
            <canvas id="categoryChart"></canvas>
        </div>
    </div>
</div>

<!-- Scripts -->
<script src="js/jquery-1.12.4.min.js"></script>
<script src="js/sockjs-1.1.1.min.js"></script>
<script src="js/stomp.min.js"></script>
<script>
    const ctx = document.getElementById('categoryChart').getContext('2d');
    const categoryChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: [],
            datasets: [{
                label: 'Cantidad de fraudes',
                data: [],
                backgroundColor: 'rgba(220, 53, 69, 0.6)',
                borderColor: 'rgba(220, 53, 69, 1)',
                borderWidth: 1
            }]
        },
        options: {
            responsive: true,
            scales: {
                y: {
                    beginAtZero: true,
                    title: {
                        display: true,
                        text: 'Número de fraudes'
                    }
                }
            }
        }
    });

    const socket = new SockJS('/stomp');
    const stompClient = Stomp.over(socket);

    stompClient.connect({}, function (frame) {
        stompClient.subscribe("/topic/fraudData", function (data) {
            const dataList = JSON.parse(data.body);
            const fraudes = dataList.fraudAlert;

            let filas = '';
            const contadorCategorias = {};

            fraudes.forEach(fraude => {
                filas += `
                    <tr>
                        <td>${fraude.cc_num}</td>
                        <td>${fraude.trans_time}</td>
                        <td>${fraude.trans_num}</td>
                        <td>${fraude.category}</td>
                        <td>${fraude.merchant}</td>
                        <td>${fraude.amt}</td>
                        <td>${fraude.distance}</td>
                        <td>${fraude.age}</td>
                    </tr>
                `;

                if (!contadorCategorias[fraude.category]) {
                    contadorCategorias[fraude.category] = 0;
                }
                contadorCategorias[fraude.category]++;
            });

            $("#fraud_table_body").html(filas);

            // Actualiza el gráfico
            const labels = Object.keys(contadorCategorias);
            const datos = Object.values(contadorCategorias);

            categoryChart.data.labels = labels;
            categoryChart.data.datasets[0].data = datos;
            categoryChart.update();
        });
    });
</script>
</body>
</html>
