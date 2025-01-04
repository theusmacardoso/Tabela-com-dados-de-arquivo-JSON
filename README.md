# Tabela-com-dados-de-arquivo-JSON
Criei uma tabela em javascript onde importei alguns dados de 5 arquivos JSON‚ formatando de forma que mostrasse 6 dados que selecionei.
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Tabela de Transações </title>
  <style>
    table {
      width: 100%; /* Área de utilização da tabela */
      border-collapse: collapse; /* Estilo das bordas */
    }
    th, td {
      border: 1px solid black;
      padding: 8px;
      text-align: center;
    }
    th {
      background-color: #f2f2f2;
    }
    .status-succeeded {
      color: green;
      font-weight: bold;
    }
    .status-failed {
      color: red;
      font-weight: bold;
    }
    .status-pending {
      color: orange;
      font-weight: bold;
    }
    .pagination {
      margin-top: 20px;
      text-align: center;
    }
    .pagination button {
      padding: 10px 15px;
      margin: 5px;
      border: 1px solid #ccc;
      background-color: #f2f2f2;
      cursor: pointer;
    }
    .pagination button.active {
      font-weight: bold;
      background-color: #ddd;
    }
  </style>
</head>
<body>
  <h1>Lista de Transações </h1>
  <table>
    <thead>
      <tr>
        <th>ID do Usuário</th>
        <th>Tipo de Operação</th>
        <th>Valor da Transação</th>
        <th>Status da Transação</th>
        <th>Marca do Cartão</th>
        <th>Data do Evento</th>
      </tr>
    </thead>
    <tbody id="table-body">
      <!-- As linhas serão geradas dinamicamente -->
    </tbody>
  </table>

  <div class="pagination" id="pagination">
    <!-- Botões para página -->
  </div>

  <script>
    // Lista dos arquivos JSON
    const jsonFiles = [
      'transaction_part_1.json',
      'transaction_part_2.json',
      'transaction_part_3.json',
      'transaction_part_4.json',
      'transaction_part_5.json'
    ];

    const rowsPerPage = 500; // Quantidade de linahs por página
    let currentPage = 1; // Página inicial
    let allTransactions = []; 

    async function fetchJson(file) {
      try {
        const response = await fetch(file);
        return await response.json();
      } catch (error) {
        console.error(`Erro ao carregar o arquivo ${file}:`, error);
        return [];
      }
    }

    async function loadAllTransactions() {

      for (const file of jsonFiles) {
        const transactions = await fetchJson(file);
        allTransactions.push(...transactions);
      }

      allTransactions.sort((a, b) => new Date(a.event_date) - new Date(b.event_date));

      renderTable();
      renderPagination();
    }

    function renderTable() {
      const tbody = document.getElementById('table-body');
      tbody.innerHTML = ''; // Limpar a tabela

      const startIndex = (currentPage - 1) * rowsPerPage;
      const endIndex = Math.min(startIndex + rowsPerPage, allTransactions.length);

      for (let i = startIndex; i < endIndex; i++) {
        const transaction = allTransactions[i];
        const row = document.createElement('tr');
        row.innerHTML = `
          <td>${transaction.uid || 'N/A'}</td>
          <td>${transaction.operation_type || 'N/A'}</td>
          <td>R$ ${parseFloat(transaction.amount).toFixed(2) || '0.00'}</td>
          <td class="status-${transaction.status.toLowerCase()}">${transaction.status || 'N/A'}</td>
          <td>${transaction.card_brand || 'N/A'}</td>
         <td>${transaction.event_date ? new Date(transaction.event_date).toLocaleString('pt-BR', { hour12: false }) : 'N/A'}</td>
        `;
        tbody.appendChild(row);
      }
    }

    function renderPagination() {
      const pagination = document.getElementById('pagination');
      pagination.innerHTML = ''; // Limpar os botões

      const totalPages = Math.ceil(allTransactions.length / rowsPerPage);

      for (let i = 1; i <= totalPages; i++) {
        const button = document.createElement('button');
        button.textContent = i;
        button.classList.toggle('active', i === currentPage);
        button.addEventListener('click', () => {
          currentPage = i;
          renderTable();
          renderPagination();
        });
        pagination.appendChild(button);
      }
    }

    loadAllTransactions();
  </script>
</body>
</html>
