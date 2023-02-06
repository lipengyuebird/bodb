<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-col v-if="isShow" @click="goHome()">

        <div class="d-flex align-center">
          <router-link to="/"> </router-link>
          <v-img alt="Vuetify Logo" class="shrink mr-2" contain
          :src="require('../assets/teddy-bear-dark.svg')" transition="scale-transition"
            width="40" />
          <h3>Bank of Duck Bear</h3>
        </div>
      </v-col>

      <v-col>
        <v-select class="pt-6" :items="ccys" label="Currency" v-model="ccy"></v-select>
      </v-col>
      <v-col>
        <v-btn @click="query(ccy)" color="white" outlined>
          <v-icon left> mdi-hand-coin </v-icon>
          Query Now
        </v-btn>
      </v-col>

      <!-- <v-spacer></v-spacer> -->
      <h4 v-if="isShow">Base Currency :</h4>

      <v-menu offset-y v-if="isShow">
        <template v-slot:activator="{ attrs, on }">
          <v-btn plain v-bind="attrs" v-on="on">
            HKD
            <v-icon left> mdi-menu-down </v-icon>
          </v-btn>
        </template>

        <v-list>
          <v-list-item v-for="item in baseCcys" :key="item" link>
            <v-list-item-title v-text="item"></v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-app-bar>

    <v-main>

      <v-container class="pa-10">
        <v-card class="d-flex mb-6" flat height="100" v-if="isShow" tile>

          <h1 class="pa-2 align-self-center">
            {{ this.$route.query.ccy }}
          </h1>

          <v-card class="ml-auto" elevation="0">
            <v-card-text>
              TT Buy:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Telegraphic Transfer Bank Buy <br>
              TT Sell:&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Telegraphic Transfer Bank Sell <br>
              Note Buy:&nbsp;&nbsp;&nbsp;Banknotes Bank Buy <br>
              Note Sell:&nbsp;&nbsp;&nbsp;Banknotes Bank Sell <br>
            </v-card-text>
          </v-card>
        </v-card>
        <v-row>
          <v-col>
            <template class="pt-mu-10">
              <v-data-table loading="dataLoading" :headers="headers" :items="results" :sort-by="['telegraphic_transfer_bank_buy']"
                class=" elevation-1">
                <template v-slot:[`item.telegraphic_transfer_bank_buy`]="{ item }">
                  {{ formatFloat(item.telegraphic_transfer_bank_buy) || 'N/A' }}
                  <!-- <v-chip :color="getColor(item.telegraphic_transfer_bank_buy)" dark>
                    {{ item.telegraphic_transfer_bank_buy }}
                  </v-chip> -->
                </template>
                <template v-slot:[`item.telegraphic_transfer_bank_sell`]="{ item }">
                  {{ formatFloat(item.telegraphic_transfer_bank_sell) || 'N/A' }}
                </template>
                <template v-slot:[`item.banknotes_bank_buy`]="{ item }">
                  {{ formatFloat(item.banknotes_bank_buy) || 'N/A' }}
                </template>
                <template v-slot:[`item.banknotes_bank_sell`]="{ item }">
                  {{ formatFloat(item.banknotes_bank_sell) || 'N/A' }}
                </template>
                <template v-slot:[`item.last_updated`]="{ item }">
                  {{ pickDate(item.last_updated )|| 'N/A' }}
                </template>
              </v-data-table>
            </template>
          </v-col>
        </v-row>
      </v-container>


    </v-main>
    <v-fab-transition v-if="!isShow">
      <v-btn dark fixed bottom right fab @click="goHome()" color="primary">
        <v-icon>mdi-home</v-icon>
      </v-btn>
    </v-fab-transition>
  </v-app>
</template>

<script>
import Axios from 'axios';

export default {

  components: {},

  created() {
    this.ccy = this.$route.query.ccy || this.ccy;
    this.baseCcy = this.$route.query.base_ccy || this.baseCcy;
    Axios.get("http://192.168.3.11:8081/base_ccys").then((res) => {
      this.baseCcys = res.data;
    });
    Axios.get("http://192.168.3.11:8081/ccys").then((res) => {
      this.ccys = res.data;
      var index = this.ccys.indexOf(this.baseCcy);
      if (index > -1) {
        this.ccys.splice(index, 1);
      }
    });
    Axios.get("http://192.168.3.11:8081/rate?base_currency=" + this.baseCcy + "&currency=" + this.ccy).then((res) => {
      this.results = [];
      for (var key in res.data) {
        this.results.push({ 'name': this.banks[key].name, ...res.data[key] })
      }
      this.dataLoading = false;
    });
  },
  mounted() {
    this.screenWidth = document.body.clientWidth;
    window.onresize = () => {
      this.screenWidth =
        window.innerWidth ||
        document.documentElement.clientWidth ||
        document.body.clientWidth;
    };
  },
  watch: {
    screenWidth: {
      handler: function (val) {
        if (val < 950) {
          this.isShow = false;
        } else {
          this.isShow = true;
        }
      },
      immediate: true,
    },
  },

  data: () => ({
    dataLoading: true,
    isShow: true,
    screenWidth: 0,
    ccy: 'USD',
    baseCcy: 'HKD',
    ccys: [],
    baseCcys: [],
    headers: [
      {
        text: 'Bank',
        align: 'start',
        sortable: false,
        value: 'name',
      },
      { text: 'TT Buy', value: 'telegraphic_transfer_bank_buy' },
      { text: 'TT Sell', value: 'telegraphic_transfer_bank_sell' },
      { text: 'Note Buy', value: 'banknotes_bank_buy' },
      { text: 'Note Sell', value: 'banknotes_bank_sell' },
      { text: 'Last Updated', value: 'last_updated', sortable: false}
    ],
    results: [],
    banks: {
      'bochk': {
        'name': 'Bank of China (Hong Kong)',
        'logo': ''
      },
      'bodb': {
        'name': 'Bank of Duck Bear',
        'logo': ''
      },
      'hsbc': {
        'name': 'HSBC',
        'logo': ''
      },
      'hang_seng': {
        'name': 'HANG SENG BANK',
        'logo': ''
      },

    }
  }),

  methods: {
    getColor(value) {
      if (value > 400) return 'red'
      else if (value > 200) return 'orange'
      else return 'green'
    },
    query(ccy) {
      this.$router.push({
        path: '/result',
        query: {
          base_ccy: this.baseCcy,
          ccy: ccy
        }
      })
      this.$router.go(0)
    },
    formatFloat(number) {
      if (number) {
        return parseFloat(number).toFixed(4);
      }
    },
    pickDate(date) {
      if (date) {
        return date.substring(0, date.lastIndexOf('.')).replace('T', ' ');
      }
    },
    goHome() {
      this.$router.push({
        path: '/',
      })
    }
  },
};
</script>
