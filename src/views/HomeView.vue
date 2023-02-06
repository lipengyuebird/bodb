<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-col>
        <div class="d-flex align-center">
          <v-img alt="Vuetify Logo" class="shrink mr-2" contain
            :src="require('../assets/teddy-bear-dark.svg')" transition="scale-transition"
            width="40" />
          <h3>Bank of Duck Bear</h3>
        </div>
      </v-col>

      <v-spacer></v-spacer>
      <h4 v-if="isShow">Base Currency :</h4>

      <v-menu offset-y>
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

      <v-container>
        <v-row class="text-center">
          <v-col cols="12">
            <v-img :src="require('../assets/teddy-bear.svg')" class="my-3" contain height="200" />
          </v-col>

          <v-col class="mb-4">
            <h1 class="display-2 font-weight-bold mb-3">
              Exchange Rate Comparison Tool
            </h1>
          </v-col>
        </v-row>
        <v-card>
          <template v-slot:progress>
            <v-progress-linear absolute height="4" indeterminate></v-progress-linear>
          </template>
          <v-form>
            <v-container>
              <v-row class="pt-2">
                <v-col cols="12" md="9">
                  <v-select :items="ccys" v-model="ccy" label="Choose a currency"
                    :rules="[v => !!v || 'Please select a currency']"></v-select>
                </v-col>
                <v-col cols="12" md="3" class="pt-6">
                  <v-btn block @click="query(ccy)" color="primary">
                    <v-icon left> mdi-hand-coin </v-icon>
                    Query Now
                  </v-btn>
                </v-col>
              </v-row>
            </v-container>
          </v-form>
        </v-card>
      </v-container>

      <v-container>
        <v-card>
          <v-card-title>
            Historical Rate against {{ baseCcy }} (last 7 days)
          </v-card-title>
          <v-card-text center class="d-flex flex-wrap">
            <TrendPanel v-for="(val, key) in history.rate" :key="key" :ccy="key" :date="history.date[key]" :rate="val" class="pa-ma-16" />
          </v-card-text>

        </v-card>

      </v-container>
    </v-main>
    <v-snackbar v-model="snackbar" timeout=2000>
      Please select a currency.

      <template v-slot:action="{ attrs }">
        <v-btn color="blue" text v-bind="attrs" @click="snackbar = false">
          Close
        </v-btn>
      </template>
    </v-snackbar>
  </v-app>
</template>

<script>
import Axios from "axios";

import TrendPanel from "../components/TrendPanel";

export default {
  name: "App",

  components: {
    TrendPanel,
  },

  created() {
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
    this.getHistory();

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
        if (val < 800) {
          this.isShow = false;
        } else {
          this.isShow = true;
        }
      },
      immediate: true,
    },
  },
  methods: {
    query(ccy) {
      if (!ccy) {
        this.snackbar = true;
        return;
      }
      this.$router.push({
        path: '/result',
        query: {
          base_ccy: this.baseCcy,
          ccy: ccy
        }
      })
    },
    getHistory() {
      Axios.get("http://192.168.3.11:8081/history").then((res) => {
        this.history = res.data;
        eval('delete this.history.rate.' + this.baseCcy);
      });
      
    }
  },

  data: () => ({
    baseCcy: 'HKD',
    baseCcys: [],
    ccys: [],
    ccy: null,
    screenWidth: 0,
    isShow: true,
    snackbar: false,
    history: {}
  }),
};
</script>
