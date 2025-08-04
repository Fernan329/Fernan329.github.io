<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Calculadora de Ahorro y Crédito</title>
  <link href="https://cdn.jsdelivr.net/npm/tailwindcss@2.2.19/dist/tailwind.min.css" rel="stylesheet" />
  <link href="https://cdnjs.cloudflare.com/ajax/libs/animate.css/4.1.1/animate.min.css" rel="stylesheet" />
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      background-image: url('https://www.banrural.com.gt/themes/custom/banrural/images/bg-main.jpg');
      background-size: cover;
      background-attachment: fixed;
      background-repeat: no-repeat;
    }
    .navbar {
      background-color: #006837;
    }
    .glass {
      background: rgba(255, 255, 255, 0.88);
      backdrop-filter: blur(8px);
      border: 2px solid rgba(0,0,0,0.1);
    }
    input[type=range]::-webkit-slider-thumb {
      background: #047857;
    }
    .tooltip {
      font-size: 0.85rem;
      color: #4B5563;
    }
    .error-message {
      color: red;
      font-size: 0.875rem;
    }
  </style>
</head>
<body class="text-gray-800">
  <!-- Barra de navegación -->
  <nav class="navbar shadow-lg text-white px-6 py-4 flex justify-between items-center">
    <div class="text-xl font-bold">💼 Calculadora Financiera</div>
    <div class="space-x-4">
      <a href="#" class="hover:underline">Inicio</a>
      <a href="#formulario" class="hover:underline">Comparar</a>
      <a href="#resultados" class="hover:underline">Resultados</a>
    </div>
  </nav>

  <div class="max-w-5xl mx-auto py-10 px-6 animate__animated animate__fadeIn">
    <div class="glass p-8 rounded-2xl shadow-xl">
      <h1 class="text-4xl font-bold text-center mb-4">💰 Comparador de Ahorro y Crédito</h1>
      <p class="text-center text-lg mb-6">Compara opciones financieras fácilmente. Calculamos el <strong>Valor Actual Neto (VAN)</strong> para que tomes decisiones informadas.</p>

      <div class="mb-6 bg-green-100 border border-green-300 p-4 rounded text-sm">
        <p><strong>¿Cómo usar esta calculadora?</strong></p>
        <ul class="list-disc ml-5 mt-2">
          <li>Selecciona si es <strong>Crédito</strong> o <strong>Ahorro</strong>.</li>
          <li>Ingresa el nombre, monto, tasa anual, y elige el plazo con la barra.</li>
          <li>Agrega varias opciones si deseas compararlas.</li>
          <li>Haz clic en <strong>Comparar Opciones</strong> y revisa resultados y gráficas.</li>
        </ul>
      </div>

      <form id="formulario" class="grid grid-cols-1 md:grid-cols-2 gap-4">
        <div>
          <label class="font-semibold">Tipo de opción:</label>
          <select id="tipo" class="w-full p-2 border rounded">
            <option value="credito">Crédito</option>
            <option value="ahorro">Ahorro</option>
          </select>
        </div>
        <div>
          <label class="font-semibold">Nombre de la opción:</label>
          <input type="text" id="nombre" class="w-full p-2 border rounded" placeholder="Ej: Préstamo Banrural" required />
          <p class="tooltip">Escribe un nombre descriptivo para identificar la opción.</p>
        </div>
        <div>
          <label class="font-semibold">Monto (Q):</label>
          <input type="text" id="monto" class="w-full p-2 border rounded" placeholder="Ej: 10000" required />
          <p class="tooltip">Entre Q1 y Q1,000,000</p>
          <p id="montoError" class="error-message hidden">El monto debe estar entre Q1 y Q1,000,000.</p>
        </div>
        <div>
          <label class="font-semibold">Plazo en meses: <span id="mesesLabel">12</span> meses</label>
          <input type="range" id="meses" min="1" max="240" step="1" value="12" class="w-full" />
          <p class="tooltip">Selecciona un plazo entre 1 y 240 meses</p>
        </div>
        <div>
          <label class="font-semibold">Tasa anual (%):</label>
          <input type="number" id="tasa" class="w-full p-2 border rounded" step="0.01" placeholder="Ej: 7.5" required />
          <p class="tooltip">Tasa de interés efectiva anual (ejemplo: 7.5)</p>
        </div>
        <div class="flex items-end">
          <button type="button" onclick="agregarOpcion()" class="bg-blue-600 text-white px-4 py-2 rounded hover:bg-blue-700 transition-all">Agregar opción</button>
        </div>
      </form>

      <div class="text-center mt-6">
        <button onclick="compararOpciones()" class="bg-green-600 text-white px-6 py-3 rounded-lg hover:bg-green-700 transition-all animate__animated animate__pulse animate__infinite">Comparar Opciones</button>
      </div>
    </div>

    <div class="glass mt-10 p-6 rounded-xl shadow-lg animate__animated animate__fadeInUp" id="resultados"></div>
    <canvas id="grafico" class="mt-6"></canvas>
  </div>

  <footer class="bg-gray-900 text-white mt-10 text-center py-4 text-sm">
    © 2025 Herramienta Educativa Financiera. Desarrollado para ayudarte a tomar decisiones informadas.
  </footer>

  <script>
    let opciones = [];

    const montoInput = document.getElementById("monto");
    const montoError = document.getElementById("montoError");

    // Validación del monto en tiempo real
    montoInput.addEventListener("input", () => {
      let valor = montoInput.value.replace(/\D/g, "");
      if (valor.length > 1) valor = parseInt(valor).toLocaleString("es-GT");
      montoInput.value = valor;

      const montoNumerico = parseFloat(valor.replace(/,/g, ""));
      if (montoNumerico < 1 || montoNumerico > 1000000) {
        montoError.classList.remove("hidden");
      } else {
        montoError.classList.add("hidden");
      }
    });

    document.getElementById("meses").addEventListener("input", e => {
      document.getElementById("mesesLabel").innerText = e.target.value;
    });

    function agregarOpcion() {
      const tipo = document.getElementById("tipo").value;
      const nombre = document.getElementById("nombre").value;
      const monto = parseFloat(document.getElementById("monto").value.replace(/,/g, ""));
      const meses = parseInt(document.getElementById("meses").value);
      const tasa = parseFloat(document.getElementById("tasa").value);

      if (!nombre || isNaN(monto) || isNaN(meses) || isNaN(tasa)) {
        alert("Por favor completa todos los campos correctamente.");
        return;
      }

      if (monto < 1 || monto > 1000000) {
        alert("El monto debe estar entre Q1 y Q1,000,000.");
        return;
      }

      opciones.push({ tipo, nombre, monto, meses, tasa });
      alert(`✅ Opción "${nombre}" agregada.`);
      document.getElementById("formulario").reset();
      document.getElementById("mesesLabel").innerText = "12";
      document.getElementById("meses").value = 12;
      montoError.classList.add("hidden");
    }

    function calcularVAN(flujos, tasaAnual) {
      const tasaMensual = tasaAnual / 100 / 12;
      return flujos.reduce((acc, flujo, i) => acc + flujo / Math.pow(1 + tasaMensual, i + 1), 0);
    }

    function generarFlujos(op) {
      const { tipo, monto, meses, tasa } = op;
      const tasaMensual = tasa / 100 / 12;
      let flujos = [];

      if (tipo === "credito") {
        const cuota = monto * tasaMensual / (1 - Math.pow(1 + tasaMensual, -meses));
        flujos = [-monto];
        for (let i = 0; i < meses; i++) flujos.push(cuota);
      } else if (tipo === "ahorro") {
        flujos = [-monto];
        let saldo = monto;
        for (let i = 0; i < meses - 1; i++) {
          const interes = saldo * tasaMensual;
          saldo += interes;
          flujos.push(interes);
        }
        // Último mes: intereses + capital
        const ultimoInteres = saldo * tasaMensual;
        flujos.push(ultimoInteres + monto);
      }
      return flujos;
    }

    function compararOpciones() {
      if (opciones.length === 0) {
        alert("Agrega al menos una opción.");
        return;
      }

      let html = "<h2 class='text-2xl font-bold mb-4'>Resultados:</h2>";
      const labels = [];
      const vans = [];

      let mejor = null;

      opciones.forEach(op => {
        const flujos = generarFlujos(op);
        const van = calcularVAN(flujos, op.tasa);
        html += `<div class='bg-white border border-gray-300 p-4 my-4 rounded shadow animate__animated animate__fadeIn'><strong>${op.nombre}</strong> (${op.tipo})<br>VAN: <span class='text-blue-600 font-semibold'>Q ${van.toFixed(2)}</span><br><span class='text-sm text-gray-600'>${op.tipo === 'credito' ? 'Crédito: recibes el dinero y pagas cuotas mensuales.' : 'Ahorro: inviertes y recibes intereses cada mes.'}</span></div>`;
        labels.push(op.nombre);
        vans.push(van.toFixed(2));

        if (!mejor || van > mejor.van) {
          mejor = { nombre: op.nombre, van };
        }
      });

      if (mejor) {
        html += `<div class='mt-6 p-4 bg-yellow-100 border border-yellow-400 rounded animate__animated animate__fadeInUp'><strong>🔍 Conclusión:</strong><br>La mejor opción es <strong>${mejor.nombre}</strong> con un VAN de <span class='text-green-700 font-semibold'>Q ${parseFloat(mejor.van).toFixed(2)}</span>. Esta opción es la que más te beneficia financieramente.</div>`;
      }

      document.getElementById("resultados").innerHTML = html;
      graficar(labels, vans);
    }

    function graficar(labels, vans) {
      const ctx = document.getElementById('grafico').getContext('2d');
      if (window.myChart) window.myChart.destroy();

      window.myChart = new Chart(ctx, {
        type: 'bar',
        data: {
          labels: labels,
          datasets: [{
            label: 'VAN (Q)',
            data: vans,
            backgroundColor: 'rgba(34, 197, 94, 0.7)',
            borderColor: 'rgba(22, 163, 74, 1)',
            borderWidth: 1
          }]
        },
        options: {
          scales: {
            y: {
              beginAtZero: true
            }
          },
          plugins: {
            legend: {
              display: false
            }
          }
        }
      });
    }
  </script>
</body>
</html>
