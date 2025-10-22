<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Saafi Shop — Shoes & Clothes</title>
  <style>
    :root{--brand:#1f6feb;--muted:#6b7280;--bg:#f8fafc}
    *{box-sizing:border-box}
    body{font-family:Inter,system-ui,Segoe UI,Roboto,"Helvetica Neue",Arial;margin:0;background:var(--bg);color:#111}
    header{background:white;box-shadow:0 2px 6px rgba(20,20,20,0.06);padding:14px 20px;display:flex;align-items:center;justify-content:space-between;gap:12px}
    .logo{font-weight:700;color:var(--brand)}
    .search{flex:1;margin:0 18px}
    .search input{width:100%;padding:8px 12px;border:1px solid #e5e7eb;border-radius:8px}
    .main{max-width:1100px;margin:24px auto;padding:0 18px}
    .grid{display:grid;grid-template-columns:repeat(auto-fit,minmax(220px,1fr));gap:18px}
    .card{background:white;border-radius:12px;padding:12px;box-shadow:0 6px 18px rgba(15,15,15,0.05);display:flex;flex-direction:column}
    .card img{width:100%;height:180px;object-fit:cover;border-radius:8px}
    .card h3{margin:10px 0 6px;font-size:16px}
    .price{font-weight:700}
    .muted{color:var(--muted);font-size:13px}
    .btn{background:var(--brand);color:white;padding:8px 12px;border-radius:8px;border:0;cursor:pointer}
    .row{display:flex;gap:8px;align-items:center}
    .filters{display:flex;gap:8px;margin-bottom:16px;align-items:center}
    .cart-btn{position:relative}
    .cart-count{position:absolute;top:-6px;right:-8px;background:#ef4444;color:white;border-radius:50%;padding:2px 6px;font-size:12px}
    /* Cart drawer */
    .drawer{position:fixed;right:18px;top:72px;width:340px;max-width:90%;background:white;border-radius:12px;box-shadow:0 20px 50px rgba(10,10,10,0.12);padding:14px;transform:translateX(120%);transition:transform .28s ease;z-index:40}
    .drawer.open{transform:translateX(0)}
    .drawer h4{margin:0 0 8px}
    .cart-item{display:flex;gap:10px;align-items:center;margin-bottom:10px}
    .cart-item img{width:60px;height:60px;object-fit:cover;border-radius:8px}
    .qty{display:flex;gap:6px;align-items:center}
    .footer{margin-top:12px;display:flex;justify-content:space-between;align-items:center}
    .checkout{width:100%;margin-top:10px}
    /* responsive */
    @media(max-width:600px){header{flex-wrap:wrap}.search{order:3;width:100%;margin:10px 0}}
  </style>
</head>
<body>
  <header>
    <div class="logo">Saafi<span style="color:#0ea5e9">Shop</span></div>
    <div class="search"><input id="search" placeholder="Search shoes, shirts, trousers..."/></div>
    <div class="row">
      <select id="category">
        <option value="all">All</option>
        <option value="shoes">Shoes</option>
        <option value="clothes">Clothes</option>
      </select>
      <button class="btn cart-btn" id="openCart">Cart <span class="cart-count" id="cartCount">0</span></button>
    </div>
  </header>

  <main class="main">
    <section class="filters">
      <div class="muted">Sort by:</div>
      <select id="sort">
        <option value="default">Recommended</option>
        <option value="price-asc">Price: Low → High</option>
        <option value="price-desc">Price: High → Low</option>
      </select>
      <div style="margin-left:auto;" class="muted">Free delivery over $50</div>
    </section>

    <section id="products" class="grid">
      <!-- Products injected by JS -->
    </section>
  </main>

  <!-- Cart drawer -->
  <aside class="drawer" id="cartDrawer" aria-hidden="true">
    <h4>Your Cart</h4>
    <div id="cartItems"></div>
    <div class="footer">
      <div class="muted">Total:</div>
      <div id="cartTotal" style="font-weight:700">$0.00</div>
    </div>
    <div class="checkout">
      <form id="checkoutForm">
        <input type="text" id="fullname" placeholder="Full name" required style="width:100%;padding:8px;margin-top:8px;border-radius:8px;border:1px solid #e5e7eb"/>
        <input type="tel" id="phone" placeholder="Phone or WhatsApp" required style="width:100%;padding:8px;margin-top:8px;border-radius:8px;border:1px solid #e5e7eb"/>
        <textarea id="address" placeholder="Delivery address" required style="width:100%;padding:8px;margin-top:8px;border-radius:8px;border:1px solid #e5e7eb"></textarea>
        <button class="btn" type="submit" style="width:100%;margin-top:8px">Place Order</button>
      </form>
    </div>
  </aside>

  <script>
    // Sample products data
    const products = [
      {id:1,title:'Classic White Sneakers',category:'shoes',price:29.99,img:'https://via.placeholder.com/600x400?text=White+Sneakers'},
      {id:2,title:'Running Sports Shoes',category:'shoes',price:49.99,img:'https://via.placeholder.com/600x400?text=Running+Shoes'},
      {id:3,title:'Black Leather Boots',category:'shoes',price:79.99,img:'https://via.placeholder.com/600x400?text=Leather+Boots'},
      {id:4,title:'Blue Denim Jeans',category:'clothes',price:39.99,img:'https://via.placeholder.com/600x400?text=Denim+Jeans'},
      {id:5,title:'Casual T-Shirt',category:'clothes',price:14.99,img:'https://via.placeholder.com/600x400?text=Casual+T-Shirt'},
      {id:6,title:'Formal Shirt',category:'clothes',price:24.99,img:'https://via.placeholder.com/600x400?text=Formal+Shirt'},
      {id:7,title:'Sporty Jacket',category:'clothes',price:59.99,img:'https://via.placeholder.com/600x400?text=Sporty+Jacket'},
    ];

    // Cart state
    const cart = {} // key: productId -> {product, qty}

    // DOM refs
    const productsEl = document.getElementById('products');
    const searchEl = document.getElementById('search');
    const categoryEl = document.getElementById('category');
    const sortEl = document.getElementById('sort');
    const openCartBtn = document.getElementById('openCart');
    const cartCountEl = document.getElementById('cartCount');
    const cartDrawer = document.getElementById('cartDrawer');
    const cartItemsEl = document.getElementById('cartItems');
    const cartTotalEl = document.getElementById('cartTotal');
    const checkoutForm = document.getElementById('checkoutForm');

    // Render functions
    function renderProducts(list){
      productsEl.innerHTML = '';
      list.forEach(p =>{
        const card = document.createElement('div'); card.className='card';
        card.innerHTML = `
          <img src="${p.img}" alt="${p.title}" />
          <h3>${p.title}</h3>
          <div class="muted">Category: ${p.category}</div>
          <div style="margin-top:auto;display:flex;justify-content:space-between;align-items:center">
            <div class="price">$${p.price.toFixed(2)}</div>
            <button class="btn" data-id="${p.id}">Add to cart</button>
          </div>
        `;
        productsEl.appendChild(card);
      });
    }

    function updateCartCount(){
      const totalQty = Object.values(cart).reduce((s,i)=>s+i.qty,0);
      cartCountEl.textContent = totalQty;
    }

    function renderCart(){
      cartItemsEl.innerHTML = '';
      const fragment = document.createDocumentFragment();
      let total = 0;
      for(const id in cart){
        const item = cart[id];
        total += item.product.price * item.qty;
        const div = document.createElement('div'); div.className='cart-item';
        div.innerHTML = `
          <img src="${item.product.img}" alt="${item.product.title}">
          <div style="flex:1">
            <div style="font-weight:600">${item.product.title}</div>
            <div class="muted">$${item.product.price.toFixed(2)}</div>
            <div class="qty">
              <button data-action="dec" data-id="${id}">-</button>
              <div style="min-width:20px;text-align:center">${item.qty}</div>
              <button data-action="inc" data-id="${id}">+</button>
              <button data-action="remove" data-id="${id}" style="margin-left:8px;color:#ef4444">Remove</button>
            </div>
          </div>
        `;
        fragment.appendChild(div);
      }
      cartItemsEl.appendChild(fragment);
      cartTotalEl.textContent = `$${total.toFixed(2)}`;
      updateCartCount();
    }

    // Add events
    productsEl.addEventListener('click', (e)=>{
      const btn = e.target.closest('button');
      if(!btn) return;
      const id = btn.getAttribute('data-id');
      if(!id) return;
      addToCart(Number(id));
    });

    cartItemsEl.addEventListener('click', (e)=>{
      const actionBtn = e.target.closest('button');
      if(!actionBtn) return;
      const id = actionBtn.getAttribute('data-id');
      const action = actionBtn.getAttribute('data-action');
      if(action==='inc') changeQty(id,1);
      if(action==='dec') changeQty(id,-1);
      if(action==='remove') removeItem(id);
    });

    function addToCart(id){
      const product = products.find(p=>p.id===id);
      if(!product) return;
      if(cart[id]) cart[id].qty +=1; else cart[id] = {product, qty:1};
      renderCart();
      openDrawer();
    }
    function changeQty(id,delta){
      if(!cart[id]) return;
      cart[id].qty += delta;
      if(cart[id].qty <=0) delete cart[id];
      renderCart();
    }
    function removeItem(id){ delete cart[id]; renderCart(); }

    // Search, filter, sort
    function getFiltered(){
      const q = searchEl.value.trim().toLowerCase();
      const cat = categoryEl.value;
      let list = products.filter(p => (cat==='all' || p.category===cat));
      if(q) list = list.filter(p=> p.title.toLowerCase().includes(q));
      const sort = sortEl.value;
      if(sort==='price-asc') list.sort((a,b)=>a.price-b.price);
      if(sort==='price-desc') list.sort((a,b)=>b.price-a.price);
      return list;
    }

    [searchEl,categoryEl,sortEl].forEach(el => el.addEventListener('change',()=>renderProducts(getFiltered())));
    searchEl.addEventListener('input',()=>renderProducts(getFiltered()));

    // Cart drawer controls
    openCartBtn.addEventListener('click', ()=>{
      cartDrawer.classList.toggle('open');
    });
    function openDrawer(){ cartDrawer.classList.add('open'); }

    // Checkout
    checkoutForm.addEventListener('submit',(e)=>{
      e.preventDefault();
      if(Object.keys(cart).length===0){ alert('Your cart is empty'); return; }
      const order = {
        name: document.getElementById('fullname').value,
        phone: document.getElementById('phone').value,
        address: document.getElementById('address').value,
        items: Object.values(cart).map(ci=>({id:ci.product.id,title:ci.product.title,qty:ci.qty,price:ci.product.price})),
        total: cartTotalEl.textContent
      };
      // In a real site, send `order` to server with fetch(). Here we just show a confirmation.
      console.log('Order placed', order);
      alert('Thanks '+order.name + '! Your order has been placed.\nWe will contact you on ' + order.phone + '.');
      // clear cart
      for(const k in cart) delete cart[k]; renderCart(); checkoutForm.reset(); cartDrawer.classList.remove('open');
    });

    // Initial render
    renderProducts(products);
    renderCart();
  </script>
</body>
</html>
