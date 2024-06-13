<template>
  <div id="app">
    <div class="parse-form" v-if="!parseLink || invalidLink">
      <input
        type="text"
        placeholder="Link"
        ref="inputLink"
        @keyup.enter="parse"
      />
      <button @click="parse">Parse</button>
      <div v-if="invalidLink" class="invalid-link">
        Invalid Link. Please enter a valid link.
      </div>
    </div>
    <div v-if="!invalidLink && loading" class="loading-link">loading...</div>

    <template v-if="parseLink && !invalidLink && !loading">
      <ParseInfo :cards="cards" :parseLink="parseLink" />
      <div class="sorting">
        <div
          class="sorting-btn"
          :class="{ active: sortBy === 'price' }"
          @click="sortBy = 'price'"
        >
          By Price
        </div>
        <div
          class="sorting-btn"
          :class="{ active: sortBy === 'name' }"
          @click="sortBy = 'name'"
        >
          By Name
        </div>
        <div
          class="sorting-btn"
          :class="{ active: sortBy === 'reset' }"
          @click="sortBy = 'reset'"
        >
          Reset
        </div>
      </div>
      <div
        class="pagination"
        v-if="sortedCards.length === 42 || currentPage !== 0"
      >
        <button
          class="pagination-btn"
          :disabled="currentPage === 0"
          @click="prevPage"
        >
          Prev
        </button>
        <button class="pagination-btn" @click="nextPage">Next</button>
      </div>
      <div class="card-list">
        <div v-for="(card, index) in sortedCards" :key="index" class="card">
          <div class="card__main">
            <strong class="card__number">{{ index + 1 }}</strong>
            <div v-if="getElement(card, 'img')" class="card__img">
              <img
                :src="getAttribute(getElement(card, 'img'), 'data-src')"
                :alt="getAttribute(getElement(card, 'img'), 'alt')"
              />
            </div>
            <p
              v-for="(name, index) in getText(card, 'p')"
              :key="index"
              :class="{ card__name: true, 'card__name--first': index === 0 }"
            >
              {{ name }}
            </p>
            <p class="card__price">&euro; {{ getPrice(card, 'span') }}</p>
          </div>
        </div>
      </div>
      <div class="pagination" v-if="sortedCards.length === 42">
        <button
          class="pagination-btn"
          :disabled="currentPage === 0"
          @click="prevPage"
        >
          Prev
        </button>
        <button class="pagination-btn" @click="nextPage">Next</button>
      </div>
    </template>
  </div>
</template>

<script>
import JsSoup from 'jssoup'
import ParseInfo from './components/ParseInfo.vue'
import axios from 'axios'
export default {
  name: 'App',
  components: {
    ParseInfo,
  },
  data() {
    return {
      cards: [],
      sortedCards: [],
      parseLink: '',
      sortBy: 'reset',
      readyToParse: false,
      invalidLink: false,
      loading: false,
      currentPage: 0,
    }
  },

  watch: {
    sortBy(value) {
      if (value === 'price') {
        this.sortedCards = [...this.cards].sort((a, b) => {
          const aPrice = parseFloat(this.getPrice(a, 'span'))
          const bPrice = parseFloat(this.getPrice(b, 'span'))
          return aPrice - bPrice
        })
      } else if (value === 'name') {
        this.sortedCards = [...this.cards].sort((a, b) =>
          this.getText(a, 'p')[1].localeCompare(this.getText(b, 'p')[1], 'ru')
        )
      } else {
        this.sortedCards = [...this.cards]
      }
    },
  },
  methods: {
    async parseData() {
      try {
        const response = await axios.get(this.parseLink)
        if (!response || !response.data) {
          throw new Error('Invalid response')
        }
        this.invalidLink = false
        const html = response.data
        const soup = new JsSoup(html)
        const cards = soup.findAll('div', { class: 'c-tile' })
        this.loading = false
        console.log('loaded')
        this.cards = cards
        this.sortedCards = [...cards]
      } catch (error) {
        this.invalidLink = true
      }
    },
    getText(element, tag) {
      const elements = element.findAll(tag)
      const textArray = elements.map((element) => element.text)
      return textArray
    },
    getElement(element, tag) {
      return element.find(tag)
    },
    getElements(element, tag) {
      return []
    },
    getAttribute(element, attribute) {
      const attrs = element.attrs
      return attrs?.[attribute] || ''
    },
    getPrice(element, tag) {
      const priceElement = element.find(tag)
      const price = priceElement ? priceElement.text : ''
      const digitsOnly = price.replace(/[^0-9.]/g, '')
      return digitsOnly
    },
    parse() {
      this.loading = true
      console.log('loading...')
      this.parseLink = this.$refs.inputLink.value
      this.parseData()
      this.readyToParse = true
    },
    nextPage() {
      if (this.parseLink.includes('?start')) {
        this.parseLink = this.parseLink.replace(
          /(\?start=\d+)$/,
          `?start=${parseInt(this.currentPage * 42, 10) + 42}`
        )
      } else if (this.currentPage === 0) {
        this.parseLink += '?start=' + 42
      } else {
        this.parseLink += '?start=' + this.currentPage * 42
      }
      this.currentPage++
      this.loading = true
      this.parseData()
    },
    prevPage() {
      if (this.parseLink.includes('?start')) {
        this.parseLink = this.parseLink.replace(
          /(\?start=\d+)$/,
          `?start=${parseInt(this.currentPage * 42, 10) - 42}`
        )
      } else {
        this.parseLink += '?start=' + this.currentPage * 42
      }
      this.currentPage--
      this.loading = true
      this.parseData()
    },
  },
}
</script>

<style lang="scss">
@function rem($value) {
  @return $value / 16px * 1rem;
}

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Roboto', sans-serif;
  background-color: #f3f3f3;
}

#app {
  min-height: 100vh;
  overflow-x: hidden;
}

.parse-form {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  margin-bottom: 1rem;
  min-height: 100vh;
  gap: rem(30px);
  input {
    padding: 0.5rem;
    border-radius: 5px;
    border: 1px solid #ccc;
    margin-right: 0.5rem;
    width: 50%;
  }
  button {
    padding: 0.5rem;
    border-radius: 5px;
    border: 1px solid #ccc;
    background-color: #5c8b81;
    color: #fff;
    cursor: pointer;
    transition: all 0.3s ease-in-out;
    &:hover {
      background-color: #fff;
      color: #5c8b81;
      transition: all 0.3s ease-in-out;
    }
    &:disabled {
      background-color: #ccc;
      border: 1px solid #ccc;
      cursor: not-allowed;
      transition: all 0.3s ease-in-out;
      color: #fff;
    }
    &[disabled] {
      background-color: #ccc;
      border: 1px solid #ccc;
      cursor: not-allowed;
      transition: all 0.3s ease-in-out;
      color: #fff;
    }
  }
  .invalid-link {
    color: red;
    font-size: 0.8rem;
  }
}

.loading-link {
  color: #5c8b81;
  font-size: 0.8rem;
  animation: pulse 1s ease-in-out infinite;
  display: flex;
  justify-content: center;
  align-items: center;
  width: 100%;
  height: 100%;
  min-height: 90vh;
  overflow-y: scroll;
}

@keyframes pulse {
  0% {
    transform: scale(1);
  }
  50% {
    transform: scale(1.3);
  }
  100% {
    transform: scale(1);
  }
}

.pagination {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  width: 100%;
  gap: 1rem;
  margin-bottom: rem(30px);
  button {
    padding: 0.5rem;
    border-radius: 5px;
    border: 1px solid #ccc;
    background-color: #5c8b81;
    color: #fff;
    cursor: pointer;
    transition: all 0.3s ease-in-out;
    &:hover {
      background-color: #fff;
      color: #5c8b81;
      transition: all 0.3s ease-in-out;
    }
  }
}

.card-list {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  width: 100%;
  flex-wrap: wrap;
  gap: rem(10px);

  .card {
    width: 19%;
    height: rem(450px);
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 5px;
    margin-bottom: 2rem;
    cursor: pointer;
    transition: all 0.3s ease-in-out;
    background-color: #fff;
    &:hover {
      border: 1px solid #000;
      transition: all 0.3s ease-in-out;
    }
    .card__img {
      width: 100%;
      height: rem(300px);
      margin-bottom: rem(10px);
      img {
        width: 100%;
        height: 100%;
      }
    }
    .card__number {
      position: absolute;
      bottom: rem(10px);
      right: rem(10px);
    }
    .card__main {
      position: relative;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: flex-start;
      height: 100%;
      background-color: #fff;
      .card__name {
        font-size: rem(14px);
        margin-bottom: rem(10px);
        background-color: #fff;
        &--first {
          font-size: rem(20px);
        }
      }
      .card__price {
        font-size: rem(20px);
        background-color: #fff;
      }
    }
  }
}

.sorting {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: rem(15px);
  align-items: center;
  width: 100%;
  margin-bottom: rem(30px);
  .sorting-btn {
    cursor: pointer;
    padding: 0.5rem 1rem;
    border: 1px solid #ccc;
    border-radius: rem(13px);
    transition: all 0.3s ease-in-out;
    background: #fff;
    &.active {
      transform: scale(0.95);
      transition: all 0.3s ease-in-out;
      background: #a3a3a3;
      cursor: not-allowed;
      border: 1px solid #a3a3a3;
      &:hover {
        border: 1px solid #a3a3a3;
      }
    }
    &:hover {
      border: 1px solid #000;
      transition: all 0.3s ease-in-out;
    }
  }
}
</style>
