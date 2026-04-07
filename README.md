# geofinanzas<!DOCTYPE html>
<html lang="es">
<head>
    <meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
<meta name="theme-color" content="#4361ee">
<link rel="apple-touch-icon" href="https://via.placeholder.com/192">
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>GeoFinanzas - Tu Gestor Personal</title>
    <style>
        /* CSS: DISEÑO Y ESTILOS */
        :root {
            --primary: #4361ee;
            --success: #2ecc71;
            --danger: #e74c3c;
            --dark: #2d3436;
            --light: #f8f9fa;
            --shadow: 0 4px 6px rgba(0,0,0,0.1);
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f2f5;
            margin: 0;
            padding: 20px;
            color: var(--dark);
        }

        .container {
            max-width: 800px;
            margin: 0 auto;
        }

        header {
            text-align: center;
            margin-bottom: 30px;
        }

        /* Tarjetas de Categoría */
        .category-card {
            background: white;
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: var(--shadow);
            border-left: 5px solid var(--primary);
        }

        .category-header {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 15px;
        }

        .balance {
            font-size: 1.5rem;
            font-weight: bold;
        }

        .positive { color: var(--success); }
        .negative { color: var(--danger); }

        /* Historial de movimientos */
        .history {
            list-style: none;
            padding: 0;
            max-height: 200px;
            overflow-y: auto;
            border-top: 1px solid #eee;
        }

        .movement-item {
            display: flex;
            justify-content: space-between;
            padding: 8px 0;
            border-bottom: 1px solid #f1f1f1;
            font-size: 0.9rem;
        }

        /* Botones */
        .btn {
            padding: 10px 15px;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            font-weight: 600;
            transition: opacity 0.2s;
        }

        .btn:active { transform: scale(0.98); }

        .btn-main { background: var(--primary); color: white; width: 100%; margin-bottom: 20px; font-size: 1rem; }
        .btn-add { background: var(--success); color: white; border-radius: 50%; width: 40px; height: 40px; font-size: 20px; }
        .btn-delete { background: none; color: var(--danger); font-size: 0.8rem; }

        /* Modal (Formularios flotantes) */
        .modal {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0,0,0,0.5);
            align-items: center;
            justify-content: center;
            z-index: 100;
        }

        .modal-content {
            background: white;
            padding: 25px;
            border-radius: 12px;
            width: 90%;
            max-width: 400px;
        }

        .form-group { margin-bottom: 15px; }
        .form-group label { display: block; margin-bottom: 5px; }
        .form-group input, .form-group select {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccc;
            border-radius: 6px;
            box-sizing: border-box;
        }

        .actions { display: flex; gap: 10px; margin-top: 20px; }
    </style>
</head>
<body>

<div class="container">
    <header>
        <h1>💰 Mis Finanzas</h1>
        <button class="btn btn-main" onclick="openModal('categoryModal')">+ Nueva Categoría</button>
    </header>

    <div id="categoriesContainer">
        </div>
</div>

<div id="categoryModal" class="modal">
    <div class="modal-content">
        <h3>Nueva Categoría</h3>
        <div class="form-group">
            <label>Nombre:</label>
            <input type="text" id="catName" placeholder="Ej: Comida, Ocio...">
        </div>
        <div class="actions">
            <button class="btn btn-main" onclick="saveCategory()">Guardar</button>
            <button class="btn" style="background:#ccc" onclick="closeModal('categoryModal')">Cancelar</button>
        </div>
    </div>
</div>

<div id="movementModal" class="modal">
    <div class="modal-content">
        <h3 id="movTitle">Añadir Movimiento</h3>
        <input type="hidden" id="currentCatId">
        <div class="form-group">
            <label>Tipo:</label>
            <select id="movType">
                <option value="ingreso">Ingreso (+)</option>
                <option value="gasto">Gasto (-)</option>
            </select>
        </div>
        <div class="form-group">
            <label>Cantidad (€):</label>
            <input type="number" id="movAmount" min="0.01" step="0.01" placeholder="0.00">
        </div>
        <div class="form-group">
            <label>Fecha:</label>
            <input type="date" id="movDate">
        </div>
        <div class="form-group">
            <label>Descripción:</label>
            <input type="text" id="movDesc" placeholder="Opcional">
        </div>
        <div class="actions">
            <button class="btn btn-main" onclick="saveMovement()">Registrar</button>
            <button class="btn" style="background:#ccc" onclick="closeModal('movementModal')">Cancelar</button>
        </div>
    </div>
</div>

<script>
    /* JAVASCRIPT: LÓGICA DE NEGOCIO */

    // Estado global de la aplicación
    let categories = JSON.parse(localStorage.getItem('finanzas_data')) || [];

    // --- Funciones de Utilidad ---
    function saveToStorage() {
        localStorage.setItem('finanzas_data', JSON.stringify(categories));
        render();
    }

    function openModal(id, catId = null) {
        document.getElementById(id).style.display = 'flex';
        if(catId !== null) {
            document.getElementById('currentCatId').value = catId;
            document.getElementById('movDate').valueAsDate = new Date();
        }
    }

    function closeModal(id) {
        document.getElementById(id).style.display = 'none';
        // Limpiar inputs
        const inputs = document.querySelectorAll(`#${id} input`);
        inputs.forEach(i => i.value = '');
    }

    // --- Gestión de Categorías ---
    function saveCategory() {
        const name = document.getElementById('catName').value.trim();
        if(!name) return alert("Escribe un nombre");

        const newCat = {
            id: Date.now(),
            name: name,
            movements: []
        };

        categories.push(newCat);
        saveToStorage();
        closeModal('categoryModal');
    }

    function deleteCategory(id) {
        if(confirm("¿Estás seguro de eliminar esta categoría y todos sus movimientos?")) {
            categories = categories.filter(c => c.id !== id);
            saveToStorage();
        }
    }

    // --- Gestión de Movimientos ---
    function saveMovement() {
        const catId = parseInt(document.getElementById('currentCatId').value);
        const type = document.getElementById('movType').value;
        const amount = parseFloat(document.getElementById('movAmount').value);
        const date = document.getElementById('movDate').value;
        const desc = document.getElementById('movDesc').value || (type === 'ingreso' ? 'Ingreso' : 'Gasto');

        // Validaciones
        if(isNaN(amount) || amount <= 0) return alert("Introduce una cantidad válida mayor a 0");
        if(!date) return alert("Selecciona una fecha");

        const movement = {
            id: Date.now(),
            type,
            amount,
            date,
            desc
        };

        const catIndex = categories.findIndex(c => c.id === catId);
        categories[catIndex].movements.push(movement);
        
        // Ordenar movimientos por fecha (descendente)
        categories[catIndex].movements.sort((a, b) => new Date(b.date) - new Date(a.date));

        saveToStorage();
        closeModal('movementModal');
    }

    function deleteMovement(catId, movId) {
        if(confirm("¿Eliminar este registro?")) {
            const catIndex = categories.findIndex(c => c.id === catId);
            categories[catIndex].movements = categories[catIndex].movements.filter(m => m.id !== movId);
            saveToStorage();
        }
    }

    // --- Renderizado de la Interfaz ---
    function render() {
        const container = document.getElementById('categoriesContainer');
        container.innerHTML = '';

        categories.forEach(cat => {
            // Calcular balance
            const balance = cat.movements.reduce((acc, mov) => {
                return mov.type === 'ingreso' ? acc + mov.amount : acc - mov.amount;
            }, 0);

            const card = document.createElement('div');
            card.className = 'category-card';
            card.innerHTML = `
                <div class="category-header">
                    <div>
                        <h2 style="margin:0">${cat.name}</h2>
                        <div class="balance ${balance >= 0 ? 'positive' : 'negative'}">
                            ${balance.toFixed(2)}€
                        </div>
                    </div>
                    <div style="display:flex; gap:10px; align-items:center;">
                        <button class="btn-delete" onclick="deleteCategory(${cat.id})">Eliminar Cat.</button>
                        <button class="btn-add" onclick="openModal('movementModal', ${cat.id})">+</button>
                    </div>
                </div>
                
                <ul class="history">
                    ${cat.movements.length === 0 ? '<li style="color:#999; font-size:0.8rem">Sin movimientos aún</li>' : ''}
                    ${cat.movements.map(mov => `
                        <li class="movement-item">
                            <div>
                                <strong>${mov.date}</strong> - ${mov.desc}
                            </div>
                            <div style="display:flex; align-items:center; gap:10px">
                                <span class="${mov.type === 'ingreso' ? 'positive' : 'negative'}">
                                    ${mov.type === 'ingreso' ? '+' : '-'}${mov.amount.toFixed(2)}€
                                </span>
                                <button class="btn-delete" onclick="deleteMovement(${cat.id}, ${mov.id})">✕</button>
                            </div>
                        </li>
                    `).join('')}
                </ul>
            `;
            container.appendChild(card);
        });
    }

    // Carga inicial
    render();
</script>

</body>
</html>
