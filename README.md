
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Calculateur de Ressources - Alchemy Factory</title>
    <style>
        /* STYLE CSS */
        body {
            font-family: 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
            background-color: #0f172a;
            color: #f8fafc;
            margin: 0;
            padding: 20px;
            display: flex;
            flex-direction: column;
            align-items: center;
        }

        h1 { color: #10b981; margin-bottom: 10px; }
        .description { color: #94a3b8; margin-bottom: 30px; text-align: center; }

        .container {
            width: 100%;
            max-width: 1000px;
        }

        .grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(130px, 1fr));
            gap: 15px;
            margin-bottom: 40px;
        }

        .card {
            background: #1e293b;
            border: 1px solid #334155;
            border-radius: 12px;
            padding: 15px;
            text-align: center;
            transition: transform 0.2s, border-color 0.2s;
        }

        .card:hover {
            transform: translateY(-5px);
            border-color: #10b981;
        }

        .card img {
            width: 50px;
            height: 50px;
            object-fit: contain;
            background: #0f172a;
            border-radius: 8px;
            padding: 5px;
            margin-bottom: 10px;
        }

        .item-name {
            display: block;
            font-size: 0.9em;
            font-weight: bold;
            margin-bottom: 10px;
            height: 2.5em;
            overflow: hidden;
        }

        input[type="number"] {
            width: 80%;
            background: #0f172a;
            border: 1px solid #475569;
            color: white;
            padding: 8px;
            border-radius: 6px;
            text-align: center;
            font-size: 1em;
        }

        #results {
            background: #1e293b;
            border-radius: 12px;
            padding: 25px;
            border: 2px solid #10b981;
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.5);
        }

        .result-grid {
            display: grid;
            grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
            gap: 10px;
            margin-top: 15px;
        }

        .result-item {
            background: #334155;
            padding: 10px 15px;
            border-radius: 8px;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        .qty-badge {
            background: #10b981;
            color: #0f172a;
            font-weight: bold;
            padding: 2px 8px;
            border-radius: 4px;
        }

        footer { margin-top: 50px; color: #64748b; font-size: 0.8em; }
    </style>
</head>
<body>

    <div class="container">
        <h1>Alchemy Factory Calculator</h1>
        <p class="description">Sélectionnez les objets à fabriquer pour calculer les matières premières nécessaires.</p>

        <div class="grid" id="items-grid">
            </div>

        <div id="results">
            <h3>📦 Liste de courses (Ressources de base)</h3>
            <div id="total-display" class="result-grid">
                <span style="color: #64748b;">Modifiez une quantité pour voir le calcul...</span>
            </div>
        </div>
    </div>

    <footer>Expert Codage AI - Outil de calcul pour Alchemy Factory</footer>

<script>
/**
 * DONNÉES DU JEU
 * Ajoutez ici les nouvelles recettes et icônes du Wiki
 */
const gameData = {
    // Schéma : "Produit fini": { "Ingrédient": Quantité, ... }
    recipes: {
        "Planche": { "Bûche": 1 },
        "Engrenage Bois": { "Planche": 2 },
        "Corde": { "Lin": 3 },
        "Chaux Vive": { "Pierre": 2, "Charbon": 1 },
        "Brique": { "Argile": 2, "Sable": 1 },
        "Acide": { "Soufre": 2, "Eau": 5 },
        "Batterie": { "Plomb": 3, "Acide": 2 },
        "Potion de Soin": { "Sauge": 2, "Fiole": 1, "Eau": 1 }
    },
    // Noms des fichiers sur le Wiki (ou placeholders si non trouvés)
    icons: {
        "Bûche": "https://img.icons8.com/color/48/log.png",
        "Planche": "https://img.icons8.com/color/48/wood.png",
        "Engrenage Bois": "https://img.icons8.com/color/48/settings.png",
        "Lin": "https://img.icons8.com/color/48/flower.png",
        "Corde": "https://img.icons8.com/color/48/rope.png",
        "Pierre": "https://img.icons8.com/color/48/rock.png",
        "Charbon": "https://img.icons8.com/color/48/coal.png",
        "Chaux Vive": "https://img.icons8.com/color/48/powder.png",
        "Argile": "https://img.icons8.com/color/48/clay.png",
        "Sable": "https://img.icons8.com/color/48/sand-timer.png",
        "Brique": "https://img.icons8.com/color/48/brick.png",
        "Soufre": "https://img.icons8.com/color/48/sulfur.png",
        "Eau": "https://img.icons8.com/color/48/water.png",
        "Acide": "https://img.icons8.com/color/48/test-tube.png",
        "Plomb": "https://img.icons8.com/color/48/lead.png",
        "Batterie": "https://img.icons8.com/color/48/battery.png",
        "Sauge": "https://img.icons8.com/color/48/plant.png",
        "Fiole": "https://img.icons8.com/color/48/glass-bottle.png",
        "Potion de Soin": "https://img.icons8.com/color/48/heart-potion.png"
    }
};

const grid = document.getElementById('items-grid');
const display = document.getElementById('total-display');

// 1. Initialisation de l'interface
function init() {
    Object.keys(gameData.icons).sort().forEach(item => {
        const card = document.createElement('div');
        card.className = 'card';
        card.innerHTML = `
            <img src="${gameData.icons[item]}" alt="${item}" onerror="this.src='https://via.placeholder.com/50?text=?'">
            <span class="item-name">${item}</span>
            <input type="number" id="qty-${item}" value="0" min="0" oninput="calculate()">
        `;
        grid.appendChild(card);
    });
}

// 2. Logique de calcul récursif
function calculate() {
    let totals = {};

    function breakdown(item, amount) {
        // Si l'objet a une recette, on le décompose
        if (gameData.recipes[item]) {
            for (let [ingredient, qtyPerItem] of Object.entries(gameData.recipes[item])) {
                breakdown(ingredient, qtyPerItem * amount);
            }
        } else {
            // Sinon, c'est une ressource de base, on l'ajoute au total
            totals[item] = (totals[item] || 0) + amount;
        }
    }

    // On parcourt tous les inputs pour voir ce que l'utilisateur veut fabriquer
    Object.keys(gameData.icons).forEach(item => {
        const input = document.getElementById(`qty-${item}`);
        const quantity = parseInt(input.value) || 0;
        if (quantity > 0) {
            breakdown(item, quantity);
        }
    });

    renderResults(totals);
}

// 3. Affichage des résultats
function renderResults(totals) {
    const entries = Object.entries(totals);
    
    if (entries.length === 0) {
        display.innerHTML = '<span style="color: #64748b;">Modifiez une quantité pour voir le calcul...</span>';
        return;
    }

    display.innerHTML = entries
        .sort((a, b) => b[1] - a[1]) // Trie par plus grosse quantité
        .map(([res, qty]) => `
            <div class="result-item">
                <span>${res}</span>
                <span class="qty-badge">${qty}</span>
            </div>
        `).join('');
}

// Lancement
init();
</script>

</body>
</html>
