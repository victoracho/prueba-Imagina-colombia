<template>
  <q-page class="flex flex-center">
    <q-header bordered class="bg-white text-primary">
      <q-toolbar>
        <q-toolbar-title class="text-center">
          <q-avatar>
            <img src="https://cdn.quasar.dev/logo-v2/svg/logo.svg" />
          </q-avatar>
          Prueba de Victor Gonzalez con Quasar.
        </q-toolbar-title>
      </q-toolbar>
    </q-header>
    <div class="row">
      <div class="col-12" v-for="post in slicedPosts" :key="post.key">
        <q-card class="my-card">
          <q-card-section> {{ post }}</q-card-section>
        </q-card>
      </div>
    </div>
    <div class="row">
      <q-pagination
        v-model="page"
        :min="currentPage"
        :max="Math.ceil(posts.length / totalPages)"
        :input="true"
        input-class="text-orange-10"
      >
      </q-pagination>
    </div>
  </q-page>
</template>

<script>
import { ref, onMounted, defineComponent, computed } from "vue";
import axios from "axios";

export default defineComponent({
  name: "MainLayout",
  setup() {
    const page = ref(1);
    const currentPage = ref(1);
    const posts = ref([]);
    const totalPages = ref(5);
    const nextPage = ref(null);

    // function that formats all the data
    const iterateObj = (results) => {
      let arr = [];
      results.forEach((s) => {
        // we get all keys from the
        let val = s.value;
        let key = s.key;
        key = key.split("::");
        let firstKey = key[0];
        key = key[1].split(".");
        let secondKey = key[0];
        let thirdKey = key[1];
        let fourthKey = key[2];
        let condition = true;
        // we add letter Q if the first character starts with I, nevertheless we append letter Q
        firstKey =
          firstKey.charAt(0) == "i"
            ? "q" + firstKey.substring(1)
            : "q" + firstKey;

        // we replace the word cms on the second key if its the case
        secondKey = secondKey.includes("cms")
          ? secondKey.replace("cms", "layout")
          : secondKey;

        // we assure that if some language key is missing, is provided with the one that exists
        let esp = s.hasOwnProperty("es") ? s.es.value : s.en.value;
        let en = s.hasOwnProperty("en") ? s.en.value : s.es.value;

        let objEsp = null;
        let objEn = null;

        // we add the fourth key if its provided, if its not, we add only until third key
        if (fourthKey) {
          objEsp = {
            [firstKey]: { [secondKey]: { [thirdKey]: esp, [fourthKey]: val } },
          };
          objEn = {
            [firstKey]: { [secondKey]: { [thirdKey]: en, [fourthKey]: val } },
          };
        } else {
          objEsp = { [firstKey]: { [secondKey]: { [thirdKey]: esp } } };
          objEn = { [firstKey]: { [secondKey]: { [thirdKey]: en } } };
        }

        arr.push({
          esp: objEsp,
          en: objEn,
        });
      });
      return arr;
    };

    // function to make the pagination and iterate through partial data, and not all of it
    const slicedPosts = computed(() => {
      let results = posts.value.slice(
        (page.value - 1) * totalPages.value,
        (page.value - 1) * totalPages.value + totalPages.value
      );
      results = iterateObj(results);
      return results;
    });

    onMounted(async () => {
      // we retrieve all data with axios
      let res = await axios.get(
        "https://inmeet.ozonohosting.com/api/translation/v2/translations?filter={%22search%22:%22::cms%22}"
      );
      posts.value = res.data.data;
    });

    return {
      page,
      currentPage,
      posts,
      totalPages,
      nextPage,
      slicedPosts,
    };
  },
});
</script>

<style lang="scss" scoped>
.my-card {
  margin: 10px;
}
</style>