<template>
  <div class="parse-info">
    <p>
      Link: <a :href="parseLink"> {{ parseLink }} </a>
    </p>
    <p>Cards Amount: {{ cards.length }}</p>
    <p>Parsing Time: {{ new Date() }}</p>
  </div>
</template>

<script>
export default {
  name: 'ParseInfo',
  props: {
    cards: {
      type: Array,
      default: () => [],
    },
    parseLink: {
      type: String,
      default: '',
    },
  },
}
</script>

<style lang="scss">
.parse-info {
  padding: 1rem;
  border-radius: 5px;
  margin-bottom: 1rem;
  a {
    text-decoration: none;
    color: #5c8b81;
    &:hover {
      text-decoration: underline;
    }
  }
  p {
    margin: 0;
    text-align: center;
    font-weight: bold;
    font-size: 1.2rem;
    color: #000;
  }
}
</style>
