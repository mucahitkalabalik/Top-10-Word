<template>
  <div class="mk-main"> 
      <template v-if="loading">
              <h1>LOADING</h1>
        </template>
    <template v-if="!loading">
        <h1>Top 10 Word</h1>
    <div v-for="(item, index) in content" :key="index">
      <span v-for="(nItem, nIndex) in item" :key="nIndex + index">
        {{ nItem }}
      </span>
    </div>
    </template>
    
  </div>
</template>

<script>
export default {
  data() {
    return {
      multiple: {},
      content: [],
      loading:true
    };
  },
  async fetch() {
    const list = await this.$axios.get(`/api/newstories.json?print=pretty`);
    const resolve = [];
    const first25 = list.data.slice(0, 25);
    const arr = [];
    first25.forEach((itemId) => {
      resolve.push(
        this.$axios.get(`/api/item/${itemId}.json`).then((res) => res.data)
      );
    });
    const fetchedData = await Promise.all(resolve);
    this.$nextTick(() => {
      fetchedData
        .filter((e) => e.type === "story")
        .forEach((element) => {
          const current = element?.title?.split(" ") ?? [];
          current.forEach((e) => {
            const sensivite = e.trim().toLowerCase();
            if (this.multiple[sensivite]) {
              this.multiple[sensivite]++;
            } else {
              this.multiple[sensivite] = 1;
            }
          });
        });
      for (const property in this.multiple) {
        const nArr = [];
        nArr.push(property, this.multiple[property]);
        arr.push(nArr);
      }

      arr.sort(function (a, b) {
        return b[1] - a[1];
      });

      const top10 = arr.slice(0, 10);
      this.content = top10;
      this.loading = false
    });
  },
  mounted() {
    this.$fetch();
  },
  methods: {},
};
</script>

<style>
.mk-main{
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  height: 100vh;
}
</style>