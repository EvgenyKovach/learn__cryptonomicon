<template>
  <div class="container mx-auto flex flex-col items-center bg-gray-100 p-4">
    <!--    <div-->
    <!--      class="fixed w-100 h-100 opacity-80 bg-purple-800 inset-0 z-50 flex items-center justify-center"-->
    <!--    >-->
    <!--      <svg-->
    <!--        class="animate-spin -ml-1 mr-3 h-12 w-12 text-white"-->
    <!--        xmlns="http://www.w3.org/2000/svg"-->
    <!--        fill="none"-->
    <!--        viewBox="0 0 24 24"-->
    <!--      >-->
    <!--        <circle-->
    <!--          class="opacity-25"-->
    <!--          cx="12"-->
    <!--          cy="12"-->
    <!--          r="10"-->
    <!--          stroke="currentColor"-->
    <!--          stroke-width="4"-->
    <!--        ></circle>-->
    <!--        <path-->
    <!--          class="opacity-75"-->
    <!--          fill="currentColor"-->
    <!--          d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"-->
    <!--        ></path>-->
    <!--      </svg>-->
    <!--    </div>-->
    <!-- 
      @todo 
    [X]  2. При удалении не сбрасывается interval timer | criti
    [Х]  1. Одинаковый код в watch | major \\ я оптимизировал эту часть изначально
    [X]  3. Кол-во запросов (сократить) | crit \\ сокращено до 1 + на старте лист + убрали запрос не валидных валют
    [Х]  6. Удаление тикера не измения localStorage | minor \\ После внесение его в watch все изменилось
    [Х]  5. Обработка ошибок (api) | crit \\ на уровне тестового проекта валидация добавлена

    [?]  7. График ужасно выглядит если будет много цен |  crit  \\ вроде всё ок, уходяд лишние
    [?]  8. Наличие в состоянии зависимых данных | crit \\ надо будет узнать, о каких шла речь
    [?]  4. Запросы кода напрямую в коде (???) | crit  \\ не пон
    
    []  9. localStorage и анонимные вкладки | major  \\ не пон
    []  10. Маг. строки и числа (url, 5000ms задержки, ключ, кол-во запросов на стр) | minor \\ слишком разбитый минор, что-то сократили в коде из этого, пока на карандаше
      --
      Дополнительно
    [X]  4. Замена имени у current > selectedTicker
    [X]  1. График сломан, если везде один-е значения
    [X]  2. При удалении тикера остается выборанный
    [X]  3. При удалении тикеров со страницы, мeнять localStorage и страницу 
    -->
    <div class="container">
      <section>
        <div class="flex">
          <div class="input-block max-w-xs">
            <label for="wallet" class="block text-sm font-medium text-gray-700"
              >Тикер</label
            >
            <div class="mt-1 relative rounded-md shadow-md">
              <input
                type="text"
                name="wallet"
                id="wallet"
                class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
                placeholder="Например DOGE"
                v-model="ticker"
                :disabled="disableInterface"
                @input="inputTicker"
              />
            </div>
            <div
              class="flex bg-white shadow-md p-1 rounded-md shadow-md flex-wrap"
              v-if="ticker.length"
            >
              <span
                class="inline-flex items-center px-2 m-1 rounded-md text-xs font-medium bg-gray-300 text-gray-800 cursor-pointer"
                v-for="tip in formatTips.slice(0, 4)"
                :key="tip"
                @click="addTicker(tip.Symbol)"
              >
                {{ tip.Symbol }}
              </span>
            </div>
            <div v-if="hasAlready" class="text-sm text-red-600">
              Такой тикер уже добавлен
            </div>
            <div v-if="errors.coinlist" class="text-sm text-red-600">
              {{ errors.coinlist }}
            </div>
          </div>
        </div>
        <button
          @click.stop="addTicker()"
          type="button"
          :class="[{ disabled: disableInterface }]"
          class="my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
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
        <div class="filter">
          <div class="input-block max-w-xs">
            <input
              v-model="filter"
              class="block w-full pr-10 border-gray-300 text-gray-900 focus:outline-none focus:ring-gray-500 focus:border-gray-500 sm:text-sm rounded-md"
            />
          </div>
          <button
            @click="page--"
            :class="{ disabled: page <= 0 }"
            class="filter-button my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
          >
            Назад
          </button>
          <button
            :class="{ disabled: !hasNextPage }"
            @click="page++"
            class="filter-button my-4 inline-flex items-center py-2 px-4 border border-transparent shadow-sm text-sm leading-4 font-medium rounded-full text-white bg-gray-600 hover:bg-gray-700 transition-colors duration-300 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-gray-500"
          >
            Вперед
          </button>
        </div>
      </section>

      <template v-if="filteredTickers.length">
        <div v-if="errors.tickers" class="text-sm text-red-600">
          {{ errors.tickers }}
        </div>
        <hr class="w-full border-t border-gray-600 my-4" />
        <dl class="mt-5 grid grid-cols-1 gap-5 sm:grid-cols-3">
          <div
            v-for="ticker in paginationTickers"
            :key="ticker.name"
            @click="selectTicker(ticker)"
            :class="{
              'border-4': selectedTicker === ticker,
            }"
            class="bg-white overflow-hidden shadow rounded-lg border-purple-800 border-solid cursor-pointer"
          >
            <div class="px-4 py-5 sm:p-6 text-center">
              <dt class="text-sm font-medium text-gray-500 truncate">
                {{ ticker.name }} - USD
              </dt>
              <dd class="mt-1 text-3xl font-semibold text-gray-900">
                {{ +ticker.count === 0 ? "> 0.01" : ticker.count }}
              </dd>
            </div>
            <div class="w-full border-t border-gray-200"></div>
            <button
              @click.stop="delTicker(ticker)"
              class="flex items-center justify-center font-medium w-full bg-gray-100 px-4 py-4 sm:px-6 text-md text-gray-500 hover:text-gray-600 hover:bg-gray-200 hover:opacity-20 transition-all focus:outline-none"
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
                ></path>
              </svg>
              Удалить
            </button>
          </div>
        </dl>
        <hr class="w-full border-t border-gray-600 my-4" />
      </template>
      <section class="relative" v-if="selectedTicker">
        <h3 class="text-lg leading-6 font-medium text-gray-900 my-8">
          {{ selectedTicker.name + " - USD" }}
        </h3>
        <div class="flex items-end border-gray-600 border-b border-l h-64">
          <div
            class="bg-purple-800 border w-10"
            v-for="(bar, ind) in normalizeGraph"
            :style="{ height: `${bar}%`, minHeight: '5%' }"
            :key="ind"
          ></div>
        </div>
        <button
          type="button"
          class="absolute top-0 right-0"
          @click="selectedTicker = null"
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
export default {
  name: "CryptoNomicon",
  data() {
    return {
      ticker: "",
      tickers: [],
      selectedTicker: null,
      graph: [],
      hasAlready: false,
      tips: [],
      filter: "",
      page: 0,

      apiIntrevalRequest: null,
      graphInterval: null,
      disableInterface: false,
      errors: {},
    }
  },
  created() {
    const windowData = Object.fromEntries(
      new URL(window.location).searchParams.entries()
    )

    if (windowData.filter) this.filter = windowData.filter
    if (windowData.page) this.page = windowData.page

    const tickersData = localStorage.getItem("crypto-list")

    if (tickersData) {
      this.tickers = JSON.parse(tickersData)
      this.filtredTickers = this.tickers
      this.tickers.forEach((ticker) => {
        this.changeIntervalTickers(ticker.name)
      })
    }
  },
  methods: {
    inputTicker() {
      this.hasAlready = false
    },
    selectTicker(t) {
      this.selectedTicker = t
      this.graph = []
      this.changeIntervalTickers()
    },
    addTicker(name) {
      const newTicker = {
        name: name ? name : this.ticker,
        count: "-",
      }

      if (this.checkTickerInArray(newTicker.name)) {
        this.tickers = [newTicker, ...this.tickers]
        this.changeIntervalTickers()
      } else {
        this.hasAlready = true
      }
    },
    changeIntervalTickers() {
      const tickersList = this.tickers.map((t) => t.name)

      clearInterval(this.apiIntrevalRequest)

      if (tickersList.length) {
        const tickersListToStr = tickersList.join(",")
        this.apiIntrevalRequest = setInterval(async () => {
          await fetch(
            `https://min-api.cryptocompare.com/data/price1multi?fsyms=${tickersListToStr}&tsyms=USD&api_key=442cefa86d2ef5c7ca29db8412ad9f4b63dc7304159088e541fb3e62954f6e69`
          )
            .then((r) => {
              const data = r.json()

              if (data.Response === "Error") {
                this.errors.tickers =
                  "Ошибка при попытке получить данные о валютах"
              } else {
                this.errors.tickers = null
                this.tickers.forEach((currency) => {
                  if (data[currency.name]) {
                    currency.count = data[currency.name]["USD"].toFixed(2)
                  }
                })
              }
            })
            .catch(() => {
              this.errors.tickers =
                "Ошибка при попытке получить данные о валютах"
            })
        }, 3000)
      }
    },
    delTicker(ticker) {
      this.tickers = [...this.tickers.filter((t) => t !== ticker)]

      if (this.selectedTicker?.name === ticker.name) {
        this.selectedTicker = null
      }
      this.changeIntervalTickers()
    },
    checkTickerInArray(name) {
      return (
        !this.tickers.filter((t) => name.toLowerCase() === t.name.toLowerCase())
          .length && this.cointList.includes(name)
      )
    },
    async getTips() {
      await fetch(`https://min-api.cryptocompare.com/data/all/coinli1st`)
        .then((r) => {
          const data = r.json()

          console.log(r)

          if (data.Response === "Error") {
            this.disableInterface = true
            this.errors.coinlist =
              "Ошибка при попытке получить данные о списке валют"
          } else {
            this.errors.cointst = null
            this.tips = data
          }
        })
        .catch(() => {
          this.errors.coinlist =
            "Ошибка при попытке получить данные о списке валют"
        })
    },
    pushStateWithChange() {
      window.history.pushState(
        null,
        document.title,
        `${window.location.pathname}?filter=${this.filter}&page=${this.page}`
      )
    },
  },
  async mounted() {
    await this.getTips()
  },
  computed: {
    startIndex() {
      return +this.page * 6
    },
    endIndex() {
      return (+this.page + 1) * 6
    },
    formatTips() {
      if (!this.tips.Data) return []
      const tipsArray = Object.values(this.tips.Data)

      return tipsArray.filter((t) => {
        return (
          t.FullName.toLowerCase().indexOf(this.ticker.toLowerCase()) !== -1 &&
          t.IsTrading
        )
      })
    },
    filteredTickers() {
      return this.tickers.filter((ticker) =>
        ticker.name.toLowerCase().includes(this.filter.toLowerCase())
      )
    },
    paginationTickers() {
      return this.filteredTickers.slice(+this.startIndex, +this.endIndex)
    },
    normalizeGraph() {
      const max = Math.max(...this.graph)
      const min = Math.min(...this.graph)

      if (min === max) {
        return this.graph.map(() => 50)
      } else {
        return this.graph.map((price) => 5 + ((price - min) * 95) / (max - min))
      }
    },
    hasNextPage() {
      return this.filteredTickers.length > (this.page + 1) * 6
    },
    pageOptions() {
      return {
        filter: this.filter,
        page: this.page,
      }
    },
    cointList() {
      return this.formatTips.map((coin) => coin.Symbol)
    },
  },
  watch: {
    tickers() {
      localStorage.setItem("crypto-list", JSON.stringify(this.tickers))
    },
    filter() {
      this.page = 0
      this.pushStateWithChange()
    },
    pageOptions() {
      this.pushStateWithChange()
    },
    paginationTickers() {
      if (this.paginationTickers.length === 0 && this.page > 0) {
        this.page--
      }
    },
    selectedTicker() {
      if (this.selectedTicker?.name) {
        clearInterval(this.graphInterval)
        this.graphInterval = setInterval(() => {
          const getTickerInfo = this.tickers.filter(
            (ticker) => ticker.name === this.selectedTicker.name
          )

          this.graph.push(getTickerInfo[0].count)
          this.graph.length > 30 ? this.graph.shift() : ""
        }, 3000)
      } else clearInterval(this.graphInterval)
    },
  },
}
</script>

<style lang="css" scoped>
.input-block {
  min-width: 450px;
}

#wallet {
  transition: 0.6s;
}

.filter-button {
  margin-right: 10px;
}

.disabled {
  pointer-events: none;
  opacity: 0.3;
}
</style>
