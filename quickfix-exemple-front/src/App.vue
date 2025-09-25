<template>
  <v-app>
    <v-main>
      <v-form>
        <v-container>
          <v-row class="d-flex justify-center">
                <v-col class="d-flex justify-center" cols="4">
                  <v-combobox
                    label="Símbolo"
                    v-model="order.symbol"
                    :items="['PETR4', 'VALE3', 'VIIA4']"
                  ></v-combobox>
                </v-col>
              </v-row>
          <v-row class="d-flex justify-center">
            <v-col class="d-flex justify-center" cols="4">
              <v-select
                v-model="order.side"
                :items="[{ label: 'Compra', value: '1' }, { label: 'Venda', value: '2'}]"
                item-title="label"
                item-value="value"
                label="Lado"
                return-object
                single-line
              ></v-select>
            </v-col>
          </v-row>
          <v-row class="d-flex justify-center">
            <v-col class="d-flex justify-center" cols="4">
              <v-number-input
                :reverse="false"
                label="Quantidade"
                :hideInput="false"
                :inset="false"
                :max="100000"
                :min="1"
                :precision="0"
                v-model="order.orderQty"
              ></v-number-input>
            </v-col>
          </v-row>
          <v-row class="d-flex justify-center">
            <v-col class="d-flex justify-center" cols="4">
              <VCurrencyField
                required
                v-model="order.price"
              />
            </v-col>
          </v-row>
          <v-row class="d-flex justify-center">
            <v-col class="d-flex justify-center" cols="4">
              <v-btn :loading="loading" @click="onSubmit">Enviar</v-btn>
            </v-col>
          </v-row>

        </v-container>
      </v-form>
      <v-snackbar
        v-model="snackbar"
        timeout="1000"
        location="top"
        :color="colorStatus"
      >
        {{ orderStatus }}

        <template v-slot:actions>
          <v-btn
            variant="text"
            @click="snackbar = false"
          >
            Close
          </v-btn>
        </template>
      </v-snackbar>
    </v-main>

  </v-app>
</template>

<script>
import VCurrencyField from './components/currancyInput.vue';
import {ref} from "vue";

export default {
  components:{
    VCurrencyField
  },
  data: () => ({
    order:{
      symbol: 'PETR4',
      side:ref({ label: 'Compra', value: '1' }),
      orderQty:ref(1),
      price:ref(0.01)
    },
    loading: false,
    snackbar: false,
    orderStatus: "",
    colorStatus: "success"

  }),
  methods:{
    onSubmit(){
      this.loading = true;
      const body = {
        symbol: this.order.symbol,
        side: this.order.side.value,
        orderQty: this.order.orderQty,
        price: this.order.price
      }
      const requestOptions = {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json, text/plain, */*',
          'Accept': "application/json, text/plain, */*",

        },
        body: JSON.stringify(body)
      };
      fetch('http://localhost:5242/order', requestOptions)
        .then(response => response.json())
        .then(async id => {
          this.orderStatus = "Sent";
          this.colorStatus = "success"
          do {
            const response = await fetch('http://localhost:5242/order/'+id);
            this.orderStatus = await response.text();
          }while (this.orderStatus === "Sent")
          if(this.orderStatus === "Rejected") this.colorStatus = "red-darken-2"
          this.snackbar = true;
        })
        .finally(()=>{
          this.loading = false;
        })
    }
  }
}
</script>
