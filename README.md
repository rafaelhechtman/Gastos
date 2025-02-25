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
        h2 {
            text-align: center;
        }
        .container {
            max-width: 600px;
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
    </style>
</head>
<body>

    <div class="container">
        <h2>Organizador de Gastos</h2>
        <label for="nome">Nome do Gasto:</label>
        <input type="text" id="nome" placeholder="Ex: Uber, Mercado">

        <label for="valor">Valor (R$):</label>
        <input type="number" id="valor" placeholder="Ex: 50.00">

        <label for="categoria">Categoria:</label>
        <select id="categoria">
            <option value="Alimentação">Alimentação</option>
            <option value="Transporte">Transporte</option>
            <option value="Lazer">Lazer</option>
            <option value="Outros">Outros</option>
        </select>

        <button onclick="adicionarGasto()">Adicionar Gasto</button>

        <h3>Lista de Gastos</h3>
        <table>
            <thead>
                <tr>
                    <th>Nome</th>
                    <th>Valor (R$)</th>
                    <th>Categoria</th>
                    <th>Ação</th>
                </tr>
            </thead>
            <tbody id="lista-gastos">
                <!-- Os gastos aparecerão aqui -->
            </tbody>
        </table>
    </div>

    <script>
        document.addEventListener("DOMContentLoaded", carregarGastos);

        function adicionarGasto() {
            let nome = document.getElementById("nome").value;
            let valor = document.getElementById("valor").value;
            let categoria = document.getElementById("categoria").value;

            if (nome === "" || valor === "") {
                alert("Por favor, preencha todos os campos.");
                return;
            }

            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
            gastos.push({ nome, valor, categoria });

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
                let row = document.createElement("tr");

                row.innerHTML = `
                    <td>${gasto.nome}</td>
                    <td>R$ ${parseFloat(gasto.valor).toFixed(2)}</td>
                    <td>${gasto.categoria}</td>
                    <td><button class="delete-btn" onclick="removerGasto(${index})">Excluir</button></td>
                `;

                listaGastos.appendChild(row);
            });
        }

        function removerGasto(index) {
            let gastos = JSON.parse(localStorage.getItem("gastos")) || [];
            gastos.splice(index, 1);
            localStorage.setItem("gastos", JSON.stringify(gastos));
            carregarGastos();
        }
    </script>

</body>
</html>
