<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <div
      v-if="isProcessing"
      class="
        fixed
        w-100
        h-100
        opacity-80
        bg-purple-800
        inset-0
        z-50
        flex
        items-center
        justify-center
      "
    >
      <svg
        class="animate-spin -ml-1 mr-3 h-12 w-12 text-white"
        xmlns="http://www.w3.org/2000/svg"
        fill="none"
        viewBox="0 0 24 24"
      >
        <circle
          class="opacity-25"
          cx="12"
          cy="12"
          r="10"
          stroke="currentColor"
          stroke-width="4"
        ></circle>
        <path
          class="opacity-75"
          fill="currentColor"
          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"
        ></path>
      </svg>
    </div>
    <div class="container">
      <section>
        <div class="flex">
          <div class="max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700"
              >Тикер</label
            >
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="ticker"
                @keydown.enter="() => handleAddTicker()"
                type="text"
                name="wallet"
                id="wallet"
                class="
                  block
                  w-full
                  pr-10
                  border-gray-300
                  text-gray-900
                  focus:outline-none focus:ring-gray-500 focus:border-gray-500
                  sm:text-sm
                  rounded-md
                "
                placeholder="Например DOGE"
              />
            </div>

            <div
              v-if="coinHelpers.length"
              class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
            >
              <span
                v-for="coin in coinHelpers"
                :key="coin"
                @click="handleAddTickerFromHelpers(coin)"
                class="
                  inline-flex
                  items-center
                  px-2
                  m-1
                  rounded-md
                  text-xs
                  font-medium
                  bg-gray-300
                  text-gray-800
                  cursor-pointer
                "
              >
                {{ coin }}
              </span>
            </div>
            <div v-if="tickerAlert" class="text-sm text-red-600">
              Такой тикер уже добавлен
            </div>
          </div>
        </div>
        <button
          @click="() => handleAddTicker()"
          type="button"
          class="
            my-4
            inline-flex
            items-center
            py-2
            px-4
            border border-transparent
            shadow-sm
            text-sm
            leading-4
            font-medium
            rounded-full
            text-white
            bg-gray-600
            hover:bg-gray-700
            transition-colors
            duration-300
            focus:outline-none
            focus:ring-2
            focus:ring-offset-2
            focus:ring-gray-500
          "
        >
          <!-- Heroicon name: solid/mail -->
          <svg
            class="-ml-0.5 mr-2 h-6 w-6"
            xmlns="http://www.w3.org/2000/svg"
            width="30"
            height="30"
            viewBox="0 0 24 24"
            fill="#ffffff"
          >
            <path
              d="M13 7h-2v4H7v2h4v4h2v-4h4v-2h-4V7zm-1-5C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zm0 18c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8z"
            ></path>
          </svg>
          Добавить
        </button>
      </section>

      <section>
        <hr class="w-full border-t border-gray-600 my-4" />
        <!-- Фильтрация | Пагинация -->
        <div class="flex justify-between content-center">
          <div>
            <label
              for="searchFilter"
              class="block text-sm font-medium text-gray-700"
              >Фильтр по наименованию</label
            >
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                v-model="searchFilter"
                type="text"
                name="searchFilter"
                id="searchFilter"
                class="
                  block
                  w-full
                  pr-10
                  border-gray-300
                  text-gray-900
                  focus:outline-none focus:ring-gray-500 focus:border-gray-500
                  sm:text-sm
                  rounded-md
                "
                placeholder="Введите валюту"
              />
            </div>
          </div>
          <div>
            <button
              type="button"
              v-if="hasPrevPage"
              @click="pagination.page -= 1"
              class="
                mx-2
                my-4
                inline-flex
                items-center
                py-2
                px-4
                border border-transparent
                shadow-sm
                text-sm
                leading-4
                font-medium
                rounded-full
                text-white
                bg-gray-600
                hover:bg-gray-700
                transition-colors
                duration-300
                focus:outline-none
                focus:ring-2
                focus:ring-offset-2
                focus:ring-gray-500
              "
            >
              Назад
            </button>
            <button
              v-if="hasNextPage"
              type="button"
              @click="pagination.page += 1"
              class="
                mx-2
                my-4
                inline-flex
                items-center
                py-2
                px-4
                border border-transparent
                shadow-sm
                text-sm
                leading-4
                font-medium
                rounded-full
                text-white
                bg-gray-600
                hover:bg-gray-700
                transition-colors
                duration-300
                focus:outline-none
                focus:ring-2
                focus:ring-offset-2
                focus:ring-gray-500
              "
            >
              Вперёд
            </button>
          </div>
        </div>
      </section>

      <template v-if="tickers.length">
        <hr class="w-full border-t border-gray-600 my-4" />

        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="t in paginateTickers"
            :key="t.name"
            @click="choiceCurrentTicker(t)"
            :class="{
              'border-4': curTicker?.name === t.name,
            }"
            class="
              bg-white
              overflow-hidden
              shadow
              rounded-lg
              border-purple-800 border-solid
              cursor-pointer
            "
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ t.name }} - USD
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ t.price }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              @click.stop="handleDeleteTicker(t)"
              class="
                flex
                items-center
                justify-center
                font-medium
                w-full
                bg-gray-100
                px-4
                py-4
                sm:px-6
                text-md text-gray-500
                hover:text-gray-600 hover:bg-gray-200 hover:opacity-20
                transition-all
                focus:outline-none
              "
            >
              <svg
                class="h-5 w-5"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 20 20"
                fill="#718096"
                aria-hidden="true"
              >
                <path
                  fill-rule="evenodd"
                  d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z"
                  clip-rule="evenodd"
                ></path></svg
              >Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>
      <section class="relative" v-if="curTicker">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
          VUE - USD
        </h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64">
          <div
            v-for="(g, idx) in recountGraphToPercent()"
            :key="idx"
            class="bg-purple-800 border w-10"
            :style="`height: ${g}%;`"
          ></div>
        </div>
        <button
          type="button"
          class="absolute top-0 right-0"
          @click="clearActiveTickerAndGraph"
        >
          <svg
            xmlns="http://www.w3.org/2000/svg"
            xmlns:xlink="http://www.w3.org/1999/xlink"
            xmlns:svgjs="http://svgjs.com/svgjs"
            version="1.1"
            width="30"
            height="30"
            x="0"
            y="0"
            viewBox="0 0 511.76 511.76"
            style="enable-background: new 0 0 512 512"
            xml:space="preserve"
          >
            <g>
              <path
                d="M436.896,74.869c-99.84-99.819-262.208-99.819-362.048,0c-99.797,99.819-99.797,262.229,0,362.048    c49.92,49.899,115.477,74.837,181.035,74.837s131.093-24.939,181.013-74.837C536.715,337.099,536.715,174.688,436.896,74.869z     M361.461,331.317c8.341,8.341,8.341,21.824,0,30.165c-4.16,4.16-9.621,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    l-75.413-75.435l-75.392,75.413c-4.181,4.16-9.643,6.251-15.083,6.251c-5.461,0-10.923-2.091-15.083-6.251    c-8.341-8.341-8.341-21.845,0-30.165l75.392-75.413l-75.413-75.413c-8.341-8.341-8.341-21.845,0-30.165    c8.32-8.341,21.824-8.341,30.165,0l75.413,75.413l75.413-75.413c8.341-8.341,21.824-8.341,30.165,0    c8.341,8.32,8.341,21.824,0,30.165l-75.413,75.413L361.461,331.317z"
                fill="#718096"
                data-original="#000000"
              ></path>
            </g>
          </svg>
        </button>
      </section>
    </div>
  </div>
</template>

<script>
// 1. Пагинация и фльтрация через роутер | Критичность [2]
// 2. Магические числа (URL, API Key, 5000) | Критичность [3]
// 3. При удалении тикера ошибка с пагинацией (если это была last страница на которой располагался тикер) | Критичность [2]
// 4. Много запросов, хотя все данные можно получить через один запрос | Критичность [4]
// 5. Запросы напрямую внутри компонента | Критичность [4]
// 6. Наличие в состоянии зависимых данных | Критичность [5]
// 7. По три запроса на один тикер, при удалени тикера остаётся подписка на событие | Критичность [5]
// 8. Обработка ошибок API | Критичность [5]
// 9. Не исправность графика + выглядит неприемлемо когда становится большим | Критичность [5]
// 10. При удалении тикера остаётся подписка на событие  | Критичность [5]

// Параллельно
//
//
export default {
  name: "App",
  data() {
    return {
      localStorage: window.localStorage,
      ticker: "",
      tickers: [],
      graph: [],
      curTicker: null,
      coinList: [],
      coinHelpers: [],
      tickerAlert: false,
      isProcessing: false,
      searchFilter: "",
      pagination: {
        page: 1,
        pageSize: 6,
      },
      tickersLength: 0,
      api: {
        url: "https://min-api.cryptocompare.com/data/price",
        apiKey:
          "1109a3d1352ffaa0fe00d179a736905584b7c8db5374d6719c7321f635e10a2b",
      },
    };
  },
  async mounted() {
    this.isProcessing = true;
    await this.fetchCoinlist();
    this.isProcessing = false;
    let storageTickers = this.localStorage.getItem("tickers");
    if (storageTickers) {
      this.tickers = JSON.parse(storageTickers);
      this.tickers.forEach((ticker) => this.subscribeToUpdate(ticker.name));
    }
  },
  computed: {
    paginateTickers() {
      const finish = this.pagination.page * this.pagination.pageSize - 1;
      const start = finish - this.pagination.pageSize + 1;
      return this.filteredTickers.slice(start, finish + 1);
    },

    filteredTickers() {
      return this.tickers.filter((t) =>
        t.name.includes(this.searchFilter.toUpperCase())
      );
    },

    hasPrevPage() {
      return this.pagination.page > 1;
    },

    hasNextPage() {
      return (
        this.pagination.page <
        Math.ceil(this.filteredTickers.length / this.pagination.pageSize)
      );
    },
  },
  methods: {
    async fetchData(currency) {
      const url = `${this.api.url}?fsym=${currency}&tsyms=USD&api_key=${this.api.apiKey}`;
      const response = await fetch(url);
      return await response.json();
    },

    recountGraphToPercent() {
      const maxVal = Math.max(...this.graph);
      const minVal = Math.min(...this.graph);
      return this.graph.map((val) => {
        return 5 + ((val - minVal) * 95) / (maxVal - minVal);
      });
    },

    async fetchCoinlist() {
      let response = await fetch(
        "https://min-api.cryptocompare.com/data/all/coinlist?summary=true"
      );
      response = await response.json();
      this.coinList = Object.keys(response.Data);
    },
    handleAddTickerFromHelpers(coin) {
      this.handleAddTicker(coin);
    },
    isTickerAlreadyExist(name) {
      return this.tickers.findIndex((t) => t.name === name) >= 0 ? true : false;
    },
    handleAddTicker(tickerName = false) {
      const name = tickerName
        ? tickerName.toUpperCase()
        : this.ticker.toUpperCase();

      this.isTickerAlreadyExist(name)
        ? (this.tickerAlert = true)
        : (this.tickers = [
            { name: name, price: "-", intervalId: null },
            ...this.tickers,
          ]);

      this.ticker = "";
    },
    subscribeToUpdate(tickerName) {
      let intervalId = setInterval(async () => {
        const response = await this.fetchData(tickerName);
        let findTicker = this.tickers.find((t) => t.name === tickerName);
        findTicker.price = response.USD;
        findTicker.intervalId = intervalId;
        if (this.curTicker?.name === tickerName) {
          this.graph.push(response.USD);
        }
      }, 5000);
    },
    handleDeleteTicker(deletedTicker) {
      this.tickers = this.tickers.filter((t) => t !== deletedTicker);
      clearInterval(deletedTicker.intervalId);
      this.localStorage.setItem("tickers", JSON.stringify(this.tickers));
      this.clearActiveTickerAndGraph();
    },
    clearActiveTickerAndGraph() {
      this.curTicker = null;
      this.graph = [];
    },
    choiceCurrentTicker(currentTicker) {
      this.clearActiveTickerAndGraph();
      this.curTicker = currentTicker;
    },
  },
  watch: {
    ticker(value) {
      this.tickerAlert = false;
      if (value === "") {
        this.coinHelpers = [];
        return;
      }
      this.coinHelpers = this.coinList
        .filter((val) => val.indexOf(value.toUpperCase()) !== -1)
        .splice(0, 4);
    },
    tickers() {
      // this.subscribeToUpdate(this.ticker); // TODO watch
      this.localStorage.setItem("tickers", JSON.stringify(this.tickers));
    },
    searchFilter() {
      this.pagination.page = 1;
    },
  },
};
</script>
