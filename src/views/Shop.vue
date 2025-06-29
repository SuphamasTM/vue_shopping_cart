<template>
  <div>
    <v-app-bar app color="white">
      <v-img
        src="https://cdn.shopify.com/s/files/1/0049/9823/4230/files/fjallraven-logo_160x.png?v=1664950850"
        max-width="100"
      >
      </v-img>
      <v-spacer></v-spacer>
      <v-btn text @click="changePage('product')"
        ><v-icon left>mdi-shopping-outline</v-icon>
        <h2>SHOP</h2>
      </v-btn>
      <v-btn text @click="changePage('cart')"
        ><v-icon left>mdi-cart-outline</v-icon>
        <h2>CART</h2>
      </v-btn>
    </v-app-bar>

    <v-main style="margin: 40px; margin-bottom: 140px">
      <div v-if="page === 'product'">
        <v-card>
          <v-card-title primary-title>
            <h1>All Products</h1>
          </v-card-title>
          <v-card-subtitle>
            <v-text-field
              v-model="searchQuery"
              label="Search products"
              outlined
              clearable
              @input="debouncedSearch"
            ></v-text-field>
          </v-card-subtitle>
          <v-row>
            <v-col
              cols="12"
              sm="6"
              md="4"
              lg="3"
              xl="2"
              v-for="(product, index) in filteredProducts"
              :key="index"
            >
              <v-card max-width="500" max-height="900">
                <v-img
                  aspect-ratio="1"
                  contain
                  :src="product.detail.img"
                ></v-img>
                <v-card-title primary-title>
                  <strong>{{ product.product_name }}</strong>
                </v-card-title>
                <v-card-subtitle>
                  <div class="detail">
                    {{ product.detail.color }}<br />
                    <span :style="{ color: 'black' }">${{ product.price }}</span
                    ><br />
                    <span
                      :style="{ color: product.quantity < 5 ? 'red' : 'black' }"
                    >
                      {{ product.quantity }} piece available
                    </span>
                  </div>
                </v-card-subtitle>
                <v-card-actions>
                  <v-btn icon @click="decrement(product)">
                    <v-icon color="black">mdi-minus</v-icon>
                  </v-btn>
                  <h1>{{ product.amount }}</h1>
                  <v-btn icon @click="increment(product)">
                    <v-icon color="black">mdi-plus</v-icon>
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-col>
          </v-row>
        </v-card>
      </div>
      <div v-if="page === 'cart'">
        <v-card>
          <v-card-title primary-title>
            <h1>CART</h1>
          </v-card-title>
          <div v-if="cart.length === 0" class="empty-cart">
            <p style="text-align: center; font-size: 22px; margin: 20px">
              Your cart is empty. Let's go shopping!
            </p>
            <v-btn
              color="primary"
              @click="changePage('product')"
              style="margin: 0 auto; display: block"
            >
              Back to SHOP
            </v-btn>
          </div>
          <div v-else>
            <v-data-table
              :headers="
                cartTableHeader.filter((header) => header.value !== 'action')
              "
              :items="cart"
              class="elevation-1"
              dense
              hide-default-footer
            >
              <template v-slot:top>
                <v-toolbar flat>
                  <v-toolbar-title>Your Shopping Cart</v-toolbar-title>
                  <v-spacer></v-spacer>
                  <v-btn color="primary" @click="changePage('product')">
                    <v-icon left>mdi-shopping-outline</v-icon>
                    Continue Shopping
                  </v-btn>
                </v-toolbar>
              </template>
              <template v-slot:item.no="{ item }">
                <p>{{ cart.indexOf(item) + 1 }}</p>
              </template>
              <template v-slot:item.product_img="{ item }">
                <v-img
                  :src="item.product_img"
                  max-width="100"
                  max-height="100"
                  aspect-ratio="1"
                  class="hover-zoom"
                ></v-img>
              </template>
              <template v-slot:item.product_name="{ item }">
                <p>{{ item.product_name }}</p>
              </template>
              <template v-slot:item.quantity="{ item }">
                <div class="d-flex justify-center align-center">
                  <v-btn icon @click="decrementCartItem(item)">
                    <v-icon color="black">mdi-minus</v-icon>
                  </v-btn>
                  <span class="mx-2">{{ item.quantity }}</span>
                  <v-btn icon @click="incrementCartItem(item)">
                    <v-icon color="black">mdi-plus</v-icon>
                  </v-btn>
                </div>
                <p
                  v-if="getProductStock(item.product_id) < 5"
                  style="color: red; font-size: 16px"
                >
                  Only {{ getProductStock(item.product_id) }} items left
                </p>
              </template>
              <template v-slot:item.product_price="{ item }">
                <p>
                  ${{
                    item.product_price.toLocaleString("en-US", {
                      minimumFractionDigits: 2,
                    })
                  }}
                </p>
              </template>
              <template v-slot:item.totalCost="{ item }">
                <p>${{ totalCost(item.product_price, item.quantity) }}</p>
              </template>
            </v-data-table>
            <v-card-actions>
              <p>
                Total items:
                {{ cart.reduce((sum, item) => sum + item.quantity, 0) }}
              </p>
            </v-card-actions>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="success"
                @click="confirmOrder()"
                :disabled="cart.length === 0"
              >
                <h2>Checkout - Total: ${{ subTotal() }}</h2>
              </v-btn>
            </v-card-actions>
          </div>
        </v-card>
      </div>
    </v-main>
    <!-- ปุ่มย้อนกลับไปด้านบน -->
    <!-- <v-btn
      v-if="page === 'product' && showScrollToTop"
      fab
      color="primary"
      class="scroll-to-top"
      @click="scrollToTop"
    >
      <v-icon>mdi-chevron-up</v-icon>
    </v-btn> -->
    <v-footer
      dark
      padless
      class="footer"
      style="z-index: 1; opacity: 0.9; height: 100px"
    >
      <v-container>
        <v-row justify="center">
          <v-col cols="12" md="6" class="text-center">
            <h3 class="white--text">
              © {{ new Date().getFullYear() }} — FJÄLLRÄVEN
            </h3>
            <p class="white--text">
              Explore our collection of sustainable outdoor gear and
              accessories.
            </p>
          </v-col>
          <v-col cols="12" md="6" class="text-center">
            <h4 class="white--text">Follow Us</h4>
            <v-icon left>mdi-facebook</v-icon>
            <v-icon left>mdi-instagram</v-icon>
            <v-icon left>mdi-twitter</v-icon>
          </v-col>
        </v-row>
      </v-container>
    </v-footer>
  </div>
</template>
<style>
@import url("https://fonts.googleapis.com/css2?family=Nanum+Pen+Script&display=swap");

/* ปรับปรุงขนาด Fonts */
* {
  font-family: "Nanum Pen Script", cursive;
}
.detail,
p {
  font-size: 22px; /* ปรับขนาดให้เหมาะสม */
}
h1 {
  font-size: 28px; /* ขนาดหัวข้อใหญ่ */
}
h2 {
  font-size: 24px; /* ขนาดหัวข้อรอง */
}
strong {
  font-size: 26px; /* ขนาดตัวหนา */
}

/* ปรับปรุง Footer */
.footer {
  position: fixed; /* เปลี่ยนกลับเป็น fixed */
  bottom: 0;
  width: 100%;
  z-index: 1000;
  background-color: #333; /* เพิ่มสีพื้นหลัง */
  padding: 10px 0; /* เพิ่ม Padding */
}

/* ปรับปรุง v-main */
.v-main {
  margin-bottom: 160px; /* เว้นระยะห่างจาก Footer */
  min-height: calc(100vh - 160px); /* ทำให้เนื้อหาเต็มหน้าจอ */
}

/* เพิ่ม hover-zoom */
.hover-zoom {
  transition: transform 0.3s ease-in-out;
}
.hover-zoom:hover {
  transform: scale(1.5);
}

/* ปรับขนาดฟอนต์ในตาราง */
.v-data-table {
  font-size: 20px; /* ขนาดฟอนต์สำหรับตาราง */
}

.v-data-table p,
.v-data-table span {
  font-size: 22px; /* ขนาดฟอนต์สำหรับข้อความในตาราง */
}

.v-data-table .v-btn {
  font-size: 18px; /* ขนาดฟอนต์สำหรับปุ่มในตาราง */
}

.v-toolbar-title {
  font-size: 26px; /* ขนาดฟอนต์สำหรับหัวข้อใน Toolbar */
}

.v-card-actions p {
  font-size: 22px; /* ขนาดฟอนต์สำหรับข้อความในส่วนท้ายของตาราง */
}

.v-btn h2 {
  font-size: 24px; /* ขนาดฟอนต์สำหรับปุ่ม Checkout */
}

/* ปุ่มย้อนกลับไปด้านบน */
.scroll-to-top {
  position: fixed; /* ทำให้ปุ่มลอยอยู่ในตำแหน่งคงที่ */
  bottom: 20px; /* ระยะห่างจากด้านล่าง */
  right: 20px; /* ระยะห่างจากด้านขวา */
  z-index: 1000; /* ทำให้ปุ่มอยู่ด้านบนสุด */
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2); /* เพิ่มเงา */
}
</style>
<script>
import Swal from "sweetalert2";

export default {
  created() {
    this.getData();
    window.addEventListener("scroll", this.handleScroll); // ตรวจจับการเลื่อน
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.handleScroll); // ลบ event listener เมื่อ component ถูกทำลาย
  },
  data() {
    return {
      dialog: false,
      page: "product",
      products: [],
      cart: [],
      showScrollToTop: false, // สถานะของปุ่ม
      searchQuery: "", // คำค้นหา
      filteredProducts: [], // สินค้าที่ผ่านการกรอง
      debounceTimeout: null, // ตัวหน่วงเวลา
      cartTableHeader: [
        {
          text: "No.",
          value: "no",
          sortable: false,
          align: "center",
          width: "10%",
        },
        {
          text: "Image",
          value: "product_img",
          sortable: false,
          align: "center",
          width: "5%",
        },
        {
          text: "Name",
          value: "product_name",
          sortable: false,
          align: "center",
          width: "15%",
        },
        {
          text: "Price",
          value: "product_price",
          sortable: false,
          align: "center",
          width: "10%",
        },
        {
          text: "Quantity",
          value: "quantity",
          sortable: false,
          align: "center",
          width: "10%",
        },
        {
          text: "Total cost",
          value: "totalCost",
          sortable: false,
          align: "center",
          width: "10%",
        },
        {
          text: "Action",
          value: "action",
          sortable: false,
          align: "center",
          width: "10%",
        },
      ],
    };
  },
  watch: {
    products: {
      immediate: true,
      handler(newProducts) {
        this.filteredProducts = newProducts; // ตั้งค่าเริ่มต้น
      },
    },
  },
  methods: {
    confirmOrder() {
      const totalItems = this.cart.reduce(
        (sum, item) => sum + item.quantity,
        0
      );
      const totalPrice = this.subTotal();

      Swal.fire({
        title: "Confirm the order?",
        html: `
          <p><strong>Total Items:</strong> ${totalItems}</p>
          <p><strong>Total Price:</strong> $${totalPrice}</p>
          <p>Are you sure you want to proceed with the checkout?</p>
        `,
        icon: "question",
        showCancelButton: true,
        confirmButtonColor: "#00aa00",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, confirm!",
        cancelButtonText: "Cancel",
      }).then((result) => {
        if (result.isConfirmed) {
          Swal.fire({
            title: "Order Successful!",
            html: `
              <p>Thank you for your order.</p>
              <p><strong>Total Items:</strong> ${totalItems}</p>
              <p><strong>Total Price:</strong> $${totalPrice}</p>
              <p>Your order will be processed shortly.</p>
            `,
            icon: "success",
          });
          console.log(this.products);
          this.cart.forEach((val) => {
            let index = this.products.findIndex(
              (item) => item._id == val.product_id
            );
            if (this.products[index].quantity > 0) {
              this.products[index].amount = 0;
            }
          });
          this.cart = [];
        }
      });
    },
    totalCost(price, quantity) {
      const total = price * quantity;
      return total.toLocaleString("en-US", { minimumFractionDigits: 2 });
    },
    subTotal() {
      let totalPrice = 0;
      this.cart.forEach((product) => {
        totalPrice += product.quantity * product.product_price;
      });
      return totalPrice.toLocaleString("en-US", { minimumFractionDigits: 2 });
    },
    removeItemFromCart(product) {
      this.cart.splice(this.cart.indexOf(product), 1);

      this.products[
        this.products.findIndex((v) => v._id === product.product_id)
      ].amount = 0;
    },
    increment(product) {
      let search = this.cart.find((cart) => cart.product_id === product._id);
      if (search === undefined) {
        // ไม่เจอสินค้า
        this.cart.push({
          product_id: product._id,
          quantity: 1,
          product_name: product.product_name,
          product_price: product.price,
          product_img: product.detail.img,
        });
        this.products[this.products.indexOf(product)].amount = 1;
      } else {
        // เจอสินค้า
        if (search.quantity >= product.quantity) return;
        search.quantity++;
        this.products[this.products.indexOf(product)].amount++;
      }
    },
    decrement(product) {
      let search = this.cart.find((cart) => cart.product_id === product._id);

      if (search) {
        if (search.quantity == 0) return;
        else {
          search.quantity--;
          this.products[this.products.indexOf(product)].amount--;

          // Remove item from cart if quantity reaches 0
          if (search.quantity === 0) {
            this.cart.splice(this.cart.indexOf(search), 1);
          }
        }
      }
    },
    incrementCartItem(item) {
      let product = this.products.find(
        (product) => product._id === item.product_id
      );
      if (product && item.quantity < product.quantity) {
        item.quantity++;
      }
    },
    decrementCartItem(item) {
      if (item.quantity > 1) {
        item.quantity--;
      } else {
        Swal.fire({
          title: "Are you sure?",
          text: "Do you want to remove this item from the cart?",
          icon: "warning",
          showCancelButton: true,
          confirmButtonColor: "#d33",
          cancelButtonColor: "#3085d6",
          confirmButtonText: "Yes, remove it!",
          cancelButtonText: "Cancel",
        }).then((result) => {
          if (result.isConfirmed) {
            this.removeItemFromCart(item);
          }
        });
      }
    },
    getProductStock(productId) {
      let product = this.products.find((product) => product._id === productId);
      return product ? product.quantity : 0;
    },
    changePage(page) {
      this.page = page;
    },
    async getData() {
      await this.axios
        .get("https://e-commerce-api-puce.vercel.app/products")
        .then((response) => {
          if (response) {
            this.products = response.data.data;
            // console.log(response.data.data);
          }
        });
    },
    async postData() {
      // สร้างออเดอร์
      try {
        let data = {
          user_id: "1234",
          order_item: [...this.cart],
        };
        await this.axios.post("http://localhost:5000/orders", data);
        this.dialog = false;
        alert("Orders Success");
      } catch (error) {
        console.log(error);
      }
    },
    scrollToTop() {
      window.scrollTo({
        top: 0,
        behavior: "smooth",
      });
    },
    handleScroll() {
      this.showScrollToTop = window.scrollY > 100; // แสดงปุ่มเมื่อเลื่อนลงไปเกิน 100px
    },
    debouncedSearch() {
      clearTimeout(this.debounceTimeout); // ล้าง timeout ก่อนหน้า
      this.debounceTimeout = setTimeout(() => {
        this.searchProducts();
      }, 300); // หน่วงเวลา 300ms
    },
    searchProducts() {
      const query = this.searchQuery.toLowerCase();
      this.filteredProducts = this.products.filter((product) =>
        product.product_name.toLowerCase().includes(query)
      );
    },
  },
};
</script>
