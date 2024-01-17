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
          <v-card-title primary-title> <h1>All Products</h1> </v-card-title>
          <v-row>
            <v-col
              cols="12"
              sm="6"
              md="4"
              lg="3"
              xl="2"
              v-for="(product, index) in products"
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
                    <span :style="{ color: 'black' }"
                      >${{ product.price }}</span
                    >
                    <br />
                    {{ product.quantity }} piece avaliable <br />
                  </div>
                  <span></span>
                </v-card-subtitle>
                <v-card-actions>
                  <v-btn icon @click="decrement(product)">
                    <v-icon color="black">mdi-minus</v-icon></v-btn
                  >
                  <h1>{{ product.amount }}</h1>
                  <v-btn icon @click="increment(product)">
                    <v-icon color="black">mdi-plus</v-icon></v-btn
                  >
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
          <v-data-table :headers="cartTableHeader" :items="cart">
            <template v-slot:item.no="{ item }">
              <p>{{ cart.indexOf(item) + 1 }}.</p>
            </template>
            <template v-slot:item.product_img="{ item }">
              <div>
                <v-img aspect-ratio="1" :src="item.product_img"></v-img>
              </div>
            </template>
            <template v-slot:item.product_name="{ item }">
              <p>{{ item.product_name }}</p>
            </template>
            <template v-slot:item.quantity="{ item }">
              <p>{{ item.quantity }}</p>
            </template>
            <template v-slot:item.product_price="{ item }">
              <p>${{ item.product_price }}</p>
            </template>
            <template v-slot:item.totalCost="{ item }">
              <p>${{ totalCost(item.product_price, item.quantity) }}</p>
            </template>
            <template v-slot:item.action="{ item }">
              <v-btn icon color="error" small @click="removeItemFromCart(item)">
                <v-icon small>mdi-delete</v-icon>
              </v-btn>
            </template>
          </v-data-table>
          <v-btn
            v-if="cart.length != 0"
            color="success"
            @click="confirmOrder()"
          >
            <h2>Checkout</h2>
          </v-btn>
          <v-btn v-else disabled>
            <h2>Checkout</h2>
          </v-btn>
        </v-card>
      </div>
    </v-main>
    <v-footer dark padless class="footer">
      <v-card class="flex" flat tile>
        <v-card-title class="dark">
          <strong class="subheading"
            >Get connected with us on social networks!</strong
          >

          <v-spacer></v-spacer>

          <v-btn v-for="icon in icons" :key="icon" class="mx-4" dark icon>
            <v-icon size="24px">
              {{ icon }}
            </v-icon>
          </v-btn>
        </v-card-title>

        <v-card-text class="py-2 white--text text-center">
          <h3>
            © {{ new Date().getFullYear() }} — <strong> FJÄLLRÄVEN hihi</strong>
          </h3>
        </v-card-text>
      </v-card>
    </v-footer>
  </div>
</template>
<style>
.footer {
  position: fixed;
  bottom: 0;
  width: 100%;
  z-index: 1000; /* Adjust the z-index as needed */
}
</style>
<script>
import Swal from "sweetalert2";

export default {
  created() {
    this.getData();
  },
  data() {
    return {
      dialog: false,
      page: "product",
      products: [],
      cart: [],
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
  methods: {
    confirmOrder() {
      Swal.fire({
        title: "Confirm the order?",
        text: `Total: $${this.subTotal()}`,
        icon: "question",
        showCancelButton: true,
        confirmButtonColor: "#00aa00",
        cancelButtonColor: "#d33",
        confirmButtonText: "Yes, confirm!",
      }).then((result) => {
        if (result.isConfirmed) {
          Swal.fire({
            title: "The order was successful.",
            text: "Thank you for ordering.",
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
      return (price * quantity)
        .toString()
        .replace(/\B(?=(\d{3})+(?!\d))/g, ",");
    },
    subTotal() {
      let totalPrice = 0;
      this.cart.forEach((product) => {
        totalPrice += product.quantity * product.product_price;
      });
      totalPrice = totalPrice.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",");
      return totalPrice;
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
        }
      }
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
  },
};
</script>
<style>
@import url("https://fonts.googleapis.com/css2?family=Nanum+Pen+Script&display=swap");
* {
  font-family: "Nanum Pen Script", cursive;
}
.detail,
p {
  font-size: 20px;
}
</style>
