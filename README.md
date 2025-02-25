<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Organizador de Gastos</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 20px;
            padding: 0;
            background-color: #f4f4f4;
        }
        h2, h3 {
            text-align: center;
        }
        .container {
            max-width: 700px;
            margin: auto;
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
        }
        input, select, button {
            width: 100%;
            padding: 10px;
            margin: 5px 0;
            border-radius: 5px;
            border: 1px solid #ddd;
        }
        button {
            background-color: #28a745;
            color: white;
            cursor: pointer;
            border: none;
        }
        button:hover {
            background-color: #218838;
        }
        table {
            width: 100%;
            margin-top: 10px;
            border-collapse: collapse;
        }
        th, td {
            padding: 10px;
            text-align: left;
            border-bottom: 1px solid #ddd;
        }
        .delete-btn {
            background-color: #dc3545;
            color: white;
            border: none;
            padding: 5px 10px;
            cursor: pointer;
            border-radius: 5px;
        }
        .delete-btn:hover {
            background-color: #c82333;
        }
        .highlight {
            font-weight: bold;
            color: red;
        }
    </style>
</head>
<body>

    <div class="container">
        <h2>Organizador de Gastos</h2>
        
        <label for="nome">Nome do Gasto:</label>
        <input type="text" id="nome" placeholder="Ex: Mercado, Academia">

        <label for="valor">Valor (R$):</label>
        <input type="number" id="valor" placeholder="Ex: 50.00">

        <label for="categoria">Categoria:</label>
        <select id="categoria">
            <option value="Alimentação">Alimentação</option>
            <option value="Transporte">Transporte</option>
            <option value="Lazer">Lazer</option>
            <option value="Moradia">Moradia</option>
            <option value="Saúde">Saúde</option>
            <option value="Educação">Educação</option>
            <option value="Entretenimento">Entretenimento</option>
            <option value="Investimentos">Investimentos</option>
            <option value="Outros">Outros</option>
        </select>

        <button onclick="adicionarGasto()">Adicionar Gasto</button>

        <h3>Filtrar Gastos por Data</h3>
        <input type="date" id="dataFiltro">
        <button onclick="filtrarGastos()">Exibir Gastos</button>
        <button onclick="gerarRelatorio()">Gerar Relatório</button>

        <h3>Relatório de Gastos</h3>
        <table>
            <thead>
                <tr>
                    <th>Data</th>
                    <th>Hora</th>
                    <th>Nome</th>
                    <th>Valor (R$)</th>
                    <th>Categoria</th>
                    <th>Ação</th>
                </tr>
            </thead>
            <tbody id="lista-gastos"></tbody>
        </table>

        <h3>Relatório Detalhado</h3>
        <div id="relatorio"></div>
    </div>

    <script>
        document.addEventListener("DOMContentLoaded", carregarGastos);

        function adicionarGasto() {
            let nome = document.getElementById("nome").value;
            let valor = document.getElementById("valor").value;
            let categoria = document.getElementById("categoria").value;
            let data = new Date();
            let dataFormatada = data.toISOString().split('T')[0]; 
            let horaFormatada = data.toLocaleTimeString();

            if (nome === "" || valor === "") {
                alert("Por favor, preencha todos os campos.");
                return;
            }

            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
            gastos.push({ dataFormatada, horaFormatada, nome, valor, categoria });

            localStorage.setItem("gastos", JSON.stringify(gastos));

            document.getElementById("nome").value = "";
            document.getElementById("valor").value = "";
            document.getElementById("categoria").value = "Alimentação";

            carregarGastos();
        }

        function carregarGastos() {
            let listaGastos = document.getElementById("lista-gastos");
            listaGastos.innerHTML = "";
            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];

            gastos.forEach((gasto, index) => {
                adicionarLinhaTabela(gasto, index);
            });
        }

        function filtrarGastos() {
            let dataEscolhida = document.getElementById("dataFiltro").value;
            if (!dataEscolhida) {
                alert("Por favor, selecione uma data.");
                return;
            }

            let listaGastos = document.getElementById("lista-gastos");
            listaGastos.innerHTML = "";
            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];

            let gastosFiltrados = gastos.filter(gasto => gasto.dataFormatada === dataEscolhida);

            if (gastosFiltrados.length === 0) {
                listaGastos.innerHTML = "<tr><td colspan='6' style='text-align: center;'>Nenhum gasto encontrado para esta data.</td></tr>";
            } else {
                gastosFiltrados.forEach((gasto, index) => {
                    adicionarLinhaTabela(gasto, index);
                });
            }
        }

        function gerarRelatorio() {
            let dataEscolhida = document.getElementById("dataFiltro").value;
            if (!dataEscolhida) {
                alert("Por favor, selecione uma data.");
                return;
            }

            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
            let gastosFiltrados = gastos.filter(gasto => gasto.dataFormatada === dataEscolhida);

            if (gastosFiltrados.length === 0) {
                document.getElementById("relatorio").innerHTML = "<p>Nenhum gasto encontrado para esta data.</p>";
                return;
            }

            let totalGasto = 0;
            let categoriaResumo = {};
            let maiorGasto = 0;
            let menorGasto = Number.MAX_VALUE;

            gastosFiltrados.forEach(gasto => {
                let valor = parseFloat(gasto.valor);
                totalGasto += valor;
                if (valor > maiorGasto) maiorGasto = valor;
                if (valor < menorGasto) menorGasto = valor;

                if (!categoriaResumo[gasto.categoria]) {
                    categoriaResumo[gasto.categoria] = { total: 0, quantidade: 0 };
                }
                categoriaResumo[gasto.categoria].total += valor;
                categoriaResumo[gasto.categoria].quantidade++;
            });

            let relatorioHTML = `<p><strong>Total Gasto:</strong> R$ ${totalGasto.toFixed(2)}</p>`;
            for (let categoria in categoriaResumo) {
                relatorioHTML += `<p><strong>${categoria}:</strong> R$ ${categoriaResumo[categoria].total.toFixed(2)} (${categoriaResumo[categoria].quantidade} transações, média de R$ ${(categoriaResumo[categoria].total / categoriaResumo[categoria].quantidade).toFixed(2)})</p>`;
            }
            relatorioHTML += `<p><strong>Maior Gasto:</strong> R$ ${maiorGasto.toFixed(2)}</p>`;
            relatorioHTML += `<p><strong>Menor Gasto:</strong> R$ ${menorGasto.toFixed(2)}</p>`;

            document.getElementById("relatorio").innerHTML = relatorioHTML;
        }
    </script>

</body>
</html>
