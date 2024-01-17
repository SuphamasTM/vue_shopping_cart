<template>
  <div>
    <template>
      <div>
        <v-app-bar color="deep-purple accent-4" dense dark>
          <v-app-bar-nav-icon></v-app-bar-nav-icon>

          <v-toolbar-title>Shopping App</v-toolbar-title>

          <v-btn text large>Product</v-btn>

          <v-badge right color="primary">
            <v-btn icon>
              <v-icon>mdi-cart</v-icon>
            </v-btn>
          </v-badge>

          <v-spacer></v-spacer>

          <v-menu left bottom>
            <template v-slot:activator="{ on, attrs }">
              <v-btn icon v-bind="attrs" v-on="on">
                <v-icon>mdi-dots-vertical</v-icon>
              </v-btn>
            </template>

            <v-list>
              <v-list-item v-for="n in 5" :key="n" @click="() => {}">
                <v-list-item-title>Option {{ n }}</v-list-item-title>
              </v-list-item>
            </v-list>
          </v-menu>
        </v-app-bar>
      </div>
    </template>
  </div>
</template>

<script>
export default {};
</script>

<style></style>
