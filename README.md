<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Grocery Ordering Website</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f8ff;
            color: #333;
            margin: 0; padding: 0;
        }
        header {
            background-color: #87ceeb;
            padding: 10px 20px;
            color: white;
            display: flex;
            justify-content: space-between;
            align-items: center;
        }
        header div.menu-item {
            background-color: white;
            color: #333;
            border: 2px solid #333;
            border-radius: 8px;
            padding: 8px 12px;
            cursor: pointer;
            margin-left: 10px;
        }
        header div.menu-item:hover {
            background-color: #b0e0f9;
        }
        main {
            padding: 20px;
        }
        .category-list {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
            flex-wrap: wrap;
        }
        .category-list button {
            padding: 10px 15px;
            border: none;
            background-color: #87ceeb;
            color: white;
            border-radius: 5px;
            cursor: pointer;
            font-weight: bold;
        }
        .category-list button:hover {
            background-color: #00bfff;
        }
        .order-form {
            background: white;
            padding: 15px;
            border-radius: 10px;
            max-width: 600px;
            box-shadow: 0 0 10px #ccc;
        }
        .order-form label {
            display: block;
            margin-top: 10px;
            font-weight: bold;
        }
        .order-form input, 
        .order-form textarea {
            width: 100%;
            padding: 8px;
            margin-top: 5px;
            border-radius: 5px;
            border: 1px solid #ccc;
            font-size: 1em;
        }
        .order-form button {
            background-color: #00bfff;
            color: white;
            border: none;
            margin-top: 15px;
            padding: 10px 20px;
            font-weight: bold;
            cursor: pointer;
            border-radius: 6px;
        }
        .order-form button:hover {
            background-color: #0077b6;
        }
        .hidden {
            display: none;
        }
        .info-display {
            margin-top: 20px;
            font-style: italic;
            color: #666;
        }
    </style>
</head>
<body>
<header>
    <div class="menu-item" onclick="showHome()">Home</div>
    <div class="menu-item" onclick="openCamera()">Camera</div>
    <div class="menu-item" onclick="openGoogleMaps()">Google Maps</div>
    <div class="menu-item" onclick="showOrders()">Orders</div>
</header>
<main>
    <div id="homeScreen">
        <h1>Welcome to Grocery Ordering</h1>
        <div class="category-list">
            <button onclick="openCategory('groceries')">Groceries</button>
            <button onclick="openCategory('vegetables')">Vegetables</button>
            <button onclick="openCategory('milk')">Milk</button>
            <button onclick="openCategory('fruits')">Fruits</button>
            <button onclick="openCategory('flowers')">Flowers</button>
            <button onclick="openCategory('pharmacy')">Pharmacy</button>
        </div>
    </div>

    <div id="categoryScreen" class="hidden">
        <h2 id="categoryTitle"></h2>
        <form id="orderForm" class="order-form">
            <label for="nameInput">Name:</label>
            <input type="text" id="nameInput" required />

            <label for="phoneInput">Phone Number:</label>
            <input type="tel" id="phoneInput" required pattern="\\d{10}" title="Enter 10 digit phone number" />

            <label for="orderItemInput">Order Item:</label>
            <textarea id="orderItemInput" rows="3" required></textarea>

            <label for="deliveryAddressInput">Delivery Address:</label>
            <textarea id="deliveryAddressInput" rows="3" required></textarea>

            <button type="submit">Place Order via WhatsApp</button>
        </form>
        <div id="infoDisplay" class="info-display"></div>
        <button onclick="goHome()" style="margin-top: 20px;">Back to Home</button>
    </div>

    <div id="ordersScreen" class="hidden">
        <h2>Your Orders</h2>
        <div id="ordersList">No orders placed yet.</div>
        <button onclick="goHome()" style="margin-top: 20px;">Back to Home</button>
    </div>
</main>

<script>
    let currentCategory = '';
    let orders = [];

    function showHome() {
        document.getElementById('homeScreen').classList.remove('hidden');
        document.getElementById('categoryScreen').classList.add('hidden');
        document.getElementById('ordersScreen').classList.add('hidden');
        document.getElementById('orderForm').reset();
        document.getElementById('infoDisplay').innerText = '';
        currentCategory = '';
    }

    function openCategory(category) {
        currentCategory = category;
        document.getElementById('homeScreen').classList.add('hidden');
        document.getElementById('categoryScreen').classList.remove('hidden');
        document.getElementById('ordersScreen').classList.add('hidden');
        document.getElementById('categoryTitle').innerText = category.charAt(0).toUpperCase() + category.slice(1);
        document.getElementById('orderForm').reset();
        document.getElementById('infoDisplay').innerText = `Order your ${category} here.`;
    }

    function goHome() {
        showHome();
    }

    document.getElementById('orderForm').addEventListener('submit', function(event) {
        event.preventDefault();
        const name = document.getElementById('nameInput').value.trim();
        const phone = document.getElementById('phoneInput').value.trim();
        const orderItem = document.getElementById('orderItemInput').value.trim();
        const address = document.getElementById('deliveryAddressInput').value.trim();
        if (!name || !phone || !orderItem || !address) {
            alert('Please fill all fields before submitting.');
            return;
        }
        if (!/^\d{10}$/.test(phone)) {
            alert('Phone number must be 10 digits.');
            return;
        }
        // Compose WhatsApp message
        const message = `Name: ${name}%0APhone: ${phone}%0AOrder Item: ${orderItem}%0ADelivery Address: ${address}%0ACategory: ${currentCategory}`;
        const whatsappUrl = `https://wa.me/?text=${message}`;
        // Save order locally
        orders.push({ name, phone, orderItem, address, category: currentCategory, date: new Date().toLocaleString() });
        alert('Order placed successfully via WhatsApp.');
        // Open WhatsApp URL
        window.open(whatsappUrl, '_blank');
        showHome();
    });

    function showOrders() {
        document.getElementById('homeScreen').classList.add('hidden');
        document.getElementById('categoryScreen').classList.add('hidden');
        document.getElementById('ordersScreen').classList.remove('hidden');
        
        let ordersList = document.getElementById('ordersList');
        if (orders.length === 0) {
            ordersList.innerText = 'No orders placed yet.';
            return;
        }
        ordersList.innerHTML = '';
        orders.forEach((order, index) => {
            let orderDiv = document.createElement('div');
            orderDiv.style.border = '1px solid #ccc';
            orderDiv.style.padding = '10px';
            orderDiv.style.margin = '10px 0';
            orderDiv.innerHTML = `<strong>${order.category.toUpperCase()}</strong> - ${order.date}<br>Name: ${order.name}<br>Phone: ${order.phone}<br>Order: ${order.orderItem}<br>Address: ${order.address}`;
            ordersList.appendChild(orderDiv);
        });
    }

    function openCamera() {
        alert('Camera feature to be implemented as per device compatibility.');
    }

    function openGoogleMaps() {
        window.open('https://maps.google.com', '_blank');
    }

    showHome();
</script>
</body>
</html>
