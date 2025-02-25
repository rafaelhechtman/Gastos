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
        .edit-btn {
            background-color: #ffc107;
            color: black;
            border: none;
            padding: 5px 10px;
            cursor: pointer;
            border-radius: 5px;
        }
        .edit-btn:hover {
            background-color: #e0a800;
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

        <h3>Selecionar Data para Relatório</h3>
        <input type="date" id="dataFiltro">
        <button onclick="gerarRelatorio()">Gerar Relatório</button>

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

            gerarRelatorio();
        }

        function carregarGastos() {
            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
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
            let relatorioHTML = `<p><strong>Total Gasto:</strong> R$ ${totalGasto.toFixed(2)}</p>`;
            let tabelaHTML = `<table><thead><tr><th>Nome</th><th>Valor (R$)</th><th>Categoria</th><th>Ação</th></tr></thead><tbody>`;

            gastosFiltrados.forEach((gasto, index) => {
                let valor = parseFloat(gasto.valor);
                totalGasto += valor;

                if (!categoriaResumo[gasto.categoria]) {
                    categoriaResumo[gasto.categoria] = 0;
                }
                categoriaResumo[gasto.categoria] += valor;

                tabelaHTML += `
                    <tr>
                        <td><input type="text" value="${gasto.nome}" id="edit-nome-${index}"></td>
                        <td><input type="number" value="${gasto.valor}" id="edit-valor-${index}"></td>
                        <td><select id="edit-categoria-${index}">
                            <option value="${gasto.categoria}" selected>${gasto.categoria}</option>
                            <option value="Alimentação">Alimentação</option>
                            <option value="Transporte">Transporte</option>
                            <option value="Lazer">Lazer</option>
                            <option value="Moradia">Moradia</option>
                            <option value="Saúde">Saúde</option>
                            <option value="Educação">Educação</option>
                            <option value="Entretenimento">Entretenimento</option>
                            <option value="Investimentos">Investimentos</option>
                            <option value="Outros">Outros</option>
                        </select></td>
                        <td>
                            <button class="edit-btn" onclick="editarGasto(${index})">Salvar</button>
                            <button class="delete-btn" onclick="removerGasto(${index})">Excluir</button>
                        </td>
                    </tr>`;
            });

            tabelaHTML += `</tbody></table>`;
            document.getElementById("relatorio").innerHTML = relatorioHTML + tabelaHTML;
        }

        function editarGasto(index) {
            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
            gastos[index].nome = document.getElementById(`edit-nome-${index}`).value;
            gastos[index].valor = document.getElementById(`edit-valor-${index}`).value;
            gastos[index].categoria = document.getElementById(`edit-categoria-${index}`).value;
            localStorage.setItem("gastos", JSON.stringify(gastos));
            gerarRelatorio();
        }

        function removerGasto(index) {
            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
            gastos.splice(index, 1);
            localStorage.setItem("gastos", JSON.stringify(gastos));
            gerarRelatorio();
        }
    </script>

</body>
</html>
