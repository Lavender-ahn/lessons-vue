<template>
  <link rel="stylesheet" href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined:opsz,wght,FILL,GRAD@24,400,0,0&icon_names=shopping_cart" />
 
  <nav class="navbar navbar-expand-lg navbar-dark bg-white sticky-top .navbar.fixed-top">
    <div class="container-fluid">

      <a class="navbar-brand fw-bold" href="#">📚 Lesson Shop</a>


      <form class="d-flex flex-grow-1 me-3" @submit.prevent>
        <input v-model="query"
               class="form-control me-2"
               type="search" placeholder="Search lessons…"
               aria-label="Search"
               @input="debouncedSearch">
      </form>

      <select v-model="sortBy"
              class="form-select form-select-sm w-auto me-2">
        <option value="subject">Subject</option>
        <option value="location">Location</option>
        <option value="price">Price</option>
        <option value="space">Spaces Left</option>
      </select>
      <button class="btn btn-light btn-sm me-3" @click="asc = !asc">
        {{ asc ? '▲' : '▼' }}
      </button>


      <button class="btn btn-warning position-relative"
              data-bs-toggle="offcanvas" data-bs-target="#cart">
        <i class="bi bi-cart4"></i>
        <span v-if="cart.length"
              class="position-absolute top-0 start-100 translate-middle
                     badge rounded-pill bg-danger">
          {{ cart.length }}
        </span>
      </button>
    </div>
  </nav>

  <header class="hero d-flex align-items-center .hero-offset">
    <div class="container text-center text-white">
      <h1 class="display-5 fw-bold">Learn Anything, Anywhere</h1>
      <p class="lead">Affordable tutoring lessons delivered by experts.</p>
    </div>
  </header>


  <section class="container mt-5 px-4">
    <div class="row g-4">
      <div v-for="l in filteredSorted" :key="l._id" class="col-md-4">
        <div class="card h-100 shadow-sm">
          <img :src="`https://picsum.photos/seed/${l.subject}/400/200`"
               class="card-img-top" alt="..." />
          <div class="card-body">
            <h5 class="card-title">{{ l.subject }}</h5>
            <p class="card-text medium text-muted">
              <i class="bi bi-geo-alt"></i> {{ l.location }}<br>
              <i class="bi bi-cash"></i> £{{ l.price }}<br>
              <i class="bi bi-person"></i> Spaces: {{ l.space }}
            </p>
            <button class="btn btn-sm btn-primary w-100"
                    :disabled="l.space===0"
                    @click="add(l)">
              {{ l.space ? 'Add to Cart' : 'Sold Out' }}
            </button>
          </div>
        </div>
      </div>
    </div>
  </section>


  <div class="offcanvas offcanvas-end" tabindex="-1" id="cart">
    <div class="offcanvas-header">
      <h5 class="offcanvas-title">🛒 Cart</h5>
      <button type="button" class="btn-close" data-bs-dismiss="offcanvas"></button>
    </div>
    <div class="offcanvas-body">
      <ul class="list-group mb-3" v-if="cart.length">
        <li v-for="(c,i) in cart" :key="i" class="list-group-item d-flex justify-content-between">
          {{ c.subject }} – £{{ c.price }}
          <button class="btn btn-sm btn-outline-danger" @click="remove(i)">
            <i class="bi bi-trash"></i>
          </button>
        </li>
      </ul>
      <p v-else class="text-muted">Your cart is empty.</p>


      <button class="btn btn-success w-100" data-bs-toggle="modal"
              data-bs-target="#checkout" :disabled="!cart.length">
        Checkout
      </button>
    </div>
  </div>


  <div class="modal fade" id="checkout" tabindex="-1">
    <div class="modal-dialog">
      <form class="modal-content needs-validation" novalidate>
        <div class="modal-header">
          <h5 class="modal-title">Checkout</h5>
          <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
        </div>
        <div class="modal-body">
          <div class="mb-3">
            <label class="form-label">Name</label>
            <input v-model="name" class="form-control" required pattern="[A-Za-z ]+">
            <div class="invalid-feedback">Letters only</div>
          </div>
          <div class="mb-3">
            <label class="form-label">Phone</label>
            <input v-model="phone" class="form-control" required pattern="[0-9]+">
            <div class="invalid-feedback">Numbers only</div>
          </div>
        </div>
        <div class="modal-footer">
          <button class="btn btn-primary" :disabled="!formValid" @click="submitOrder">Pay £{{ total }}</button>
        </div>
      </form>
    </div>
  </div>


  <div class="toast-container position-fixed top-0 end-0 p-3">
    <div id="orderToast" class="toast align-items-center text-white bg-success"
        role="alert" aria-live="assertive" aria-atomic="true">
      <div class="d-flex">
        <div class="toast-body">
          🎉 Order placed successfully!
        </div>
        <button type="button" class="btn-close btn-close-white me-2 m-auto"
                data-bs-dismiss="toast"></button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed } from "vue";

const API = "https://lessons-api.onrender.com";   // your Render URL

export default {
  setup() {
    // state
    const lessons = ref([]);
    const cart    = ref([]);
    const sortBy  = ref("subject");
    const asc     = ref(true);
    const query   = ref("");
    const name    = ref("");
    const phone   = ref("");

    // fetch data
    (async () => {
      lessons.value = await fetch(`${API}/lessons`).then(r=>r.json());
    })();

    // sort + search
    const filteredSorted = computed(() => {
      const regex = new RegExp(query.value, "i");
      return [...lessons.value]
        .filter(l => regex.test(l.subject) || regex.test(l.location) ||
                     regex.test(String(l.price)) || regex.test(String(l.space)))
        .sort((a,b)=>{
          const k = sortBy.value;
          return (asc.value ? 1 : -1) * ((a[k] > b[k]) ? 1 : -1);
        });
    });

    // cart ops
    const add = l => { l.space--; cart.value.push(l); };
    const remove = i => { const l=cart.value.splice(i,1)[0]; l.space++; };

    // checkout validation
    const total = computed(() => cart.value.reduce((s,l)=>s+l.price,0));
    const formValid = computed(() =>
      /^[A-Za-z ]+$/.test(name.value) && /^[0-9]+$/.test(phone.value));

    
    async function submitOrder() {
      if (!formValid.value) return;

      // 1. send order + update spaces
      const body = { name: name.value,
                    phone: phone.value,
                    lessons: cart.value.map(l => l._id),
                    total: total.value,
                    createdAt: new Date()
      };
      await fetch(`${API}/orders`, {method:'POST',
            headers:{'Content-Type':'application/json'}, body:JSON.stringify(body)});
      for (const l of cart.value) {
        await fetch(`${API}/lessons/${l._id}`, {method:'PUT',
              headers:{'Content-Type':'application/json'}, body:JSON.stringify({space:l.space})});
      }

      // 2. close checkout modal & cart off‑canvas
      bootstrap.Modal.getInstance(document.getElementById('checkout'))?.hide();
      bootstrap.Offcanvas.getInstance(document.getElementById('cart'))?.hide();

      // 3. show success toast
      const toast = new bootstrap.Toast(document.getElementById('orderToast'), { delay: 5000 });
      toast.show();

      // 4. reset state & refresh lesson list after toast hides
      cart.value = []; name.value=''; phone.value='';
      toast._element.addEventListener('hidden.bs.toast', () => {
        fetchLessons();          // re‑pull lessons so spaces reset
      });

  }

    // search debounce (270 ms)
    let t; const debouncedSearch = () => { clearTimeout(t); t = setTimeout(()=>{},270); };

    return { lessons, cart, sortBy, asc, query,
             filteredSorted, add, remove, name, phone,
             total, formValid, debouncedSearch };
  }
};
</script>

<style>
.navbar.fixed-top {
  width: 100vw;    
  left: 0;         
  right: 0;
  padding-top: .25rem;      
  padding-bottom: .25rem;
}
/* navbar height helper */
.mt-navbar { margin-top: 5px; }  /* <= default bootstrap navbar height */

.hero{
  height: 260px;
  background: url('https://picsum.photos/1200/400?blur=2') center / cover no-repeat;
}
.hero .container{ text-shadow: 0 2px 4px rgba(0,0,0,.6); }

body{ background:#f8f9fa }
.card-title{ font-weight:600 }
</style>


<!-- <style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style> -->
