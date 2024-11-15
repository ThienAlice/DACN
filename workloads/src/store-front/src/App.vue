<template>
  <TopNav :cartItemCount="cartItemCount"/>
  <router-view
    :products="products"
    :cartItems="cartItems"
    @addToCart="addToCart"
    @removeFromCart="removeFromCart"
    @submitOrder="submitOrder"
  ></router-view>
  <!-- Danh sách đánh giá hiển thị trực tiếp -->
  <div v-for="review in reviews" :key="review.text" class="review">
    <p>{{ review.text }}</p> <!-- XSS nếu review.text chứa mã JavaScript -->
  </div>
  <!-- Thêm ô nhập đánh giá và nút gửi -->
  <input v-model="newReview" placeholder="Write a review" />
  <button @click="addReview(newReview)">Submit Review</button>
</template>

<script>
import TopNav from './components/TopNav.vue'

export default {
  name: 'App',
  components: {
    TopNav
  },
  data() {
    return {
      cartItems: [],
      products: [],
      reviews: [], // Thêm danh sách đánh giá
      newReview: '' // Thêm biến lưu trữ đánh giá mới
    }
  },
  computed: {
    cartItemCount() {
      return this.cartItems.reduce((total, item) => {
        return total + item.quantity
      }, 0)
    }
  },
  mounted() {
    this.getProducts()
  },
  methods: {
    // SQL Injection - Lỗ hổng Injection
    getProducts() {
      const searchQuery = "test' OR '1'='1";  // Giả lập một payload SQL Injection
      fetch(`/products?search=${searchQuery}`)
        .then(response => response.json())
        .then(products => {
          console.log('success getting products')
          this.products = products
        })
        .catch(error => {
          console.log(error)
          alert('Error occurred while fetching products')
        })
    },

    // Broken Authentication - Lỗ hổng xác thực yếu
    login(username, password) {
      // Không kiểm tra mật khẩu một cách bảo mật (giả lập lỗ hổng)
      if (username === 'admin' && password === 'password') {
        this.isAuthenticated = true; // Không bảo vệ đúng cách
        alert('Logged in as admin');
      } else {
        alert('Invalid login');
      }
    },

    // Sensitive Data Exposure - Tiết lộ dữ liệu nhạy cảm
    fetchSensitiveData() {
      fetch('/api/userdata', {
        method: 'GET',
        headers: {
          'Authorization': 'Bearer ' + userToken // Giả lập không mã hóa đúng cách
        }
      })
    },

    // Security Misconfiguration - Cấu hình bảo mật sai
    accessUnsecuredAPI() {
      fetch('http://localhost:3000/openApi')  // Kết nối đến một API không được bảo vệ
        .then(response => response.json())
        .then(data => console.log(data))
        .catch(error => console.log('Failed to fetch data'));
    },

    // Cross-Site Scripting (XSS)
    addReview(review) {
      // Thêm review trực tiếp vào danh sách mà không kiểm tra XSS
      this.reviews.push({ text: review });
    },

    // Insecure Deserialization - Lỗ hổng khi deserialization dữ liệu không an toàn
    submitOrder() {
      const customerData = '{"username":"admin","password":"password"}'; // Dữ liệu có thể bị tấn công nếu không kiểm tra
      const order = JSON.parse(customerData);  // Giả lập lỗ hổng deserialization không an toàn
      fetch(`/order`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(order)
      });
    },

    // Cross-Site Request Forgery (CSRF) - Không bảo vệ CSRF
    submitOrderWithoutCSRF() {
      const order = {
        customerId: '12345',
        items: this.cartItems.map(item => ({
          productId: item.product.id,
          quantity: item.quantity
        }))
      };

      // CSRF không có token bảo vệ
      fetch(`/order`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(order)
      })
      .then(response => console.log('Order submitted'))
      .catch(error => console.error('Order submission failed'));
    },

    // Unvalidated Redirects and Forwards - Chuyển hướng không kiểm tra
    goToPage(page) {
      window.location.href = page;  // Không kiểm tra các URL trước khi chuyển hướng
    },

    // Broken Access Control - Truy cập không hợp lệ
    viewSensitiveData() {
      if (this.isAuthenticated) {
        alert('Sensitive Data Here!');
      } else {
        alert('Access Denied!');
      }
    },

    // Misconfigured CORS - CORS không cấu hình chính xác
    fetchDataWithCORS() {
      fetch('http://another-domain.com/api/data')  // Chưa cấu hình CORS trên server
        .then(response => response.json())
        .then(data => console.log(data))
        .catch(error => console.error('Failed to fetch data'));
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 120px;
}

footer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background-color: #333;
  color: #fff;
  padding: 1rem;
  margin: 0;
}

nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

ul {
  display: flex;
  list-style: none;
  margin: 0;
  padding: 0;
}

li {
  margin: 0 1rem;
}

a {
  color: #fff;
  text-decoration: none;
}

button {
  padding: 10px;
  background-color: #005f8b;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  height: 42px;
}

.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
}

.product-card {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  margin: 1rem;
  padding: 1rem;
  border: 1px solid #ccc;
  border-radius: 0.5rem;
}

.product-card img {
  max-width: 100%;
  margin-bottom: 1rem;
}

.product-card a {
  text-decoration: none;
  color: #333;
}

.product-card h2 {
  font-weight: bold;
  margin-bottom: 0.5rem;
}

.product-card p {
  margin-bottom: 1rem;
}

.product-controls {
  display: flex;
  align-items: center;
  margin-top: 0.5rem;
}

.product-controls p {
  margin-right: 20px;
}

.product-controls button:hover {
  background-color: #005f8b;
}

.product-price {
  font-weight: bold;
  font-size: 1.2rem;
}

.quantity-input {
  width: 50px;
  height: 30px;
  border: 1px solid #ccc;
  border-radius: 5px;
  padding: 5px;
  margin-right: 10px;
}

.shopping-cart {
  display: flex;
  flex-direction: column;
  align-items: center;
}

.shopping-cart h2 {
  font-size: 24px;
  margin-bottom: 20px;
}

.shopping-cart-table {
  width: 100%;
  border-collapse: collapse;
}

.shopping-cart-table th,
.shopping-cart-table td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.shopping-cart-table th {
  font-weight: bold;
}

.shopping-cart-table td img {
  display: block;
  margin: 0 auto;
}

.checkout-button {
  margin-top: 20px;
  padding: 10px 20px;
  background-color: #007acc;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.checkout-button:hover {
  background-color: #005f8b;
}
</style>
